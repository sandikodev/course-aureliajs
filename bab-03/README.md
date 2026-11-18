# Bab 03 — Untuk Ex-Angular: Persamaan dan Perbedaan

> Bab ini untuk kamu yang sudah kenal Angular atau AngularJS. Kalau belum, tidak masalah — langsung lompat ke Bab 04. Tapi kalau kamu punya background Angular, bab ini akan menghemat banyak waktu karena kita petakan secara langsung apa yang sama, apa yang berbeda, dan apa yang lebih baik.

> **Catatan singkat sebelum mulai:** Aurelia dan Angular berada di klaster paradigma yang sama — keduanya Enterprise Class OOP. Itulah kenapa perbandingan di bab ini bermakna. Vue, React, dan Svelte hidup di klaster yang berbeda sehingga perbandingan dengan mereka adalah perbandingan lintas-budaya. Analisis lengkapnya ada di [`FRAMEWORK_POSITIONING.md`](../FRAMEWORK_POSITIONING.md).

---

## Mengapa Aurelia Terasa Familiar bagi Developer Angular

Seperti yang sudah dijelaskan di Bab 1, Aurelia dan Angular punya akar yang sama. Rob Eisenberg ikut merancang Angular 2 di Google sebelum akhirnya keluar dan membuat Aurelia. Hasilnya: banyak konsep yang sama, tapi implementasinya lebih bersih.

Kalau kamu pernah kerja dengan Angular dan langsung buka kode Aurelia, reaksi pertamamu biasanya: *"Ini Angular, tapi... lebih ringan."*

---

## Perbandingan Langsung: Sintaks

### Deklarasi Komponen

**Angular:**
```typescript
import { Component, Input } from '@angular/core';

@Component({
  selector: 'app-user-card',
  templateUrl: './user-card.component.html',
  styleUrls: ['./user-card.component.css']
})
export class UserCardComponent {
  @Input() name = '';
  @Input() email = '';
}
```

**Aurelia:**
```typescript
import { customElement, bindable } from 'aurelia';

@customElement('user-card')
export class UserCard {
  @bindable name = '';
  @bindable email = '';
}
```

Perbedaan langsung terlihat:
- `@Component` → `@customElement` (atau bahkan tanpa decorator kalau kamu pakai konvensi nama file)
- `@Input()` → `@bindable`
- Tidak ada `templateUrl` terpisah — Aurelia pakai konvensi: file `user-card.ts` otomatis berpasangan dengan `user-card.html`
- Tidak ada `styleUrls` — CSS di-handle terpisah

### Template Binding

**Angular:**
```html
<!-- Property binding -->
<input [value]="userName">

<!-- Event binding -->
<button (click)="save()">Simpan</button>

<!-- Two-way binding -->
<input [(ngModel)]="userName">

<!-- Conditional -->
<div *ngIf="isLoggedIn">Selamat datang!</div>

<!-- Loop -->
<li *ngFor="let item of items">{{ item.name }}</li>
```

**Aurelia:**
```html
<!-- Property binding -->
<input value.bind="userName">

<!-- Event binding -->
<button click.trigger="save()">Simpan</button>

<!-- Two-way binding (otomatis pada input) -->
<input value.bind="userName">

<!-- Conditional -->
<div if.bind="isLoggedIn">Selamat datang!</div>

<!-- Loop -->
<li repeat.for="item of items">${item.name}</li>
```

Pola `.bind`, `.trigger`, `.for` lebih eksplisit dan lebih mudah dibaca karena menggunakan kata kerja bahasa Inggris daripada simbol kurung siku yang membingungkan.

### Dependency Injection

**Angular:**
```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';

@Injectable({ providedIn: 'root' })
export class UserService {
  constructor(private http: HttpClient) {}

  getUser(id: string) {
    return this.http.get(`/api/users/${id}`);
  }
}

// Komponen yang menggunakan:
@Component({ ... })
export class UserComponent {
  constructor(private userService: UserService) {}
}
```

**Aurelia:**
```typescript
import { resolve } from 'aurelia';
import { IHttpClient } from '@aurelia/fetch-client';

export class UserService {
  private http = resolve(IHttpClient);

  async getUser(id: string) {
    const res = await this.http.get(`/api/users/${id}`);
    return res.json();
  }
}

// Komponen yang menggunakan:
export class UserComponent {
  private userService = resolve(UserService);
}
```

Perbedaan penting:
- Tidak ada `@Injectable()` — Aurelia tidak membutuhkan decorator ini
- `resolve()` dipanggil di class field, bukan di constructor parameter
- Tidak ada `constructor(private service: Service)` — lebih bersih

---

## Perbedaan Arsitektur Utama

### 1. Tidak Ada NgModule

Ini yang paling sering bikin developer Angular heran. Di Angular, setiap komponen harus didaftarkan ke `NgModule`:

```typescript
// Angular — kamu harus daftarkan semuanya
@NgModule({
  declarations: [UserCardComponent, UserListComponent, UserFormComponent],
  imports: [CommonModule, ReactiveFormsModule],
  exports: [UserCardComponent]
})
export class UserModule { }
```

