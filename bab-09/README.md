# Bab 09 — Studi Kasus: Dashboard SaaS dari Nol ke Production

> Di bab-bab sebelumnya kamu belajar komponen, binding, DI, dan routing secara terpisah. Sekarang saatnya melihat semua itu bekerja bersama dalam satu aplikasi nyata. Bab ini membahas arsitektur Aurelia Ops — SaaS admin dashboard yang menjadi referensi course ini — dan bagaimana setiap keputusan teknis dibuat.

---

## Gambaran Aplikasi

**Aurelia Ops** adalah SaaS admin dashboard multi-tenant dengan fitur:
- Login dengan session cookie (Argon2id password hash)
- Multi-workspace — satu user bisa punya beberapa workspace
- Manajemen proyek: CRUD, filter, sort, pagination server-side
- Manajemen tim: invite, role (owner/admin/member), revoke
- Billing via Stripe Checkout + Customer Portal
- Audit log aktivitas

Stack teknisnya:
- **Frontend:** Aurelia 2 + Vite + Tailwind CSS 4
- **Backend:** Hono + Drizzle ORM + PostgreSQL
- **Auth:** session cookie, Argon2id, SHA-256 token hash
- **Billing:** Stripe (Hosted Checkout, tidak ada card data di server)
- **Shared types:** Zod schemas di `packages/shared`

---

## Arsitektur: Satu Seam yang Mengatur Segalanya

Keputusan arsitektur terpenting di Aurelia Ops adalah **satu titik isolasi tenancy**: file `apps/api/src/middleware/tenant.ts`.

```
Request masuk ke API
       ↓
/health, /api/auth/*, /api/billing/webhook
→ Tidak melalui tenant middleware

Semua route lain (/api/projects, /api/members, dll)
       ↓
tenant middleware
├── Verifikasi session cookie
├── Muat data user dari DB
├── Muat data workspace dari DB
└── Set c.var.tenant = { workspaceId, plan, userId, role }
       ↓
Handler route — hanya panggil tenantOf(c)
```

```typescript
// apps/api/src/middleware/tenant.ts
export const tenant = (): MiddlewareHandler => async (c, next) => {
  const session = await loadSession(getCookie(c, SESSION_COOKIE));
  if (!session) throw unauthorized('Your session has expired. Sign in again.');

  const workspace = await resolveWorkspace(session.userId, session.activeWorkspaceId);
  if (!workspace) throw forbidden('You are not a member of any workspace');

  // Satu set data tenant yang bersih — semua handler pakai ini
  c.set('tenant', {
    workspaceId: workspace.id,
    workspaceName: workspace.name,
    plan: workspace.plan,
    userId: session.userId,
    actorName: session.name,
    role: workspace.role,
  });
  await next();
};

// Fungsi helper yang dipakai di setiap handler
export function tenantOf(c: Context): TenantScope {
  return c.get('tenant');
}

// Guard role — dipakai untuk operasi sensitif
export function requireRole(tenant: TenantScope, ...roles: Role[]): void {
  if (!roles.includes(tenant.role)) {
    throw forbidden(`This action requires the ${roles.join(' or ')} role`);
  }
}
```

**Mengapa ini penting:** Kalau kamu mewarisi codebase Aurelia dan mau memahami bagaimana multi-tenancy bekerja, baca satu file ini. Semua isolasi data ada di sini. Tidak ada handler yang baca cookie atau header sendiri.

---

## Database Schema: 8 Tabel, Semua Terhubung

```
users
  ↓ (many-to-many via memberships)
workspaces ← sessions (active_workspace_id)
  ↓
├── memberships (workspace_id, user_id, role)
├── projects    (workspace_id, name, status, budget, due_date)
├── invites     (workspace_id, email, token_hash, expires_at)
├── audit_logs  (workspace_id, actor_name, action, target)
└── stripe_events (id dari Stripe, type, workspace_id)
```

Setiap tabel yang berisi data tenant memiliki `workspace_id`. Setiap query di handler selalu difilter dengan `WHERE workspace_id = ?`. Ini yang membuat isolasi data antar tenant terjamin di level SQL, bukan di level aplikasi.

```typescript
// Contoh query yang benar — workspace_id selalu disertakan
const projects = await db
  .select()
  .from(schema.projects)
  .where(eq(schema.projects.workspaceId, tenant.workspaceId)) // ← ini kunci isolasi
  .orderBy(desc(schema.projects.updatedAt));
```

