# Bab 08 — Routing: Navigasi Antar Halaman

> Routing adalah cara aplikasi berpindah dari satu halaman ke halaman lain. Di Aurelia, routing terintegrasi langsung dengan sistem komponen — bukan modul terpisah seperti di Angular. Bab ini membahas cara setup routing, parameter URL, navigasi programmatik, dan mengamankan halaman yang butuh autentikasi.

---

## Konsep Dasar

Di Aurelia, routing didefinisikan dengan decorator `@route` di root component (biasanya `app.ts`). Router kemudian merender halaman aktif di dalam elemen `<au-viewport>`.

```
Browser URL: /projects
      ↓
Router mencocokkan path 'projects'
      ↓
ProjectsPage di-render di dalam <au-viewport>
      ↓
User melihat halaman Projects
```

---

## Setup Router

Pertama, install package router:

```bash
pnpm add @aurelia/router
```

Daftarkan di `main.ts`:

```typescript
// main.ts
import Aurelia from 'aurelia';
import { RouterConfiguration } from '@aurelia/router';
import { App } from './app';
import { HomePage } from './pages/home';
import { AboutPage } from './pages/about';
import { ProductPage } from './pages/product';

void Aurelia
  .register(
    RouterConfiguration.customize({
      useUrlFragmentHash: false, // pakai pushState, bukan hash (#)
    }),
    // Semua halaman yang dipakai router harus terdaftar di sini
    HomePage,
    AboutPage,
    ProductPage,
  )
  .app({ host: document.querySelector('app-root')!, component: App })
  .start();
```

---

## Mendefinisikan Routes

Gunakan decorator `@route` di root component:

```typescript
// app.ts
import { customElement } from 'aurelia';
import { route } from '@aurelia/router';
import { HomePage } from './pages/home';
import { AboutPage } from './pages/about';
import { ProductPage } from './pages/product';
import { NotFoundPage } from './pages/not-found';
import template from './app.html';

@route({
  routes: [
    // Path array untuk alias ('' dan 'home' keduanya render HomePage)
    { path: ['', 'home'], component: HomePage, title: 'Home' },
    { path: 'about', component: AboutPage, title: 'Tentang' },
    // Parameter URL dengan :id
    { path: 'product/:id', component: ProductPage, title: 'Produk' },
    // Fallback untuk URL yang tidak dikenali
    { path: 'not-found', component: NotFoundPage, title: 'Tidak Ditemukan' },
  ],
  fallback: 'not-found', // redirect ke halaman ini kalau path tidak match
})
@customElement({ name: 'app-root', template })
export class App {}
```

```html
<!-- app.html -->
<nav>
  <a load="route: home">Home</a>
  <a load="route: about">Tentang</a>
</nav>

<!-- Router render halaman aktif di sini -->
<au-viewport></au-viewport>
```

---

## Pola Rekomendasi: Pisahkan Route Config

Untuk aplikasi yang lebih besar, pindahkan konfigurasi route ke file terpisah. Ini pola yang dipakai Aurelia Ops:

```typescript
// src/config/routes.ts
import { HomePage } from '../pages/home';
import { AboutPage } from '../pages/about';
import { LoginPage } from '../pages/login';
import { DashboardPage } from '../pages/dashboard';
import { NotFoundPage } from '../pages/not-found';

// Pisahkan public dan protected secara eksplisit
export const PUBLIC_ROUTES = [
  { path: '',       component: HomePage,  title: 'Home'  },
  { path: 'login',  component: LoginPage, title: 'Login' },
] as const;

export const PROTECTED_ROUTES = [
  { path: 'dashboard', component: DashboardPage, title: 'Dashboard' },
  { path: 'about',     component: AboutPage,     title: 'Tentang'   },
] as const;

// Set path public untuk dipakai di app.ts
export const PUBLIC_ROUTE_PATHS = new Set(
  PUBLIC_ROUTES.map(r => r.path)
);

export function getRouterConfig() {
  return {
    routes: [
      ...PUBLIC_ROUTES,
      ...PROTECTED_ROUTES,
      { path: 'not-found', component: NotFoundPage, title: 'Tidak Ditemukan' },
    ],
    fallback: 'not-found',
  };
}
```

```typescript
// app.ts — jauh lebih bersih
import { route } from '@aurelia/router';
import { getRouterConfig } from './config/routes';

const config = getRouterConfig();

@route(config)
@customElement({ name: 'app-root', template })
export class App {}
```

