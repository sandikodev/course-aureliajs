# Bab 07 — Dependency Injection: Mengelola Ketergantungan dengan Bersih

> Kalau kamu pernah kerja di Angular, Dependency Injection (DI) sudah familiar. Kalau datang dari React atau Vue, konsep ini mungkin terasa asing tapi akan segera terasa masuk akal. DI adalah cara Aurelia mengelola semua "ketergantungan" antar bagian kode — service, repository, konfigurasi — tanpa harus meneruskannya secara manual dari komponen ke komponen.

---

## Masalah yang Diselesaikan DI

Bayangkan tanpa DI. Setiap komponen yang butuh `ApiService` harus membuat instance-nya sendiri:

```typescript
// ❌ Tanpa DI — setiap komponen buat instance sendiri
export class ProjectsPage {
  private api = new ApiService(); // new setiap kali
}

export class MembersPage {
  private api = new ApiService(); // instance berbeda, state tidak tershare
}
```

Masalahnya:
- Dua instance berbeda = state tidak tershare (misalnya cache atau token auth)
- Susah di-test: tidak bisa inject mock
- Kalau `ApiService` butuh dependency lain, kamu harus urus sendiri

Dengan DI:

```typescript
// ✅ Dengan DI — satu instance tershare
export class ProjectsPage {
  private api = resolve(ApiService); // instance yang sama
}

export class MembersPage {
  private api = resolve(ApiService); // instance yang sama persis
}
```

---

## `resolve()` — Cara Utama Injeksi

`resolve()` adalah fungsi bawaan Aurelia untuk mendapatkan instance dari DI container. Panggil di class field atau di constructor:

```typescript
import { resolve } from 'aurelia';
import { ApiService } from '../services/api';
import { AuthService } from '../services/auth';
import { ToastService } from '../services/toast';

export class ProjectsPage {
  // Semua ini di-inject oleh container — kamu tidak buat instance sendiri
  private api = resolve(ApiService);
  private auth = resolve(AuthService);
  private toast = resolve(ToastService);
}
```

Ini adalah cara yang direkomendasikan untuk Aurelia RC.2 ke atas. Lebih bersih dari constructor injection karena tidak perlu deklarasi parameter panjang.

---

## Singleton vs Transient: Berapa Kali Instance Dibuat?

Secara default, semua service yang kamu `resolve()` adalah **singleton** dalam satu DI container. Artinya satu instance dibuat dan dipakai bersama oleh semua yang resolve service tersebut.

```typescript
// AuthService — satu instance di seluruh aplikasi
export class ProjectsPage {
  private auth = resolve(AuthService); // instance A
}

export class MembersPage {
  private auth = resolve(AuthService); // instance A yang sama!
}

export class AppHeader {
  private auth = resolve(AuthService); // masih instance A
}
```

Kalau `ProjectsPage` login dan mengubah `auth.user`, semua komponen lain yang inject `AuthService` yang sama akan otomatis melihat perubahan tersebut karena mereka memegang referensi ke object yang sama.

**Transient** — instance baru setiap kali di-resolve — jarang dibutuhkan tapi tersedia:

```typescript
import { resolve, newInstanceOf } from 'aurelia';
import { IEventAggregator } from 'aurelia';

export class MyComponent {
  // Instance EventAggregator khusus untuk komponen ini — tidak tershare
  private localEa = resolve(newInstanceOf(IEventAggregator));
}
```

---

## Menulis Service yang Baik

Service adalah class biasa — tidak perlu decorator khusus seperti `@Injectable()` di Angular.

```typescript
// src/services/auth.ts
import type { User, Workspace, Role } from '@aureliajs/shared';

export class AuthService {
  // State yang diobservasi oleh Aurelia
  user: User | null = null;
  workspace: Workspace | null = null;
  role: Role | null = null;

  // Computed property — otomatis reactive
  get isAuthenticated(): boolean {
    return this.user !== null;
  }

  get canManage(): boolean {
    return this.role === 'owner' || this.role === 'admin';
  }

  get initials(): string {
    return (this.user?.name ?? '?')
      .split(' ')
      .map(part => part[0] ?? '')
      .join('')
      .slice(0, 2)
      .toUpperCase();
  }

  apply(session: { user: User | null; workspace: Workspace | null; role: Role | null }): void {
    this.user = session.user;
    this.workspace = session.workspace;
    this.role = session.role;
  }

  clear(): void {
    this.user = null;
    this.workspace = null;
    this.role = null;
  }
}
```