---

## Membangun Halaman Projects: Semua Konsep Menyatu

Halaman Projects adalah halaman paling kompleks — CRUD lengkap, filter, sort, pagination. Mari kita bedah bagian per bagian.

### API: Handler GET /projects

```typescript
// apps/api/src/routes/projects.ts (disederhanakan)
projectRoutes.get('/projects', async (c) => {
  const { workspaceId } = tenantOf(c); // ← ambil tenant dari middleware
  const { page, pageSize, q, status, sort } = c.req.valid('query');
  const db = getDb();

  // Bangun filter secara dinamis
  const filters = [
    eq(schema.projects.workspaceId, workspaceId), // ← isolasi tenant
  ];
  if (status) filters.push(eq(schema.projects.status, status));
  if (q) {
    const like = `%${q}%`;
    filters.push(
      or(
        ilike(schema.projects.name, like),
        ilike(schema.projects.ownerName, like),
      )!
    );
  }

  // Jalankan count dan data secara paralel (lebih cepat)
  const [rows, [countRow]] = await Promise.all([
    db.select().from(schema.projects)
      .where(and(...filters))
      .orderBy(sort === 'name' ? schema.projects.name : desc(schema.projects.updatedAt))
      .limit(pageSize)
      .offset((page - 1) * pageSize),
    db.select({ value: count() }).from(schema.projects).where(and(...filters)),
  ]);

  return c.json({
    items: rows.map(toProject),
    page,
    pageSize,
    total: Number(countRow?.value ?? 0),
  });
});
```

**Poin teknis penting:**
- Count dan data diambil dalam satu `Promise.all` — tidak sequential
- Offset pagination cukup untuk 1.000 baris (tapi tidak untuk jutaan — pakai cursor pagination kalau data besar)
- Semua filter divalidasi Zod sebelum masuk handler

### API: Handler POST /projects dengan Plan Limits

```typescript
projectRoutes.post('/projects', async (c) => {
  const tenant = tenantOf(c);
  const input = c.req.valid('json');
  const db = getDb();

  // Cek limit plan — logika ini ada di packages/shared, bukan hardcoded
  const limits = limitsFor(tenant.plan);
  const [countRow] = await db
    .select({ value: count() })
    .from(schema.projects)
    .where(eq(schema.projects.workspaceId, tenant.workspaceId));

  if (Number(countRow?.value ?? 0) >= limits.projects) {
    return c.json({
      error: {
        code: 'plan_limit_reached',
        message: `Plan ${tenant.plan} dibatasi ${limits.projects} proyek.`,
      }
    }, 402); // 402 Payment Required
  }

  const [created] = await db
    .insert(schema.projects)
    .values({ ...input, workspaceId: tenant.workspaceId })
    .returning();

  // Catat ke audit log
  await db.insert(schema.auditLogs).values({
    workspaceId: tenant.workspaceId,
    actorName: tenant.actorName,
    action: 'project.created',
    target: created!.name,
  });

  return c.json({ project: created }, 201);
});
```

### Frontend: Halaman Projects