---

## Navigasi di Template

### Atribut `load`

```html
<!-- Navigasi ke path langsung -->
<a load="route: home">Home</a>
<a load="route: about">Tentang</a>

<!-- Dengan parameter -->
<a load="route: product/42">Lihat Produk #42</a>

<!-- Parameter dinamis dari binding -->
<a load="route: product/${product.id}">${product.name}</a>

<!-- Paksa navigasi dari root (berguna di nested router) -->
<a load="route: dashboard; context.bind: null">Dashboard</a>
```

### Active Class

Router otomatis menambahkan class `is-active` (atau class yang kamu konfigurasi) ke link yang aktif:

```typescript
RouterConfiguration.customize({
  useUrlFragmentHash: false,
  activeClass: 'is-active', // default
})
```

```html
<nav>
  <!-- Link ini dapat class 'is-active' kalau URL-nya /home -->
  <a load="route: home" class="nav-link">Home</a>
  <a load="route: about" class="nav-link">Tentang</a>
</nav>
```

```css
.nav-link.is-active {
  font-weight: bold;
  color: var(--primary);
}
```

---

## Navigasi Programmatik

```typescript
import { resolve } from 'aurelia';
import { IRouter } from '@aurelia/router';

export class LoginPage {
  private router = resolve(IRouter);

  async submit() {
    await this.auth.login(this.email, this.password);

    // Navigasi setelah login berhasil
    await this.router.load('dashboard');
  }

  goToSignup() {
    void this.router.load('signup');
  }

  // Navigasi dengan parameter
  goToProduct(id: string) {
    void this.router.load(`product/${id}`);
  }
}
```

---

## Parameter URL

### Mengambil Parameter di Halaman

Implementasikan `IRouteViewModel` dan gunakan hook `loading()`:

```typescript
import type { IRouteViewModel, Params, RouteNode } from '@aurelia/router';

export class ProductPage implements IRouteViewModel {
  productId = '';
  product: Product | null = null;

  // Dipanggil sebelum halaman ditampilkan
  // Bisa return Promise untuk memblokir render sampai selesai
  async loading(params: Params): Promise<void> {
    this.productId = params.id as string;
    this.product = await this.api.getProduct(this.productId);
  }
}
```

```html
<!-- product.html -->
<div if.bind="product">
  <h1>${product.name}</h1>
  <p>${product.description}</p>
</div>
```

### Query Parameters

```typescript
async loading(params: Params, next: RouteNode): Promise<void> {
  // URL: /products?page=2&status=active
  const page = next.queryParams.get('page') ?? '1';
  const status = next.queryParams.get('status') ?? 'all';

  this.currentPage = parseInt(page, 10);
  this.selectedStatus = status;

  await this.loadProducts();
}
```

---

## Guard: Proteksi Halaman

Guard memungkinkan kamu mencegah navigasi ke halaman tertentu. Kamu bisa redirect ke halaman lain.

```typescript
import { resolve } from 'aurelia';
import { AuthService } from '../services/auth';
import type { IRouteViewModel, Params, RouteNode } from '@aurelia/router';

export class DashboardPage implements IRouteViewModel {
  private auth = resolve(AuthService);

  // Dipanggil sebelum halaman dimuat
  // Return false atau redirect path untuk memblokir
  canLoad(params: Params, next: RouteNode): boolean | string {
    if (!this.auth.isAuthenticated) {
      // Redirect ke login dengan return URL
      const currentPath = window.location.pathname;
      return `login?returnUrl=${encodeURIComponent(currentPath)}`;
    }
    return true; // izinkan navigasi
  }
}
```

### Guard Berdasarkan Role

```typescript
export class AdminPage implements IRouteViewModel {
  private auth = resolve(AuthService);

  canLoad(): boolean | string {
    if (!this.auth.isAuthenticated) {
      return 'login';
    }
    if (!this.auth.canManage) {
      // User login tapi tidak punya akses admin
      return 'dashboard'; // redirect ke dashboard biasa
    }
    return true;
  }
}
```

### Mencegah Navigasi Keluar (Unsaved Changes)

```typescript
export class EditPage implements IRouteViewModel {
  hasUnsavedChanges = false;

  // Dipanggil saat user coba navigasi keluar
  canUnload(): boolean {
    if (this.hasUnsavedChanges) {
      return confirm('Ada perubahan yang belum disimpan. Yakin ingin keluar?');
    }
    return true;
  }
}
```