Dan di komponen manapun, bisa langsung bind ke property service:

```html
<!-- Template bisa bind langsung ke property service yang di-inject -->
<span if.bind="auth.isAuthenticated">Halo, ${auth.user.name}!</span>
<span else>Silakan login</span>
```

```typescript
export class AppHeader {
  readonly auth = resolve(AuthService); // readonly karena tidak perlu reassign
}
```

---

## Service yang Butuh Dependency Lain

Service bisa depend ke service lain — DI container akan resolve secara rekursif:

```typescript
// src/services/project.service.ts
import { resolve } from 'aurelia';
import { ApiService } from './api';
import { AuthService } from './auth';
import { ToastService } from './toast';
import type { Project, CreateProjectInput } from '@aureliajs/shared';

export class ProjectService {
  private api = resolve(ApiService);
  private auth = resolve(AuthService);
  private toast = resolve(ToastService);

  projects: Project[] = [];
  isLoading = false;

  async loadAll(): Promise<void> {
    if (!this.auth.isAuthenticated) return;

    this.isLoading = true;
    try {
      const result = await this.api.listProjects();
      this.projects = result.items;
    } catch {
      this.toast.error('Gagal memuat daftar proyek.');
    } finally {
      this.isLoading = false;
    }
  }

  async create(input: CreateProjectInput): Promise<void> {
    const result = await this.api.createProject(input);
    this.projects = [...this.projects, result.project];
    this.toast.success(`Proyek "${result.project.name}" berhasil dibuat.`);
  }
}
```

Komponen cukup resolve service-nya:

```typescript
export class ProjectsPage {
  readonly projectService = resolve(ProjectService);

  async binding() {
    await this.projectService.loadAll();
  }
}
```

```html
<div if.bind="projectService.isLoading">Memuat...</div>
<ul>
  <li repeat.for="project of projectService.projects">${project.name}</li>
</ul>
```

---

## Interface DI: Memisahkan Kontrak dari Implementasi

Untuk code yang lebih mudah di-test, kamu bisa definisikan interface dan daftarkan implementasinya:

```typescript
// src/services/storage.interface.ts
import { DI } from '@aurelia/kernel';

export interface IStorageService {
  get<T>(key: string): T | null;
  set<T>(key: string, value: T): void;
  remove(key: string): void;
}

// Buat token DI yang terikat ke interface
export const IStorageService = DI.createInterface<IStorageService>(
  'IStorageService',
  x => x.singleton(LocalStorageService), // default implementation
);
```

```typescript
// src/services/local-storage.service.ts
export class LocalStorageService implements IStorageService {
  get<T>(key: string): T | null {
    const raw = localStorage.getItem(key);
    if (!raw) return null;
    try {
      return JSON.parse(raw) as T;
    } catch {
      return null;
    }
  }

  set<T>(key: string, value: T): void {
    localStorage.setItem(key, JSON.stringify(value));
  }

  remove(key: string): void {
    localStorage.removeItem(key);
  }
}
```

Pemakaian — inject via interface token, bukan class konkret:

```typescript
import { IStorageService } from '../services/storage.interface';

export class ThemeService {
  private storage = resolve(IStorageService);

  getTheme(): string {
    return this.storage.get<string>('theme') ?? 'light';
  }

  setTheme(theme: string): void {
    this.storage.set('theme', theme);
    document.documentElement.dataset.theme = theme;
  }
}
```

Keuntungan: di test, kamu bisa inject `MockStorageService` yang tidak menyentuh `localStorage`.

---

## Mendaftarkan Service

Service yang kamu buat perlu didaftarkan di DI container. Ada beberapa cara:

### Cara 1: Daftarkan di `main.ts` (Paling Umum)

```typescript
// main.ts
import { AuthService } from './services/auth';
import { ToastService } from './services/toast';
import { ThemeService } from './services/theme';

void Aurelia
  .register(
    RouterConfiguration.customize({ useUrlFragmentHash: false }),
    AuthService,   // ← daftarkan
    ToastService,  // ← daftarkan
    ThemeService,  // ← daftarkan
  )
  .app({ host: document.querySelector('app-root')!, component: App })
  .start();
```

