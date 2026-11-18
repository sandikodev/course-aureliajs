# Bab 11 — Portfolio Builder: Aurelia Ops sebagai Bukti Kerja

> Skill tanpa bukti tidak menjual. Bab ini membahas cara menggunakan Aurelia Ops — proyek yang menjadi referensi course ini — sebagai portfolio yang bisa diklik, diverifikasi, dan dipercaya oleh klien potensial. Ini bukan soal "membuat portfolio yang bagus", tapi soal membuat bukti yang spesifik untuk pertanyaan yang akan ditanya klien.

---

## Masalah Portfolio Generik

Portfolio kebanyakan developer terlihat seperti ini:
- Todo app dengan React
- Weather app yang fetch dari API publik
- Clone UI dari Dribbble

Klien yang butuh engineer Aurelia tidak akan terkesan. Yang mereka tanya adalah:
- "Pernah handle multi-tenancy?"
- "Pernah integrasi Stripe?"
- "Bisa debug codebase Aurelia yang sudah ada?"
- "Paham class-based architecture dan DI?"

Aurelia Ops menjawab semua pertanyaan itu. Kamu tinggal tahu cara menyajikannya.

---

## Yang Sudah Ada di Aurelia Ops

Sebelum menyajikannya ke klien, pahami dulu apa yang ada di dalamnya:

| Fitur | File Kunci | Yang Bisa Dibuktikan |
|-------|-----------|----------------------|
| Multi-tenant isolation | `apps/api/src/middleware/tenant.ts` | Isolasi data antar workspace di SQL level |
| Session auth (Argon2id) | `apps/api/src/routes/auth.ts` | Auth yang benar, bukan localStorage token |
| RBAC (owner/admin/member) | `apps/api/src/middleware/tenant.ts` | `requireRole()` per endpoint |
| Projects CRUD + pagination | `apps/api/src/routes/projects.ts` | Server-side filter, sort, 1000 baris data |
| Invite via magic link | `apps/api/src/routes/invites.ts` | Email tidak harus ada sebelum di-invite |
| Stripe Checkout | `apps/api/src/routes/billing.ts` | Hosted checkout, tidak ada card di server |
| Plan limits enforcement | `packages/shared/src/schemas.ts` | `PLAN_LIMITS` satu tempat, API + UI sinkron |
| Audit log | `apps/api/src/db/schema.ts` | Setiap aksi tercatat |
| Shared Zod schemas | `packages/shared/src/schemas.ts` | Satu kontrak, dua consumer |
| UI kit ported dari shadcn | `apps/web/src/ui/` | 19 komponen, design system lengkap |
| i18n EN + ID | `apps/web/src/locales/` | 250+ key, disimpan di workspace |
| Playwright E2E tests | `e2e/` | Smoke, billing, team, a11y |

---

## Langkah 1: Fork dan Jalankan Sendiri

Langkah pertama adalah punya versi yang berjalan atas namamu sendiri.

```bash
# Fork repo di GitHub ke akun kamu
# Lalu clone fork tersebut

git clone git@github.com:username-kamu/aureliajs.git
cd aureliajs

# Setup
cp .env.example .env
pnpm install
pnpm db:up
pnpm db:migrate
pnpm db:seed

# Jalankan
pnpm dev
```

Buka `http://localhost:4600`, login dengan `demo@aureliajs.dev` / `demo-password-123`.

Pastikan semua fitur berjalan di lokal dulu sebelum deploy ke mana pun.

---

## Langkah 2: Buat Perubahan Nyata

Jangan sajikan fork yang 100% identik dengan yang asli. Buat minimal 2–3 perubahan yang bisa kamu jelaskan. Perubahan tidak harus besar — yang penting ada trail bahwa kamu paham kodenya.

**Ide perubahan yang mudah dan terlihat:**

1. **Tambahkan field baru ke form Projects** (misalnya `priority: low/medium/high`)
   - Tambah kolom ke schema Drizzle
   - Tambah field ke `createProjectSchema` di shared
   - Tambah input di `projects-page.html`
   - Tampilkan di tabel

2. **Tambahkan export CSV yang lebih kaya** — misal sertakan total budget workspace di baris pertama

3. **Ganti UI** — modifikasi warna, font, atau layout. Buat versi "dark mode first" atau tema berbeda.

4. **Tambahkan halaman baru** — misalnya halaman "Activity" yang menampilkan audit log dengan filter.

5. **Tulis 1 unit test** untuk service atau utility function yang belum ada test-nya.

Kalau kamu sudah bikin perubahan ini, kamu bisa bilang ke klien: "Saya fork proyek ini, tambahkan X, dan ini commit-nya."

---

## Langkah 3: Deploy ke URL Publik

Demo yang bisa diklik jauh lebih kuat dari screenshot.

### Opsi A: Deploy ke Railway (Paling Mudah)

Railway bisa deploy aplikasi Node.js + PostgreSQL dari GitHub dalam hitungan menit.