---

## Pola Boot: Auth Check di App Root

Ini pola yang dipakai Aurelia Ops untuk menangani auth saat aplikasi pertama dibuka:

```typescript
// app.ts
import { resolve } from 'aurelia';
import { IRouter, IRouterEvents } from '@aurelia/router';
import { AuthService } from './services/auth';
import { PUBLIC_ROUTE_PATHS } from './config/routes';

@route(getRouterConfig())
@customElement({ name: 'app-root', template })
export class App {
  private router = resolve(IRouter);
  private events = resolve(IRouterEvents);
  readonly auth = resolve(AuthService);

  // True saat auth check pertama berjalan
  // Template hanya tampilkan spinner selama ini true
  loadingAuth = true;

  // Apakah shell (sidebar + header) perlu ditampilkan
  showShell = false;

  private updateShell(): void {
    const path = window.location.pathname.replace(/^\//, '');
    this.showShell = this.auth.isAuthenticated && !PUBLIC_ROUTE_PATHS.has(path);
  }

  async attached(): Promise<void> {
    // 1. Cek auth dari server
    const authenticated = await this.auth.refresh();

    const currentPath = window.location.pathname.replace(/^\//, '');
    const onPublicPath = PUBLIC_ROUTE_PATHS.has(currentPath) || currentPath === '';

    // 2. Redirect kalau perlu
    if (authenticated && onPublicPath) {
      await this.router.load('dashboard');
    } else if (!authenticated && !onPublicPath) {
      await this.router.load('login');
    }

    // 3. Update tampilan shell
    this.updateShell();

    // 4. Sembunyikan spinner
    this.loadingAuth = false;

    // 5. Listen perubahan route untuk update shell
    this.events.subscribe('au:router:navigation-end', () => {
      this.updateShell();
    });
  }
}
```

```html
<!-- app.html -->
<!-- Spinner selama auth check pertama -->
<div if.bind="loadingAuth" class="loading-screen">
  <div class="spinner"></div>
</div>

<!-- Authenticated layout dengan sidebar -->
<template if.bind="!loadingAuth && showShell">
  <aside><!-- sidebar navigasi --></aside>
  <main><au-viewport></au-viewport></main>
</template>

<!-- Public layout tanpa sidebar -->
<template if.bind="!loadingAuth && !showShell">
  <au-viewport></au-viewport>
</template>
```

Pola ini memastikan:
- Tidak ada flicker (konten tidak muncul sebelum auth selesai dicek)
- User yang sudah login langsung ke dashboard
- User yang belum login langsung ke halaman login
- Sidebar hanya muncul di halaman protected

---

## Lifecycle Hook Routing Lengkap

```typescript
export class MyPage implements IRouteViewModel {

  // 1. Boleh masuk halaman ini?
  // Return false atau path string untuk redirect
  canLoad(params: Params, next: RouteNode): boolean | string | Promise<boolean | string> {
    return this.auth.isAuthenticated || 'login';
  }

  // 2. Load data sebelum render
  // Return Promise untuk blocking load — render ditunda sampai selesai
  async loading(params: Params, next: RouteNode): Promise<void> {
    this.data = await this.api.getData(params.id as string);
  }

  // 3. Halaman sudah aktif penuh
  loaded(params: Params): void {
    analytics.track('page_view', { page: 'my-page', id: params.id });
  }

  // 4. Boleh keluar halaman ini?
  canUnload(): boolean {
    return !this.hasUnsavedChanges || confirm('Keluar tanpa simpan?');
  }

  // 5. Cleanup sebelum navigasi pergi
  unloading(): void {
    this.cleanup();
  }
}
```

---

## Ringkasan

- Router di-setup di `main.ts` dengan `RouterConfiguration.customize()`
- Routes didefinisikan di root component dengan `@route()`
- Pisahkan `PUBLIC_ROUTES` dan `PROTECTED_ROUTES` untuk kejelasan
- Halaman yang dipakai router **harus** didaftarkan di `main.ts`
- `<au-viewport>` adalah slot tempat router merender halaman aktif
- `<a load="route: nama">` untuk navigasi di template
- `IRouter.load()` untuk navigasi programmatik
- `canLoad()` untuk guard — return `false` atau path string untuk redirect
- `loading()` untuk fetch data sebelum halaman ditampilkan — bisa async
- Pola boot auth: `loadingAuth = true` → cek auth → redirect kalau perlu → `loadingAuth = false`

