# Bab 04 — Setup Project

> Sudah cukup teori. Di bab ini kita setup project Aurelia 2 dari nol, pahami setiap file yang dihasilkan, dan jalankan aplikasi pertama kamu. Di akhir bab ini kamu punya project yang berjalan dan siap dikembangkan.

---

## Yang Kamu Butuhkan

Pastikan sudah terinstall:

- **Node.js 20 atau 22** (cek: `node --version`)
- **pnpm 9+** atau npm (cek: `pnpm --version`)

Kalau belum punya pnpm, install dengan:

```bash
npm install -g pnpm
```

---

## Cara Paling Cepat: `npx makes aurelia`

Aurelia punya generator resmi yang membuat project lengkap dengan pilihan konfigurasi. Jalankan:

```bash
npx makes aurelia my-first-app
```

Kamu akan disambut wizard interaktif. Untuk bab ini, pilih opsi ini:

```
? Project type:          app
? Bundler:               vite
? Transpiler:            typescript
? CSS preprocessor:      css (plain)
? Unit tests:            no-unit-tests
? Sample code:           app-with-router
```

Atau kalau mau langsung tanpa wizard (silent mode):

```bash
npx makes aurelia my-first-app -s app,vite,typescript,no-unit-tests,app-with-router
```

Setelah selesai:

```bash
cd my-first-app
pnpm install   # generator tidak install otomatis
pnpm dev
```

Buka `http://localhost:9000`. Kamu akan melihat app Aurelia berjalan.

---

## Struktur File yang Dihasilkan

```
my-first-app/
├── src/
│   ├── main.ts          ← Entry point: bootstrap Aurelia
│   ├── my-app.ts        ← Root component (class)
│   ├── my-app.html      ← Root component (template)
│   ├── resource.d.ts    ← Type declarations untuk .html dan .css
│   └── routes/          ← Folder halaman (dari pilihan app-with-router)
│       ├── home/
│       │   ├── home.ts
│       │   └── home.html
│       └── about/
│           ├── about.ts
│           └── about.html
├── index.html           ← HTML shell
├── vite.config.ts       ← Konfigurasi Vite
├── tsconfig.json        ← Konfigurasi TypeScript
└── package.json
```

---

## Memahami Setiap File Kunci

### `index.html` — Shell Aplikasi

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>My First App</title>
    <!-- PENTING: base href wajib ada untuk deep link bekerja -->
    <base href="/" />
  </head>
  <body>
    <!-- Aurelia akan render di dalam tag ini -->
    <my-app></my-app>
    <!-- Entry point TypeScript -->
    <script type="module" src="/src/main.ts"></script>
  </body>
</html>
```

Tag `<base href="/">` wajib ada. Tanpanya, kalau kamu buka langsung URL seperti `/about` di browser, router tidak bisa resolve path dengan benar.

### `src/main.ts` — Bootstrap Aurelia

```typescript
// src/main.ts
import Aurelia from 'aurelia';
import { RouterConfiguration } from '@aurelia/router';
import { MyApp } from './my-app';

void Aurelia
  .register(
    // Daftarkan router dengan opsi clean URL (tanpa #)
    RouterConfiguration.customize({
      useUrlFragmentHash: false,
    }),
    // Daftarkan service dan komponen global di sini
  )
  .app({
    host: document.querySelector('my-app')!,
    component: MyApp,
  })
  .start();
```

Pola ini mirip `bootstrapApplication` di Angular standalone. Semua konfigurasi global ada di sini.

**Satu hal penting:** halaman yang di-navigate oleh router **harus didaftarkan di sini** atau diimport secara eksplisit. Kalau tidak, router akan error `AUR3174: cannot find component`.

```typescript
// Kalau punya halaman yang dipakai router, daftarkan:
import { HomePage } from './routes/home/home';
import { AboutPage } from './routes/about/about';

void Aurelia
  .register(
    RouterConfiguration.customize({ useUrlFragmentHash: false }),
    HomePage,   // ← daftarkan di sini
    AboutPage,  // ← daftarkan di sini
  )
  .app({ host: document.querySelector('my-app')!, component: MyApp })
  .start();
```

### `src/my-app.ts` — Root Component

```typescript
// src/my-app.ts
import { customElement } from 'aurelia';
import { route } from '@aurelia/router';
import template from './my-app.html';