### Cara 2: `static register` di Class Service

```typescript
import { Registration } from '@aurelia/kernel';
import { IContainer } from 'aurelia';

export class MySpecialService {
  static register(container: IContainer) {
    container.register(
      Registration.singleton(MySpecialService, MySpecialService)
    );
  }
}
```

### Cara 3: Daftarkan Langsung di Komponen (Jarang)

Kalau sebuah service hanya relevan untuk subtree komponen tertentu:

```typescript
import { Registration } from '@aurelia/kernel';

@customElement({
  name: 'my-widget',
  template,
  dependencies: [
    Registration.singleton(WidgetLocalService, WidgetLocalService),
  ],
})
export class MyWidget {}
```

---

## Contoh Nyata: Pola API Service dari Aurelia Ops

Proyek referensi menggunakan pendekatan yang elegan — semua API call dibungkus dalam satu objek `api`:

```typescript
// Disederhanakan dari apps/web/src/services/api.ts

// Error class khusus
export class ApiError extends Error {
  constructor(
    readonly status: number,
    readonly code: string,
    message: string,
  ) {
    super(message);
    this.name = 'ApiError';
  }
}

// Fungsi request internal
async function request<T>(path: string, init: RequestInit = {}): Promise<T> {
  const response = await fetch(`/api${path}`, {
    credentials: 'include', // kirim cookie sesi
    headers: { 'content-type': 'application/json', ...init.headers },
    ...init,
  });

  if (!response.ok) {
    const payload = await response.json().catch(() => null);
    throw new ApiError(
      response.status,
      payload?.error?.code ?? 'request_failed',
      payload?.error?.message ?? `Request failed (${response.status})`,
    );
  }

  if (response.status === 204) return undefined as T;
  return response.json() as Promise<T>;
}

// Object API — bukan class, cukup object dengan method
export const api = {
  me: () => request<AuthSession>('/auth/me'),
  login: (input: LoginInput) => request<AuthSession>('/auth/login', {
    method: 'POST',
    body: JSON.stringify(input),
  }),
  listProjects: () => request<PaginatedProjects>('/projects'),
  createProject: (input: CreateProjectInput) => request<{ project: Project }>('/projects', {
    method: 'POST',
    body: JSON.stringify(input),
  }),
  // ... dan seterusnya
};
```

Pola ini tidak pakai class DI untuk `api` sendiri — cukup export object singleton. `AuthService`, `ToastService`, dll yang perlu reactive state barulah menggunakan class yang di-resolve via DI.

---

## Kapan Pakai DI, Kapan Tidak

**Pakai DI (class + resolve) untuk:**
- Service yang perlu menyimpan state reaktif (`user`, `toasts`, `theme`)
- Service yang bergantung ke service lain
- Service yang perlu di-mock saat testing

**Cukup export object/fungsi biasa untuk:**
- Fungsi utility murni (format tanggal, format angka)
- Konstanta konfigurasi
- Fungsi yang tidak butuh state

```typescript
// ✅ Ini tidak perlu DI — fungsi murni, cukup export biasa
export function formatDate(iso: string): string {
  return new Intl.DateTimeFormat('id-ID').format(new Date(iso));
}

export function formatCurrency(cents: number): string {
  return new Intl.NumberFormat('id-ID', {
    style: 'currency',
    currency: 'IDR',
  }).format(cents / 100);
}

// ✅ Ini perlu DI — punya state reaktif
export class CartService {
  items: CartItem[] = [];
  get total() { return this.items.reduce((sum, i) => sum + i.price, 0); }
}
```

---

## HTTP Client: `@aurelia/fetch-client`

Aurelia punya HTTP client bawaan yang membungkus native `fetch` dengan fitur tambahan: interceptors, retry, caching, dan konfigurasi via DI.

### Setup

`@aurelia/fetch-client` sudah termasuk dalam package `aurelia` — tidak perlu install terpisah.