---

## Nested Routing (Child Routes)

Route bisa punya child routes — halaman di dalam halaman. Berguna untuk layout yang punya sub-navigasi (misal: Settings → Profile, Settings → Billing, Settings → Security).

```typescript
// src/pages/settings.ts
import { route } from '@aurelia/router';
import { customElement } from 'aurelia';
import template from './settings.html';

@route({
  routes: [
    { path: ['', 'profile'], component: import('./profile'), title: 'Profil' },
    { path: 'billing', component: import('./billing'), title: 'Billing' },
    { path: 'security', component: import('./security'), title: 'Keamanan' },
  ],
})
@customElement({ name: 'settings-page', template })
export class SettingsPage {}
```

```html
<!-- settings.html -->
<div class="settings-layout">
  <nav class="settings-nav">
    <a load="route: profile">Profil</a>
    <a load="route: billing">Billing</a>
    <a load="route: security">Keamanan</a>
  </nav>
  <div class="settings-content">
    <!-- Child viewport — render sub-halaman -->
    <au-viewport></au-viewport>
  </div>
</div>
```

Di root config, daftarkan parent route:

```typescript
// main.ts atau config/routes.ts
{ path: 'settings', component: import('../pages/settings'), title: 'Settings' },
```

Sekarang URL `/settings/profile` akan render `SettingsPage` → `ProfilePage` di dalamnya.

```
URL: /settings/profile

Root router → SettingsPage (render di <au-viewport> root)
  ↓
SettingsPage router → ProfilePage (render di <au-viewport> child)
```

**Navigasi relatif di dalam child:**

```html
<!-- Dalam settings.html, link relatif -->
<a load="../billing">Billing</a>  <!-- naik ke parent, lalu ke billing -->
```

---

## 404 Handling yang Benar

Ada dua level 404 yang perlu ditangani:

### Level 1: Route tidak dikenali (Client-side)

```typescript
@route({
  routes: [
    { path: '', component: import('./home'), title: 'Home' },
    { path: 'about', component: import('./about'), title: 'About' },
    // Fallback route — tangkap semua yang tidak match
    { path: 'not-found', component: import('./not-found'), title: 'Tidak Ditemukan' },
  ],
  fallback: 'not-found', // ← kalau tidak ada yang match, arahkan ke sini
})
export class MyApp {}
```

### Level 2: Data tidak ditemukan di handler

```typescript
export class ProductDetail implements IRouteViewModel {
  product: Product | null = null;

  async canLoad(params: Params): Promise<boolean | string> {
    this.product = await this.api.getProduct(params.id as string);

    if (!this.product) {
      return 'not-found'; // redirect ke halaman 404
    }

    return true;
  }
}
```

### Level 3: Server-side fallback (Production)

Di production, kalau user langsung buka `/products` di browser (bukan via link SPA), server harus mengembalikan `index.html` agar router bisa resolve path.

**nginx:**
```nginx
location / {
  try_files $uri /index.html;
}
```

**Cloudflare Workers / Hono:**
```typescript
// SPA fallback — semua route yang tidak match API return index.html
app.get('*', (c) => {
  return c.html(indexHtml);
});
```

**Express:**
```typescript
app.get('*', (req, res) => {
  res.sendFile(path.join(__dirname, 'dist/index.html'));
});
```

Tanpa ini, refresh di URL selain `/` akan return 404 dari server.

---

## Route Metadata

Kamu bisa menambahkan data custom ke setiap route untuk keperluan guard, analytics, atau layout switching:

```typescript
@route({
  routes: [
    {
      path: 'dashboard',
      component: import('./dashboard'),
      title: 'Dashboard',
      data: { requiresAuth: true, layout: 'app', noindex: true },
    },
    {
      path: 'login',
      component: import('./login'),
      title: 'Login',
      data: { requiresAuth: false, layout: 'public' },
    },
  ],
})
```

Akses metadata di guard:

```typescript
canLoad(params: Params, next: RouteNode): boolean | string {
  if (next.data?.requiresAuth && !this.auth.isAuthenticated) {
    return 'login';
  }
  return true;
}
```

---

## Latihan Bab 08

### Latihan 1: Nested Routing untuk Settings