@route({
  routes: [
    { path: ['', 'home'], component: import('./routes/home/home'), title: 'Home' },
    { path: 'about', component: import('./routes/about/about'), title: 'About' },
  ],
})
@customElement({ name: 'my-app', template })
export class MyApp {}
```

Root component adalah tempat routing didefinisikan. Decorator `@route` menempel di class root app, bukan di modul terpisah seperti Angular.

### `src/my-app.html` — Root Template

```html
<!-- src/my-app.html -->
<nav>
  <a load="route: home">Home</a>
  <a load="route: about">About</a>
</nav>

<!-- Router render halaman aktif di sini -->
<au-viewport></au-viewport>
```

`<au-viewport>` adalah slot tempat router merender halaman aktif. Setara dengan `<router-outlet>` di Angular atau `<RouterOutlet>` di React Router.

### `tsconfig.json` — Konfigurasi TypeScript

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "skipLibCheck": true,
    "experimentalDecorators": false,
    "useDefineForClassFields": true
  }
}
```

**Dua baris terpenting:**

- `"experimentalDecorators": false` — Aurelia 2 menggunakan stage-3 decorators (standar TC39), **bukan** legacy decorators. Ini berbeda dari Angular dan tutorial Aurelia lama yang masih pakai `experimentalDecorators: true`. Pastikan ini `false`.
- `"useDefineForClassFields": true` — diperlukan agar stage-3 decorators bekerja benar.

Ini sering jadi sumber kebingungan. Kalau kamu pernah pakai Aurelia 1 atau tutorial lama, ingat: **RC.2 ke atas pakai `experimentalDecorators: false`**.

### `vite.config.ts` — Konfigurasi Vite

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import aurelia from '@aurelia/vite-plugin';

export default defineConfig({
  plugins: [
    aurelia(),
  ],
});
```

Plugin Aurelia untuk Vite menangani:
- Kompilasi template `.html` menjadi rendering instructions
- Transformasi stage-3 decorators
- Hot Module Replacement (HMR) untuk komponen

---

## Membuat Komponen Pertama

Sekarang kita buat komponen sendiri. Buat dua file baru:

```typescript
// src/components/greeting.ts
import { bindable } from 'aurelia';

export class Greeting {
  @bindable name = 'Dunia';
}
```

```html
<!-- src/components/greeting.html -->
<p>Halo, ${name}! Selamat datang di Aurelia.</p>
```

Gunakan di halaman home:

```typescript
// src/routes/home/home.ts
import { Greeting } from '../../components/greeting';

// Daftarkan komponen yang dipakai di halaman ini
export class Home {
  static dependencies = [Greeting];

  guestName = 'Sandikodev';
}
```

```html
<!-- src/routes/home/home.html -->
<greeting name.bind="guestName"></greeting>
```

Jalankan `pnpm dev`, buka `http://localhost:9000`, kamu akan melihat:

```
Halo, Sandikodev! Selamat datang di Aurelia.
```

Coba ubah nilai `guestName` di `home.ts`, simpan — Vite akan hot reload dan teks langsung berubah.

---

## Perbedaan Penting: Import Template Eksplisit

Kamu mungkin memperhatikan di contoh `my-app.ts` tadi ada ini:

```typescript
import template from './my-app.html';

@customElement({ name: 'my-app', template })
export class MyApp {}
```

Ini karena under Vite, Aurelia butuh **import template eksplisit** ketika kamu menggunakan `@customElement` decorator. Plugin Aurelia mendeteksi ini dan mengkompilasi template HTML menjadi rendering instructions.

Tapi kalau kamu **tidak** pakai `@customElement` dan hanya mengandalkan konvensi nama file (file `.ts` dan `.html` di folder yang sama, nama yang sama), maka kamu **tidak perlu** import eksplisit. Generator biasanya sudah handle ini dengan benar.

**Aturan praktis:**
- Pakai `@customElement({ template })` kalau kamu butuh kustomisasi (nama tag custom, dependencies eksplisit)
- Tanpa decorator, konvensi nama file otomatis bekerja — file `greeting.ts` + `greeting.html` = komponen `<greeting>`

---

## Menambahkan Tailwind CSS (Opsional)