Di Aurelia, tidak ada konsep ini. Komponen terdaftar dengan cara:

```typescript
// Aurelia — 3 cara, pilih yang paling sesuai
// 1. Import lokal di template (paling umum)
// user-list.html:
// <import from="./user-card"></import>

// 2. dependencies di @customElement
@customElement({
  name: 'user-list',
  dependencies: [UserCard]
})
export class UserList {}

// 3. Global di main.ts
Aurelia
  .register(UserCard, UserList)
  .app(MyApp)
  .start();
```

Tidak ada barrel module, tidak ada NgModule, tidak ada module imports. Lebih sederhana.

### 2. Tidak Ada Zone.js = Tidak Ada "Magic" Change Detection

Di Angular, kamu mungkin pernah mengalami situasi di mana perubahan state tidak ter-detect dan kamu harus memanggil `ChangeDetectorRef.detectChanges()`. Ini karena Zone.js tidak selalu bisa mendeteksi perubahan dengan benar, terutama di kasus edge.

Di Aurelia, tidak ada Zone.js. Observasi dilakukan secara eksplisit melalui getter/setter. Kalau sesuatu tidak update, penyebabnya jelas: kamu tidak mengubah properti yang diobservasi.

```typescript
// Ini TIDAK akan memperbarui DOM di Aurelia
this.items[0].name = 'Baru'; // Modifikasi nested object langsung

// Ini yang benar — assign ulang array
this.items = this.items.map((item, i) =>
  i === 0 ? { ...item, name: 'Baru' } : item
);

// Atau pakai metode array yang diobservasi
this.items.splice(0, 1, { ...this.items[0], name: 'Baru' });
```

Perilaku ini sebenarnya lebih *predictable* daripada Angular — kamu tahu persis kapan update terjadi.

### 3. Routing: Lebih Deklaratif

**Angular:**
```typescript
const routes: Routes = [
  { path: '', component: HomeComponent },
  { path: 'users', component: UserListComponent },
  { path: 'users/:id', component: UserDetailComponent },
  { path: '**', component: NotFoundComponent }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

**Aurelia:**
```typescript
import { route } from '@aurelia/router';
import { HomeComponent } from './home';
import { UserListComponent } from './user-list';