Buat struktur route settings dengan 3 sub-halaman (profile, notifications, security).

<details>
<summary>Solusi</summary>

```typescript
// src/pages/settings/settings.ts
import { route, customElement } from 'aurelia';
import template from './settings.html';

@route({
  routes: [
    { path: ['', 'profile'], component: import('./profile'), title: 'Profil' },
    { path: 'notifications', component: import('./notifications'), title: 'Notifikasi' },
    { path: 'security', component: import('./security'), title: 'Keamanan' },
  ],
})
@customElement({ name: 'settings-page', template })
export class SettingsPage {}
```

```html
<!-- settings.html -->
<div class="flex">
  <nav>
    <a load="route: profile">Profil</a>
    <a load="route: notifications">Notifikasi</a>
    <a load="route: security">Keamanan</a>
  </nav>
  <au-viewport></au-viewport>
</div>
```

```typescript
// src/pages/settings/profile.ts
export class ProfileSettings {
  name = '';
  email = '';
}
```

```html
<!-- profile.html -->
<h2>Profil</h2>
<input value.bind="name" placeholder="Nama">
<input value.bind="email" placeholder="Email">
```

Di root config:
```typescript
{ path: 'settings', component: import('../pages/settings/settings'), title: 'Settings' },
```

</details>

### Latihan 2: Guard dengan Redirect + Return URL

Buat guard yang redirect ke login dan simpan URL asli untuk kembali setelah login.

<details>
<summary>Solusi</summary>

```typescript
// src/pages/dashboard/dashboard.ts
import { resolve } from 'aurelia';
import { AuthService } from '../../services/auth';
import type { IRouteViewModel, Params, RouteNode } from '@aurelia/router';

export class DashboardPage implements IRouteViewModel {
  private auth = resolve(AuthService);

  canLoad(_params: Params, next: RouteNode): boolean | string {
    if (!this.auth.isAuthenticated) {
      // Simpan URL yang diminta untuk kembali setelah login
      const returnUrl = encodeURIComponent(next.path);
      return `login?returnUrl=${returnUrl}`;
    }
    return true;
  }
}
```

```typescript
// src/pages/login/login.ts
import { resolve } from 'aurelia';
import { IRouter } from '@aurelia/router';
import { AuthService } from '../../services/auth';
import type { IRouteViewModel, Params, RouteNode } from '@aurelia/router';

export class LoginPage implements IRouteViewModel {
  private router = resolve(IRouter);
  private auth = resolve(AuthService);

  email = '';
  password = '';
  returnUrl = 'dashboard';

  loading(params: Params, next: RouteNode): void {
    // Baca returnUrl dari query params
    const url = next.queryParams.get('returnUrl');
    if (url) this.returnUrl = url;
  }

  async submit() {
    await this.auth.login(this.email, this.password);
    await this.router.load(this.returnUrl);
  }
}
```

</details>

---

## Ringkasan

- Router di-setup di `main.ts` dengan `RouterConfiguration.customize()`
- Routes didefinisikan di root component dengan `@route()`
- Pisahkan `PUBLIC_ROUTES` dan `PROTECTED_ROUTES` untuk kejelasan
- Halaman yang dipakai router **harus** didaftarkan di `main.ts`
- `<au-viewport>` adalah slot tempat router merender halaman aktif
- `<a load="route: nama">` untuk navigasi di template
- `IRouter.load()` untuk navigasi programmatik
- `canLoad()` untuk guard — return `false` atau path string untuk redirect
- `loading()` untuk fetch data sebelum halaman ditampilkan — bisa async
- Nested routing: `@route` di child component + `<au-viewport>` di child template
- 404: `fallback` di route config + `canLoad` return path + server-side SPA fallback
- Route metadata via `data: { ... }` — akses di guard via `next.data`
- Pola boot auth: `loadingAuth = true` → cek auth → redirect kalau perlu → `loadingAuth = false`

---

## Selanjutnya

Kamu sekarang punya semua fondasi untuk membangun aplikasi Aurelia nyata. Di bab berikutnya kita rakit semuanya dalam sebuah studi kasus — membangun dashboard SaaS sederhana menggunakan pola-pola dari Aurelia Ops.

→ [Bab 09: Studi Kasus — Dashboard SaaS](../bab-09/README.md)
→ [Bab 09b: Production Build & Deploy](../bab-09b/README.md)