```typescript
// apps/web/src/pages/projects-page.ts (disederhanakan)
import { resolve } from 'aurelia';
import { api, ApiError } from '@/services/api';
import { ToastService } from '@/services/toast';
import type { Project } from '@aureliajs/shared';
import type { IRouteViewModel } from '@aurelia/router';

export class ProjectsPage implements IRouteViewModel {
  readonly toast = resolve(ToastService);

  // State
  projects: Project[] = [];
  total = 0;
  page = 1;
  pageSize = 25;
  q = '';
  status = '';
  sort = 'updated';
  isLoading = false;
  isCreating = false;

  // Dialog state
  dialogEl!: HTMLDialogElement;
  form = { name: '', ownerName: '', status: 'planning', budget: '', dueDate: '' };
  formError = '';

  // Load data saat halaman pertama dibuka
  async loading(): Promise<void> {
    await this.loadProjects();
  }

  async loadProjects(): Promise<void> {
    this.isLoading = true;
    try {
      const result = await api.listProjects({
        page: this.page,
        pageSize: this.pageSize,
        q: this.q || undefined,
        status: this.status as any || undefined,
        sort: this.sort as any,
      });
      this.projects = result.items;
      this.total = result.total;
    } catch {
      this.toast.error('Gagal memuat proyek.');
    } finally {
      this.isLoading = false;
    }
  }

  // Debounce search — tunggu 400ms setelah user berhenti mengetik
  private searchTimer?: ReturnType<typeof setTimeout>;
  onSearchInput(): void {
    clearTimeout(this.searchTimer);
    this.searchTimer = setTimeout(() => {
      this.page = 1;
      void this.loadProjects();
    }, 400);
  }

  onFilterChange(): void {
    this.page = 1;
    void this.loadProjects();
  }

  // Navigasi halaman
  goToPage(newPage: number): void {
    this.page = newPage;
    void this.loadProjects();
  }

  // CRUD
  openCreate(): void {
    this.form = { name: '', ownerName: '', status: 'planning', budget: '', dueDate: '' };
    this.formError = '';
    this.dialogEl.showModal();
  }

  async saveProject(event?: Event): Promise<void> {
    event?.preventDefault();
    this.formError = '';
    this.isCreating = true;

    try {
      await api.createProject({
        name: this.form.name.trim(),
        ownerName: this.form.ownerName.trim(),
        status: this.form.status as any,
        budgetCents: Math.round(parseFloat(this.form.budget || '0') * 100),
        dueDate: this.form.dueDate || null,
      });
      this.dialogEl.close();
      this.toast.success('Proyek berhasil dibuat!');
      await this.loadProjects();
    } catch (error) {
      if (error instanceof ApiError && error.code === 'plan_limit_reached') {
        this.formError = 'Limit plan tercapai. Upgrade untuk menambah lebih banyak proyek.';
      } else {
        this.formError = 'Gagal menyimpan proyek.';
      }
    } finally {
      this.isCreating = false;
    }
  }

  async deleteProject(id: string, name: string): Promise<void> {
    if (!confirm(`Hapus proyek "${name}"?`)) return;
    try {
      await api.deleteProject(id);
      this.toast.success('Proyek dihapus.');
      await this.loadProjects();
    } catch {
      this.toast.error('Gagal menghapus proyek.');
    }
  }

  detaching(): void {
    clearTimeout(this.searchTimer);
  }
}
```

---

## Pola Shared Types: Satu Sumber Kebenaran

Ini keputusan arsitektur yang paling mencegah bug di Aurelia Ops. Semua kontrak data ada di `packages/shared`:

```typescript
// packages/shared/src/schemas.ts

// Schema ini dipakai oleh:
// - API untuk validasi request body (Zod parse)
// - Frontend untuk validasi form sebelum submit
// - Keduanya untuk tipe TypeScript
export const createProjectSchema = z.object({
  name: z.string().trim().min(2).max(120),
  status: z.enum(['planning', 'active', 'paused', 'done']),
  budgetCents: z.number().int().min(0).max(1_000_000_000),
  dueDate: z.iso.date().nullable().default(null),
  ownerName: z.string().min(1).max(80),
});

export type CreateProjectInput = z.infer<typeof createProjectSchema>;

export const PLAN_LIMITS = {
  free:  { projects: 3,     seats: 3  },
  pro:   { projects: 5000,  seats: 10 },
  scale: { projects: 50000, seats: 50 },
} as const;
```

**Mengapa ini penting:** Kalau kamu ubah limit proyek dari 3 ke 5 untuk plan free, kamu cukup ubah satu baris di file ini. API yang enforce dan UI yang menampilkan limit langsung sinkron. Tidak mungkin drift.

---

## Error Contract: Format yang Konsisten

Semua error dari API menggunakan format yang sama:

```json
{
  "error": {
    "code": "validation_failed",
    "message": "Periksa form",
    "issues": [
      { "path": "name", "message": "Nama terlalu pendek" },
      { "path": "budget", "message": "Harus berupa angka" }
    ]
  }
}
```

Di frontend, field `issues[].path` dipetakan langsung ke field form yang error. Tidak perlu translasi atau logika khusus:

```typescript
// Kalau API return 422 validation_failed
if (error instanceof ApiError && error.issues.length > 0) {
  // Tampilkan error per field
  for (const issue of error.issues) {
    this.fieldErrors[issue.path] = issue.message;
  }
}
```

```html
<input name="name" value.two-way="form.name">
<span class="error" if.bind="fieldErrors.name">${fieldErrors.name}</span>
```

---

## Anti-Pattern yang Dihindari

Aurelia Ops dengan sengaja menghindari beberapa pola yang terlihat "mudah" tapi merusak jangka panjang:

**1. Tidak ada hardcode tenant ID di client**