@route({
  routes: [
    { path: '', component: HomeComponent, title: 'Home' },
    { path: 'users', component: UserListComponent, title: 'Users' },
    { path: 'users/:id', component: import('./user-detail'), title: 'User' },
    { path: 'not-found', component: import('./not-found') },
  ],
  fallback: 'not-found',
})
export class MyApp {}
```

Routing di Aurelia adalah bagian dari component tree, bukan modul terpisah. Dan mendukung lazy loading dengan `import()` secara native.

### 4. Lifecycle Hooks: Nama Berbeda

| Angular | Aurelia | Kapan |
|---------|---------|-------|
| `constructor` | `constructor` | Injeksi service |
| `ngOnInit` | `binding()` | Setup awal, bisa return Promise |
| — | `bound()` | Setelah binding aktif, ref tersedia |
| `ngAfterViewInit` | `attached()` | DOM sudah ada, init library |
| `ngOnChanges` | `fooChanged()` | Setelah `@bindable foo` berubah |
| `ngOnDestroy` | `detaching()` / `unbinding()` | Cleanup |

Perbedaan penting: di Aurelia `binding()` bisa return Promise untuk memblokir render sampai data tersedia. Ini sangat berguna untuk halaman yang butuh data sebelum bisa ditampilkan.

---

## Apa yang Lebih Baik di Aurelia

Ini bukan soal fanboy — ini evaluasi teknis jujur:

| Aspek | Angular | Aurelia | Pemenang |
|-------|---------|---------|----------|
| Bundle size (minimal) | ~60–120 KB | ~30 KB | Aurelia |
| Performa rendering | Incremental DOM | Direct DOM | Aurelia |
| Learning curve | Tinggi (NgModule, etc.) | Sedang | Aurelia |
| Ekosistem & komunitas | Sangat besar | Kecil | Angular |
| Dukungan enterprise | Google backed | Community | Angular |
| Boilerplate per komponen | Tinggi | Rendah | Aurelia |
| Testability | Baik | Sangat baik | Aurelia |
| TypeScript standard compliance | Partial (zone.js) | Penuh | Aurelia |

**Kesimpulan praktis:** Kalau kamu bisa Angular, kamu bisa produktif di Aurelia dalam 1–2 hari.

---

## Realita: Angular Jauh Lebih Terbukti

Sebelum melanjutkan, kita harus jujur soal kematangan kedua framework. Ini data per November 2026:

| Dimensi | Angular 22 | Aurelia 2 |
|---------|------------|-----------|
| Versi | 22.2.6 (stabil, major releases reguler) | 2.0.0-rc.2 (release candidate, belum stable) |
| Backing | Google + komunitas besar | Komunitas kecil |
| Signals | Native sejak Angular 16 (`signal()`) | Native (ObserverLocator) |
| SSR | Built-in, matang | Community PoC (`@hono/aurelia-ssr`) |
| CLI | `ng` — sangat lengkap (scaffold, generate, serve, test, deploy) | `npx makes aurelia` — minimal |
| Component library | Angular Material, PrimeNG, NG-Zorro | Tidak ada (port manual) |
| DevTools | Angular DevTools (browser extension) | Tidak ada |
| Docs | Sangat lengkap, terawat, update reguler | Cukup, tapi ada gap |
| Job market | Ratusan job aktif | 15–30 job aktif |
| Bukti produksi | Ribuan enterprise di seluruh dunia | Belum ada bukti skala besar |

**Kesimpulan yang jujur:** Angular lebih terbukti di hampir semua dimensi kecuali ukuran bundle dan simplisitas kode. Aurelia menang di cleanliness, tapi cleanliness tanpa bukti produksi skala adalah hipotesis, bukan fakta.

Ini bukan alasan untuk tidak belajar Aurelia. Ini adalah realita yang harus kamu tahu supaya bisa berbicara jujur ke klien. Klien enterprise yang tanya "kenapa Aurelia bukan Angular?" berhak dapat jawaban yang jujur, bukan sales pitch.

---

## Apa yang Lebih Baik di Angular

Jujur ada beberapa hal di mana Angular lebih unggul untuk saat ini:

1. **Ekosistem component library** — Angular Material, PrimeNG, NG-Zorro, dll. Aurelia tidak punya padanan yang setara
2. **Angular DevTools** — tooling debug yang sangat baik, Aurelia kurang di sini
3. **Hiring pool** — lebih mudah cari developer Angular
4. **Signals (Angular 16+)** — Angular sudah evolusi ke fine-grained reactivity yang setara Aurelia. Angular 22 punya `signal()`, `computed()`, `effect()` yang setara dengan ObserverLocator Aurelia. Keunggulan "tidak ada Zone.js" yang dulu jadi alasan utama pilih Aurelia, sekarang sudah bisa di-disable di Angular modern (`zoneless` mode)
5. **Dokumentasi** — Angular docs jauh lebih lengkap dan terawat
6. **Stability** — Angular sudah 22 major releases. Aurelia masih RC. Untuk enterprise yang butuh jaminan LTS, ini perbedaan yang signifikan.
7. **Ecosystem maturity** — ng-bootstrap, Nx monorepo support, schematics untuk auto-generate code, Universal SSR, dll. Aurelia tidak punya padanan untuk semua ini.

Ini bukan alasan untuk tidak belajar Aurelia — tapi jujur harus disebutkan. Posisi Aurelia bukan "lebih baik dari Angular" tapi "alternatif yang lebih ringan untuk tim yang sudah paham pola Angular".

---

## Praktik: Konversi Komponen Angular ke Aurelia

Coba konversi komponen Angular sederhana ini ke Aurelia:

```typescript
// ANGULAR — user-badge.component.ts
import { Component, Input } from '@angular/core';

@Component({
  selector: 'app-user-badge',
  template: `
    <div [class]="'badge badge-' + role">
      {{ name }}
    </div>
  `
})
export class UserBadgeComponent {
  @Input() name = '';
  @Input() role: 'admin' | 'member' = 'member';
}
```

<details>
<summary>Lihat jawaban Aurelia</summary>

```typescript
// AURELIA — user-badge.ts
import { bindable } from 'aurelia';

export class UserBadge {
  @bindable name = '';
  @bindable role: 'admin' | 'member' = 'member';
}
```

```html
<!-- user-badge.html -->
<div class="badge badge-${role}">
  ${name}
</div>
```

Yang hilang:
- `@Component` decorator (tidak perlu kalau pakai konvensi nama file)
- `selector` (otomatis dari nama file: `user-badge`)
- Inline template (pisah ke file `.html`)
- `[class]=` syntax → `class="${role}"` yang lebih natural

</details>

---

## Ringkasan

- Aurelia dan Angular punya DNA yang sama — developer Angular bisa produktif sangat cepat
- Sintaks binding lebih eksplisit: `.bind`, `.trigger`, `.for` vs `[]`, `()`, `*ng`
- Tidak ada NgModule, tidak ada Zone.js, tidak ada `@Injectable()`
- DI via `resolve()` di class field, bukan constructor injection
- Lifecycle hooks punya nama berbeda tapi konsep sama
- Aurelia unggul di performa dan simplisitas, Angular unggul di ekosistem, tooling, dan kematangan
- **Angular 22 lebih terbukti dari Aurelia RC.2** — ini fakta, bukan kritik. Posisi Aurelia adalah niche spesialis, bukan pengganti Angular

> Untuk analisis lengkap tiga klaster paradigma frontend (kenapa Aurelia tidak bisa dibandingkan dengan React/Svelte) dan perbandingan Angular vs Aurelia yang lebih detail: baca `FRAMEWORK_POSITIONING.md`.

---

## Selanjutnya

Cukup teori perbandingan. Sekarang saatnya menulis kode sungguhan. Di bab berikutnya kita setup project Aurelia dari nol dengan `npx makes aurelia`.

→ [Bab 04: Setup Project](../bab-04/README.md)