1. Buat akun di [railway.app](https://railway.app)
2. "New Project" → "Deploy from GitHub repo"
3. Pilih fork Aurelia Ops kamu
4. Railway otomatis deteksi Dockerfile dan deploy

Variabel environment yang perlu diset di Railway:
```
DATABASE_URL=postgresql://...  (Railway sediakan otomatis)
SESSION_SECRET=random-string-panjang
```

Untuk Stripe, cukup set ke test mode dulu:
```
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...
```

### Opsi B: Vercel (Frontend) + Railway/Fly.io (API)

Kalau mau deploy terpisah:
- Frontend (`apps/web`) → Vercel, Cloudflare Pages
- API (`apps/api`) → Railway, Fly.io, Render

Setup environment variable `VITE_API_BASE_URL` di frontend yang menunjuk ke URL API.

### Opsi C: VPS dengan Docker

```bash
# Di server kamu
git clone https://github.com/username-kamu/aureliajs.git
cp .env.example .env
# edit .env dengan nilai production

docker compose -f docker-compose.prod.yml up -d
```

---

## Langkah 4: Tulis Case Study Satu Halaman

Ini yang paling sering dilewati dan paling penting. Case study bukan "daftar fitur" — tapi narasi tentang keputusan yang kamu buat dan mengapa.

**Struktur case study yang efektif:**

```markdown
# Aurelia Ops — SaaS Admin Dashboard

## Masalah yang Diselesaikan
Tidak ada contoh publik dashboard SaaS production-ready dengan
Aurelia 2. Developer yang inherit codebase Aurelia enterprise
tidak punya referensi untuk menjawab pertanyaan: bagaimana multi-
tenancy, auth, billing, dan role management dilakukan dengan benar?

## Keputusan Teknis Penting

### 1. Satu Seam Tenancy
Semua isolasi tenant ada di satu file: middleware/tenant.ts.
Handler tidak pernah baca cookie atau cari workspace sendiri.
Ini membuat audit dan perubahan auth menjadi operasi satu file.

### 2. Shared Schema sebagai Kontrak
Plan limits, validation rules, dan tipe data ada di packages/shared.
API dan frontend tidak mungkin drift karena keduanya import dari sumber
yang sama.

### 3. Stripe Hosted Checkout
Card data tidak pernah menyentuh server. Tidak ada PCI scope.
API hanya mengembalikan URL, browser yang pergi ke Stripe.

## Yang Sengaja Tidak Dibuat
- Tidak ada email server: invite link ditampilkan di UI
- Tidak ada real-time: polling cukup untuk dashboard admin

## Cara Verifikasi
- Demo: [url]
- Login: demo@... / demo-password-123
- Repo: github.com/username-kamu/aureliajs
```

---

## Langkah 5: Hubungkan ke Profil Upwork

Di profil Upwork, tambahkan portfolio item dengan:

**Judul:** "Aurelia 2 Multi-Tenant SaaS Dashboard — Production Architecture"

**Deskripsi (200 kata):**
```
Dashboard admin SaaS yang dibangun dengan Aurelia 2, Hono, Drizzle
ORM, dan PostgreSQL. Fitur: multi-tenant dengan isolasi SQL, session
auth (Argon2id), Stripe billing, team management, dan audit log.

Yang membuat ini bukan sekedar clone tutorial:

Multi-tenancy diimplementasikan melalui satu middleware yang
menangani semua resolusi workspace — tidak ada handler yang
perlu baca session atau cari workspace sendiri. Isolasi data
dilakukan di level SQL query, bukan di level aplikasi.

Plan limits (jumlah project per tier) didefinisikan di package
shared yang di-import oleh API untuk enforcement dan oleh
frontend untuk display — mereka tidak mungkin tidak sinkron.

1000 proyek di-seed untuk membuktikan pagination dan filter
benar-benar bekerja, bukan hanya tampil bagus dengan 5 data.

Stack: Aurelia 2 (rc.2) + Vite 8 + Tailwind 4 + Hono + Drizzle
+ PostgreSQL + Stripe + TypeScript strict

Demo live: [url] | Repo: github.com/username-kamu/aureliajs
```

---

## Cara Menjawab Pertanyaan Teknis di Interview

Saat klien mulai bertanya tentang portfolio ini, kamu bisa menjawab dengan spesifik karena kamu sudah paham kodenya:

**"Bagaimana kamu handle multi-tenancy?"**
> Di setiap request yang masuk ke endpoint protected, ada satu middleware yang membaca session cookie, memverifikasi hash-nya di database, lalu mencari workspace yang terikat ke session itu. Semua handler cukup panggil `tenantOf(c)` — tidak ada yang baca cookie sendiri. Kalau cara kerja tenancy perlu diubah, cukup edit satu file.

**"Bagaimana kamu prevent user dari akses data tenant lain?"**
> Di level SQL. Setiap query yang fetch data project, member, atau billing selalu ada `WHERE workspace_id = ?` yang di-inject dari middleware, bukan dari client. Frontend tidak bisa kirim workspaceId yang berbeda karena server tidak terima parameter itu dari client.

**"Pernah kerja dengan Stripe?"**
> Di proyek ini saya pakai Stripe Hosted Checkout — card data tidak pernah menyentuh server saya. API hanya buat checkout session dan return URL-nya. Webhook di-verify dengan signature HMAC sebelum diproses. Saya juga implement idempotency — event yang sama tidak akan diproses dua kali karena Stripe event ID disimpan sebagai primary key di tabel `stripe_events`.

---

## Ringkasan

- Portfolio yang bisa diklik dan dijawab dengan spesifik jauh lebih kuat dari daftar skill
- Fork Aurelia Ops, buat minimal 2–3 perubahan nyata, deploy ke URL publik
- Tulis case study satu halaman: masalah → keputusan → verifikasi
- Di profil Upwork, tampilkan portfolio dengan konteks teknis — bukan hanya screenshot
- Siap jawab pertanyaan teknis tentang multi-tenancy, auth, Stripe, dan arsitektur

---

## Penutup Course

Kamu sudah menyelesaikan course Aurelia 2. Ini yang sudah kamu pelajari:

1. **Kenapa Aurelia** — sejarah, filosofi, posisi di ekosistem, tiga klaster paradigma
2. **Mental model** — class-based, direct-DOM, bukan re-render
3. **Perbandingan Angular** — apa yang sama, apa yang berbeda, Angular 22 lebih terbukti di banyak dimensi
4. **Setup project** — `npx makes aurelia`, konfigurasi TypeScript yang benar
5. **Binding** — semua jenis binding, value converters, binding behaviors, containerless
6. **Forms & Validation** — semua input types, `@aurelia/validation`, error display
7. **Komponen** — `@bindable`, `<au-slot>`, custom attributes, HTML-only, EventAggregator
8. **Dependency Injection** — `resolve()`, singleton, interface, HTTP client, service pattern
9. **Testing** — `createFixture`, assertions, mocks, routing tests
10. **Routing** — `@route`, nested routing, guard, 404 handling, pola boot auth
11. **Studi kasus** — multi-tenant SaaS, shared types, error contract
12. **Production deploy** — Cloudflare, Vercel, VPS/Nginx, Docker
13. **Strategi karier** — positioning Upwork, proposal, pricing, 3 fakta jujur
14. **Portfolio** — cara menyajikan Aurelia Ops sebagai bukti kerja

---

## Latihan Bab 11

### Latihan: Tulis Case Study Satu Halaman

Berdasarkan panduan di bab ini, tulis case study Aurelia Ops versi kamu sendiri — maksimal satu halaman, dengan format: masalah → keputusan → trade-off → cara verifikasi.

Gunakan pertanyaan ini sebagai kerangka:
1. Apa masalah yang diselesaikan oleh proyek ini?
2. Satu keputusan teknis apa yang paling kamu pahami (bisa dari Bab 09)?
3. Apa yang sengaja tidak dikerjakan (dan kenapa)?
4. Bagaimana klien bisa memverifikasi klaim kamu dalam 10 menit?

<details>
<summary>Panduan jawaban</summary>

**Masalah:** Tidak ada contoh publik dashboard SaaS multi-tenant dengan Aurelia 2. Developer yang inherit codebase Aurelia enterprise tidak punya referensi untuk arsitektur auth, billing, dan tenancy yang benar.

**Keputusan teknis:**
- Satu middleware untuk semua isolasi tenant — ubah auth = ubah satu file
- Plan limits di package shared — API enforce dan UI render dari sumber yang sama, tidak mungkin drift

**Yang sengaja tidak dibuat:**
- Tidak ada email server — invite link ditampilkan di UI, bukan dikirim via email
- Stripe dipakai Hosted Checkout — card tidak pernah menyentuh server, tidak ada PCI scope

**Cara verifikasi (< 10 menit):**
1. Buka demo URL
2. Login dengan demo@... / demo-password-123
3. Coba buat project sampai limit
4. Lihat pesan error yang muncul
5. Buka GitHub, lihat `middleware/tenant.ts` — satu file, semua isolasi
6. Lihat `packages/shared/src/schemas.ts` — PLAN_LIMITS didefinisikan di sini

</details>

---

Langkah selanjutnya yang paling penting: **deploy demo** dan **kirim 10 proposal pertama**. Bukan membaca bab lagi, bukan menyempurnakan portfolio, bukan menunggu. Kirim 10 proposal.

Satu proposal yang dikirim lebih berharga dari 10 yang belum pernah dikirim.

> Untuk memahami posisi Aurelia dibanding React, Angular, Svelte, dan framework lain secara mendalam: baca `FRAMEWORK_POSITIONING.md`.

---

*Semoga berhasil. Kalau ada pertanyaan, hubungi @sandikodev di TikTok atau Facebook.*