```typescript
// ❌ JANGAN — client mengirim workspaceId sendiri
await api.createProject({ ...data, workspaceId: this.auth.workspace.id });

// ✅ BENAR — server ambil workspaceId dari session
await api.createProject(data); // server pakai tenantOf(c).workspaceId
```

**2. Tidak ada logika bisnis di frontend**

```typescript
// ❌ JANGAN — frontend check plan limit sendiri
if (this.projects.length >= 3 && this.auth.workspace.plan === 'free') {
  this.showUpgradePrompt = true;
}

// ✅ BENAR — cek di server, frontend hanya handle response
// Server return 402 plan_limit_reached
// Frontend handle error code-nya
```

**3. Tidak ada "silent fallback" yang menipu**

Kalau API tidak tersedia, aplikasi menampilkan banner "Demo data" dan data sample — bukan berpura-pura seperti data nyata. Lebih jujur, lebih aman.

---

## Latihan Bab 09

### Latihan 1: Trace Alur Request

Tanpa melihat kode, jawab: kalau user kirim `GET /api/projects?status=active&page=2`, apa saja yang terjadi secara berurutan di server sebelum response dikirim?

<details>
<summary>Jawaban</summary>

1. Request masuk ke Hono app di `app.ts`
2. `tenant()` middleware dijalankan:
   - Baca session cookie dari header
   - `loadSession()` — hash token, cari di tabel `sessions`
   - `resolveWorkspace()` — cari membership untuk user + workspace aktif
   - Set `c.var.tenant` dengan `workspaceId`, `plan`, `userId`, `role`
3. Request diteruskan ke `projectRoutes.get('/projects')`
4. Query params divalidasi dengan Zod schema (`paginationQuerySchema`)
5. `tenantOf(c)` dipanggil — ambil `workspaceId` dari context
6. Query dibuild: `WHERE workspace_id = ? AND status = ?`
7. `Promise.all([data query, count query])` — keduanya dijalankan paralel
8. Response di-parse dengan `paginated(projectSchema)` dan dikembalikan sebagai JSON

</details>

### Latihan 2: Tambah Field ke Project

Sebagai latihan konseptual: kalau kamu mau tambah field `priority` (`low`, `medium`, `high`) ke tabel projects, file apa saja yang perlu diubah dan dalam urutan apa?

<details>
<summary>Jawaban</summary>

**Urutan perubahan:**

1. **`apps/api/src/db/schema.ts`** — tambah `priorityEnum` dan kolom `priority` ke tabel `projects`
2. **`apps/api/drizzle/`** — generate migration baru: `pnpm db:generate`
3. **`packages/shared/src/schemas.ts`** — tambah `priority` ke `projectSchema`, `createProjectSchema`, `updateProjectSchema`
4. **`apps/api/src/routes/projects.ts`** — tidak perlu ubah kalau sudah spread `...input` dari schema
5. **`apps/web/src/pages/projects-page.ts`** — tambah `priority` ke form state
6. **`apps/web/src/pages/projects-page.html`** — tambah `<select>` untuk priority di form

**Mengapa urutan ini penting:**
- Schema DB dulu — tanpa migration, insert akan error
- Shared schema setelah DB — API butuh tipe baru untuk validasi
- Frontend terakhir — bisa deploy API dulu, frontend menyusul tanpa downtime

</details>

---

## Ringkasan

- **Satu seam tenancy** (`middleware/tenant.ts`) mengurus semua auth dan isolasi — semua handler cukup panggil `tenantOf(c)`
- **Isolasi SQL** — setiap query filter dengan `workspace_id`, bukan di level aplikasi
- **Shared schema** di `packages/shared` — API validate dan frontend type-check dari sumber yang sama
- **Plan limits di shared** — API enforce dan UI render, tidak mungkin drift
- **Error contract konsisten** — `{ code, message, issues[] }` — frontend bisa tampilkan langsung tanpa transformasi
- **Audit log** — setiap aksi penting dicatat dengan actor dan target

---

## Selanjutnya

Kamu sekarang paham bagaimana aplikasi Aurelia yang nyata dibangun. Pertanyaan selanjutnya: bagaimana cara menggunakan skill ini untuk mendapat pekerjaan? Di bab berikutnya kita bahas strategi konkret untuk masuk ke pasar freelance via Upwork dengan niche Aurelia.

→ [Bab 10: Aurelia di Dunia Kerja](../bab-10/README.md)
→ [Bab 09b: Production Build & Deploy](../bab-09b/README.md)