```typescript
import { resolve, newInstanceOf } from 'aurelia';
import { IHttpClient, json } from '@aurelia/fetch-client';

export class ApiService {
  // newInstanceOf = instance terpisah yang bisa dikonfigurasi sendiri
  private http = resolve(newInstanceOf(IHttpClient));

  constructor() {
    this.http.configure(c => c
      .withBaseUrl('/api/')
      .withDefaults({
        credentials: 'same-origin', // kirim cookie sesi
        headers: { Accept: 'application/json' },
      })
      .rejectErrorResponses() // 4xx/5xx jadi throw, bukan resolve
    );
  }

  async getProjects() {
    const res = await this.http.get('projects');
    return res.json();
  }

  async createProject(data: CreateProjectInput) {
    const res = await this.http.post('projects', json(data));
    return res.json();
  }
}
```

### Interceptors

Interceptor memungkinkan kamu hook ke setiap request/response:

```typescript
const authInterceptor = {
  request(request: Request): Request {
    // Tambah auth header sebelum kirim
    const token = localStorage.getItem('token');
    if (token) {
      request.headers.set('Authorization', `Bearer ${token}`);
    }
    return request;
  },

  response(response: Response, request: Request): Response {
    // Log setiap response
    console.log(`${request.method} ${request.url} → ${response.status}`);
    return response;
  },
};

// Daftarkan di configure
this.http.configure(c => c
  .withInterceptor(authInterceptor)
  .withBaseUrl('/api/')
);
```

**Interceptor hooks:**
- `request(request)` — modifikasi request sebelum kirim
- `requestError(error)` — tangani error dari interceptor sebelumnya
- `response(response, request)` — modifikasi response
- `responseError(error, request)` — tangani error response

### Retry

```typescript
import { RetryStrategy } from '@aurelia/fetch-client';

this.http.configure(c => c
  .withBaseUrl('/api/')
  .withRetry({
    maxRetries: 3,
    strategy: RetryStrategy.exponential,
    interval: 2000,
    // Hanya retry kalau server error (5xx)
    doRetry: (res) => res.status >= 500,
  })
);
```

**Aturan penting:** `withRetry` harus menjadi interceptor terakhir yang didaftarkan. Hanya boleh ada satu.

### Cancellation dengan AbortController

```typescript
private controller = new AbortController();

async loadProjects() {
  this.controller.abort(); // batalkan request sebelumnya kalau ada
  this.controller = new AbortController();

  try {
    const res = await this.http.get('projects', {
      signal: this.controller.signal,
    });
    this.projects = await res.json();
  } catch (e) {
    if (e.name !== 'AbortError') throw e;
    // Aborted — abaikan
  }
}

  detaching() {
    this.controller.abort(); // batalkan saat halaman ditutup
  }
```

### Pola dari Aurelia Ops

Aurelia Ops tidak pakai `IHttpClient` — melainkan fungsi `fetch` langsung dengan wrapper sendiri. Ini juga valid dan lebih sederhana untuk banyak kasus:

```typescript
// Disederhanakan dari apps/web/src/services/api.ts
async function request<T>(path: string, init: RequestInit = {}): Promise<T> {
  let response: Response;
  try {
    response = await fetch(`/api${path}`, {
      credentials: 'include',
      headers: { 'content-type': 'application/json', ...init.headers },
      signal: AbortSignal.timeout(8000), // timeout 8 detik
      ...init,
    });
  } catch {
    throw new ApiError(0, 'network_error', 'API tidak terjangkau');
  }

  if (response.status === 204) return undefined as T;

  const payload = await response.json().catch(() => null);

  if (!response.ok) {
    if (response.status === 401) {
      // Auto-redirect ke login
      onUnauthorized?.();
    }
    throw new ApiError(
      response.status,
      payload?.error?.code ?? 'request_failed',
      payload?.error?.message ?? `Request failed (${response.status})`,
      payload?.error?.issues ?? [],
    );
  }

  return payload as T;
}

export const api = {
  listProjects: (query?: ProjectQuery) => {
    const params = new URLSearchParams();
    if (query?.page) params.set('page', String(query.page));
    if (query?.q) params.set('q', query.q);
    const qs = params.toString();
    return request<PaginatedProjects>(`/projects${qs ? `?${qs}` : ''}`);
  },
  createProject: (input: CreateProjectInput) =>
    request<{ project: Project }>('/projects', {
      method: 'POST',
      body: JSON.stringify(input),
    }),
};
```