Kalau kamu ingin pakai Tailwind, install dulu:

```bash
pnpm add -D tailwindcss @tailwindcss/vite
```

Update `vite.config.ts`:

```typescript
import { defineConfig } from 'vite';
import aurelia from '@aurelia/vite-plugin';
import tailwindcss from '@tailwindcss/vite';

export default defineConfig({
  plugins: [
    aurelia(),
    tailwindcss(), // ← tambahkan ini
  ],
});
```

Buat `src/styles/main.css`:

```css
@import "tailwindcss";
```

Import di `main.ts`:

```typescript
import './styles/main.css'; // ← tambahkan di paling atas
import Aurelia from 'aurelia';
// ...
```

Sekarang kamu bisa pakai class Tailwind langsung di template:

```html
<p class="text-lg font-semibold text-blue-600">
  Halo, ${name}!
</p>
```

---

## Troubleshooting Setup Umum

**Error: `AUR3174: cannot find component`**

Router tidak bisa menemukan komponen halaman yang di-navigate. Solusi: pastikan semua halaman yang dipakai router didaftarkan di `main.ts`.

```typescript
// main.ts — pastikan semua halaman terdaftar
import { HomePage } from './routes/home/home';
void Aurelia.register(
  RouterConfiguration.customize({ useUrlFragmentHash: false }),
  HomePage, // ← ini yang sering lupa
).app(...).start();
```

**Error: TypeScript tentang decorator**

Periksa `tsconfig.json`. Harus ada:
```json
"experimentalDecorators": false,
"useDefineForClassFields": true
```

Jika kamu mengcopy `tsconfig.json` dari tutorial lama atau proyek Angular, kemungkinan besar `experimentalDecorators: true` di sana — itu yang harus diganti.

**Deep link tidak bekerja saat refresh (`/about` jadi 404)**

Dua hal yang perlu dicek:
1. `<base href="/">` ada di `index.html`
2. Server development (atau production server) dikonfigurasi untuk fallback ke `index.html`

Untuk Vite dev server, ini sudah otomatis. Untuk production, perlu konfigurasi nginx/server yang sesuai.

---

## Ringkasan

- `npx makes aurelia` adalah cara tercepat membuat project baru
- Tiga file terpenting: `index.html` (shell), `main.ts` (bootstrap), `my-app.ts` (routing)
- `experimentalDecorators: false` adalah wajib untuk Aurelia RC.2 — berbeda dari Angular dan tutorial lama
- Halaman yang dipakai router harus didaftarkan di `main.ts`
- Konvensi nama file (`greeting.ts` + `greeting.html`) bekerja otomatis tanpa decorator

---

## Latihan Bab 04

### Latihan: Buat Halaman Baru Lengkap

Buat halaman "About" yang:
1. Punya halaman sendiri dengan konvensi nama file
2. Terdaftar di route config
3. Terdaftar di `main.ts`
4. Punya teks sederhana dan tombol navigasi balik ke home

<details>
<summary>Solusi</summary>

```typescript
// src/routes/about/about.ts
export class About {
  message = 'Tentang aplikasi ini';
}
```

```html
<!-- src/routes/about/about.html -->
<div>
  <h1>${message}</h1>
  <p>Aplikasi ini dibangun dengan Aurelia 2.</p>
  <a load="route: home">← Kembali ke Home</a>
</div>
```

Di `my-app.ts`, tambahkan route:
```typescript
@route({
  routes: [
    { path: ['', 'home'], component: import('./routes/home/home'), title: 'Home' },
    { path: 'about', component: import('./routes/about/about'), title: 'About' }, // ← baru
  ],
})
export class MyApp {}
```

Di `main.ts`, daftarkan:
```typescript
import { AboutPage } from './routes/about/about';

void Aurelia.register(
  RouterConfiguration.customize({ useUrlFragmentHash: false }),
  AboutPage, // ← daftarkan
).app(...).start();
```

</details>

---

## Selanjutnya

Project sudah jalan. Sekarang kita masuk ke bagian yang paling sering dipakai setiap hari: **binding** — cara Aurelia menghubungkan data di class TypeScript dengan tampilan di template HTML.

→ [Bab 05: Binding](../bab-05/README.md)
→ [Bab 05b: Forms & Validation](../bab-05b/README.md)