**Kapan pakai `IHttpClient` vs `fetch` langsung?**
- `IHttpClient` — kalau butuh interceptors, retry, caching terstruktur
- `fetch` langsung — kalau cukup dengan wrapper sederhana dan error handling custom

Keduanya valid. Pilih yang paling sesuai dengan kompleksitas proyek.

---

## Latihan Bab 07

### Latihan 1: Service dengan Error Handling

Buat `CartService` yang:
- Menyimpan array items
- Mempunyai getter `total` (computed)
- Mempunyai method `addItem` dan `removeItem`
- Mengirim event `cart:updated` via EventAggregator saat items berubah

<details>
<summary>Solusi</summary>

```typescript
import { resolve } from 'aurelia';
import { IEventAggregator } from 'aurelia';

export type CartItem = { id: string; name: string; price: number; qty: number };

export class CartService {
  private ea = resolve(IEventAggregator);

  items: CartItem[] = [];

  get total(): number {
    return this.items.reduce((sum, item) => sum + item.price * item.qty, 0);
  }

  get count(): number {
    return this.items.reduce((sum, item) => sum + item.qty, 0);
  }

  addItem(item: CartItem): void {
    const existing = this.items.find(i => i.id === item.id);
    if (existing) {
      existing.qty += item.qty;
      this.items = [...this.items]; // trigger reactivity
    } else {
      this.items = [...this.items, item];
    }
    this.ea.publish('cart:updated', { count: this.count, total: this.total });
  }

  removeItem(id: string): void {
    this.items = this.items.filter(i => i.id !== id);
    this.ea.publish('cart:updated', { count: this.count, total: this.total });
  }

  clear(): void {
    this.items = [];
    this.ea.publish('cart:updated', { count: 0, total: 0 });
  }
}
```

</details>

### Latihan 2: API Service dengan Timeout

Buat `ProductApiService` yang fetch dari `/api/products` dengan timeout 5 detik dan error handling.

<details>
<summary>Solusi</summary>

```typescript
export class ProductApiError extends Error {
  constructor(readonly status: number, message: string) {
    super(message);
    this.name = 'ProductApiError';
  }
}

export class ProductApiService {
  async list(): Promise<Product[]> {
    try {
      const res = await fetch('/api/products', {
        signal: AbortSignal.timeout(5000),
        headers: { Accept: 'application/json' },
      });
      if (!res.ok) {
        throw new ProductApiError(res.status, `Gagal memuat produk (${res.status})`);
      }
      return res.json();
    } catch (e) {
      if (e instanceof ProductApiError) throw e;
      throw new ProductApiError(0, 'Tidak bisa terhubung ke server');
    }
  }

  async create(data: Omit<Product, 'id'>): Promise<Product> {
    const res = await fetch('/api/products', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data),
      signal: AbortSignal.timeout(5000),
    });
    if (!res.ok) {
      const error = await res.json().catch(() => null);
      throw new ProductApiError(res.status, error?.message ?? 'Gagal membuat produk');
    }
    return res.json();
  }
}
```

</details>

---

## Ringkasan

- `resolve()` mendapatkan instance dari DI container — gunakan di class field, bukan di method
- Semua service yang di-resolve adalah singleton secara default — satu instance tershare
- Service adalah class biasa — tidak perlu `@Injectable()` atau decorator khusus
- Daftarkan service di `main.ts` agar tersedia di seluruh aplikasi
- Gunakan `DI.createInterface` + implementasi terpisah kalau mau mudah di-test dengan mock
- `@aurelia/fetch-client` menyediakan HTTP client dengan interceptors, retry, caching
- Interceptors hook ke setiap request/response — auth header, logging, error handling
- `withRetry` harus jadi interceptor terakhir — hanya satu yang diperbolehkan
- `AbortController` untuk batalkan request saat halaman ditutup atau search baru dimulai
- Pola `fetch` langsung dengan wrapper custom juga valid dan lebih sederhana

---

## Selanjutnya

Service dan komponen sudah bisa saling berkomunikasi. Sekarang kita bahas cara menghubungkan semua halaman aplikasi dengan **Routing** — navigasi, parameter URL, guard akses, dan lifecycle halaman.

→ [Bab 08: Routing](../bab-08/README.md)
→ [Bab 07b: Testing dengan Vitest](../bab-07b/README.md)
