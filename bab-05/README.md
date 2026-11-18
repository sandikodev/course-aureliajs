# Bab 05 — Binding: Jembatan antara Data dan Tampilan

> Binding adalah fitur yang paling sering kamu pakai di Aurelia. Setiap kali kamu menghubungkan data dari class TypeScript ke template HTML — atau sebaliknya — kamu sedang menggunakan binding. Bab ini membahas semua jenis binding yang ada, kapan pakai yang mana, dan beberapa jebakan yang sering dijumpai pemula.

---

## Konsep Dasar: Aliran Data

Sebelum masuk ke sintaks, pahami dulu arah aliran data:

```
Class TypeScript  ──────────────────→  Template HTML
(View Model)         to-view            (View)

Class TypeScript  ←──────────────────  Template HTML
(View Model)         from-view          (View)

Class TypeScript  ←─────────────────→  Template HTML
(View Model)         two-way            (View)
```

Dipetakan ke sintaks Aurelia:

```
Arah aliran          Sintaks              Kapan dipakai
─────────────────────────────────────────────────────────
Class → Template    ${nilai}             Interpolasi teks
                    prop.to-view         Atribut satu arah
                    prop.bind            Otomatis (non-form)

Template → Class    prop.from-view       Output komponen
                    ref="elRef"          Referensi elemen DOM

Dua arah            prop.bind            Otomatis (form input)
                    prop.two-way         Eksplisit dua arah

Event               click.trigger        DOM events
                    submit.trigger:prevent  Form submit
```

Aurelia punya binding yang eksplisit untuk setiap arah ini.

---

## 1. Interpolasi: `${ekspresi}`

Yang paling sederhana — tampilkan nilai dari view model langsung di teks:

```html
<!-- greeting.html -->
<p>Halo, ${name}!</p>
<p>Kamu punya ${items.length} item.</p>
<p>Total: ${price * quantity} rupiah.</p>
```

```typescript
// greeting.ts
export class Greeting {
  name = 'Sandikodev';
  items = ['a', 'b', 'c'];
  price = 50000;
  quantity = 3;
}
```

Interpolasi otomatis `to-view` — perubahan di class langsung terlihat di template. Template tidak bisa mengubah nilai balik ke class melalui interpolasi.

---

## 2. `.bind` — Otomatis Pilih Mode

`.bind` adalah binding paling umum. Aurelia otomatis memilih mode yang tepat berdasarkan elemen:

```html
<!-- Pada elemen non-form: to-view -->
<span title.bind="tooltipText">Hover me</span>
<img src.bind="avatarUrl" alt.bind="userName">
<div class.bind="isActive ? 'active' : 'inactive'">

<!-- Pada elemen form: two-way otomatis -->
<input type="text" value.bind="email">
<textarea value.bind="bio"></textarea>
<input type="checkbox" checked.bind="agreeTerms">
<select value.bind="selectedRole">
```

Kalau kamu tidak yakin pakai yang mana, mulai dengan `.bind`. Aurelia akan memilihkan mode yang masuk akal.

---

## 3. `.to-view` — Satu Arah: Class → Template

Data mengalir dari class ke template. Template tidak bisa mengubah nilai balik.

```html
<!-- Cocok untuk data yang hanya ditampilkan, tidak diinput user -->
<p textcontent.to-view="description"></p>
<span>${formattedDate}</span>
```

Sebetulnya `${ekspresi}` sudah `to-view`. Kamu jarang perlu menulis `.to-view` secara eksplisit kecuali untuk binding attribute yang bukan interpolasi.

---

## 4. `.two-way` — Dua Arah: Class ↔ Template

Data mengalir dua arah. Cocok untuk input form:

```html
<input value.two-way="searchQuery">
```

Ketika user mengetik, `searchQuery` di class langsung berubah. Ketika class mengubah `searchQuery`, input field juga berubah.

**Kapan perlu eksplisit `.two-way` vs cukup `.bind`?**

Untuk elemen standard HTML (`<input>`, `<select>`, `<textarea>`), `.bind` sudah otomatis two-way. Kamu perlu `.two-way` secara eksplisit kalau:
- Binding ke custom element yang perlu dua arah
- Kamu ingin kode lebih eksplisit dan mudah dibaca

---

## 5. `.from-view` — Satu Arah: Template → Class

Data mengalir dari template ke class. Jarang dipakai, tapi berguna untuk:

```html
<!-- Mendapatkan reference elemen DOM -->
<canvas ref="myCanvas"></canvas>

<!-- Atau binding dari custom element yang hanya output -->
<date-picker value.from-view="selectedDate"></date-picker>
```

---

## 6. `.trigger` — Event Binding

Untuk menangani event DOM:

```html
<button click.trigger="save()">Simpan</button>
<input input.trigger="onSearch($event)">
<form submit.trigger:prevent="submitForm()">
```

**Penting: `:prevent`**

Modifier `:prevent` memanggil `event.preventDefault()` **sebelum** handler kamu dijalankan. Ini sangat penting untuk form submission — tanpa `:prevent`, browser akan reload halaman.

```html
<!-- ❌ Tanpa :prevent — halaman reload setelah submit -->
<form submit.trigger="submitForm()">

<!-- ✅ Dengan :prevent — browser tidak reload -->
<form submit.trigger:prevent="submitForm()">
```

```typescript
// submitForm tidak perlu panggil event.preventDefault() secara manual
async submitForm() {
  await this.api.save(this.formData);
}
```

**Mendapatkan event object:**

```html
<input input.trigger="handleInput($event)">
```

```typescript
handleInput(event: Event) {
  const input = event.target as HTMLInputElement;
  this.query = input.value;
}
```

---

## 7. `if.bind` dan `show.bind` — Kondisional

```html
<!-- if.bind: elemen DIHAPUS dari DOM kalau false -->
<div if.bind="isLoggedIn">
  Selamat datang!
</div>
<div else>
  Silakan login.
</div>

<!-- show.bind: elemen TETAP di DOM, hanya visibility diubah -->
<div show.bind="isLoading">
  Memuat...
</div>
```

**Kapan pakai `if.bind` vs `show.bind`?**

- `if.bind`: Kalau elemen tidak perlu ada di DOM sama sekali (komponen mahal, form yang tidak relevan)
- `show.bind`: Kalau elemen perlu tetap di DOM (animasi, state yang ingin dipertahankan)

**`switch.bind`** untuk kondisi banyak:

```html
<template switch.bind="status">
  <span case="active">Aktif</span>
  <span case="inactive">Nonaktif</span>
  <span case="pending">Menunggu</span>
  <span default-case>Tidak diketahui</span>
</template>
```

---

## 8. `repeat.for` — Loop

```html
<!-- Loop array sederhana -->
<ul>
  <li repeat.for="item of items">${item.name}</li>
</ul>

<!-- Dengan index -->
<ul>
  <li repeat.for="item of items">
    ${$index + 1}. ${item.name}
  </li>
</ul>

<!-- Dengan key untuk performa (penting untuk list dinamis) -->
<ul>
  <li repeat.for="item of items; key: id">${item.name}</li>
</ul>
```

**Mengapa `key` penting?**

Tanpa `key`, kalau kamu mengubah urutan atau menambah item di tengah array, Aurelia perlu membangun ulang semua elemen DOM. Dengan `key`, Aurelia tahu elemen mana yang perlu diupdate, dipindahkan, atau dibuat baru — jauh lebih efisien.

**Hati-hati dengan mutasi array:**

```typescript
// ❌ Ini TIDAK memperbarui DOM
this.items[0] = { id: 1, name: 'Baru' };

// ✅ Ini yang benar — replace array
this.items = [{ id: 1, name: 'Baru' }, ...this.items.slice(1)];

// ✅ Atau pakai metode array yang diobservasi
this.items.splice(0, 1, { id: 1, name: 'Baru' });
```

---

## 9. `class.bind` dan `style.bind`

```html
<!-- Toggle class tunggal -->
<div active.class="isActive">Konten</div>
<div loading.class="isLoading">Konten</div>

<!-- Ekspresi class -->
<div class="${isActive ? 'bg-blue-500' : 'bg-gray-200'} rounded p-4">

<!-- Object style -->
<div style.bind="{ color: textColor, fontSize: fontSize + 'px' }">
```

---

## 10. `ref` — Mendapatkan Referensi Elemen DOM

```html
<input type="text" ref="nameInput">
<canvas ref="myCanvas"></canvas>

<!-- Untuk custom element, dapatkan view model-nya -->
<my-component component.ref="myComponentVm"></my-component>
```

```typescript
export class MyPage {
  nameInput!: HTMLInputElement;
  myCanvas!: HTMLCanvasElement;

  // ⚠️ ref HANYA tersedia setelah bound() — tidak di constructor atau binding()
  bound() {
    this.nameInput.focus(); // ✅ Aman di sini
  }

  attached() {
    const ctx = this.myCanvas.getContext('2d'); // ✅ Aman di sini
  }
}
```

---

## 11. Binding Behavior: Memodifikasi Cara Kerja Binding

Binding behavior mengubah *perilaku* binding tanpa mengubah nilai. Sintaksnya pakai `&`:

```html
<!-- Debounce: tunggu 500ms setelah user berhenti mengetik -->
<input value.bind="searchQuery & debounce:500">

<!-- Throttle: update maksimal sekali per 300ms -->
<input value.bind="data & throttle:300">

<!-- One-time: hanya bind sekali, tidak update lagi -->
<span>${staticValue & oneTime}</span>
```

**Contoh nyata — search dengan debounce:**

```typescript
export class SearchPage {
  searchQuery = '';

  // Dipanggil setelah debounce 500ms
  async performSearch() {
    if (!this.searchQuery.trim()) return;
    this.results = await this.api.search(this.searchQuery);
  }
}
```

```html
<input
  type="search"
  value.bind="searchQuery & debounce:500"
  input.trigger="performSearch()">
```

---

## Contoh Nyata dari Aurelia Ops

Berikut bagian dari halaman Projects di proyek referensi — perhatikan berbagai jenis binding yang dipakai sekaligus:

```html
<!-- Dari apps/web/src/pages/projects-page.html -->

<!-- Search input dengan two-way dan event handler -->
<input
  type="search"
  value.two-way="q"
  input.trigger="onSearchInput()"
  placeholder="Search by project or owner…"
/>

<!-- Select dengan binding dan event -->
<ui-select
  value.two-way="status"
  options.bind="statusOptions"
  change.trigger="onFilterChange()"
  aria-label.bind="copy['projects.filterStatus']"
></ui-select>

<!-- Button dengan click trigger -->
<ui-button click.trigger="openCreate()">
  New Project
</ui-button>
```

Dan di class-nya:

```typescript
// Dari apps/web/src/pages/projects-page.ts (disederhanakan)
export class ProjectsPage {
  q = '';           // ← dua arah dengan search input
  status = 'all';   // ← dua arah dengan select filter
  projects: Project[] = [];

  onSearchInput() {
    // Dipanggil setiap kali user mengetik di search
    this.debouncedSearch();
  }

  onFilterChange() {
    // Dipanggil saat filter status berubah
    void this.loadProjects();
  }
}
```

---

---

## 12. Value Converters: Transformasi Data di Template

Value converter mengubah cara data ditampilkan tanpa mengubah nilai aslinya. Sintaksnya pakai pipe `|`.

```typescript
// src/resources/currency.ts
import { valueConverter } from 'aurelia';

@valueConverter('currency')
export class CurrencyValueConverter {
  toView(cents: number, currencyCode = 'IDR'): string {
    return new Intl.NumberFormat('id-ID', {
      style: 'currency',
      currency: currencyCode,
    }).format(cents / 100);
  }
}
```

Pemakaian di template:

```html
<!-- Import lokal -->
<import from="../resources/currency"></import>

<!-- Pakai -->
<span>${project.budgetCents | currency}</span>
<span>${project.budgetCents | currency:'USD'}</span>

<!-- Chaining — output converter pertama jadi input kedua -->
<span>${project.budgetCents | currency | uppercase}</span>
```

**Contoh nyata dari Aurelia Ops — format cents ke rupiah:**

```typescript
// apps/web/src/lib/utils.ts (disederhanakan)
export function formatCents(cents: number): string {
  return new Intl.NumberFormat('id-ID', {
    style: 'currency',
    currency: 'IDR',
    maximumFractionDigits: 0,
  }).format(cents / 100);
}
```

```html
<!-- Di template -->
<td>${project.budgetCents | formatCents}</td>
<!-- Output: "Rp 50.000.000" -->
```

**Converter lain yang sering dipakai:**

```typescript
// Date formatter
@valueConverter('date')
export class DateValueConverter {
  toView(iso: string, format: Intl.DateTimeFormatOptions = {}): string {
    return new Intl.DateTimeFormat('id-ID', format).format(new Date(iso));
  }
}

// Truncate text
@valueConverter('truncate')
export class TruncateValueConverter {
  toView(text: string, length = 50): string {
    return text.length > length ? text.slice(0, length) + '…' : text;
  }
}

// Sort array
@valueConverter('sort')
export class SortValueConverter {
  toView(items: any[], property: string, direction: 'asc' | 'desc' = 'asc'): any[] {
    const sorted = [...items].sort((a, b) => {
      const av = a[property], bv = b[property];
      return av < bv ? -1 : av > bv ? 1 : 0;
    });
    return direction === 'desc' ? sorted.reverse() : sorted;
  }
}
```

```html
<span>${project.createdAt | date:{ dateStyle: 'long' }}</span>
<p>${project.description | truncate:100}</p>
<li repeat.for="item of items | sort:'name':'asc'">${item.name}</li>
```

Daftarkan converter secara global di `main.ts` kalau dipakai di banyak halaman:

```typescript
import { CurrencyValueConverter } from './resources/currency';

void Aurelia.register(CurrencyValueConverter).app(...).start();
```

---

## 13. Binding Behaviors: Modifikasi Cara Kerja Binding

Binding behavior mengubah perilaku binding di runtime. Sintaksnya pakai `&`.

Sudah dibahas `& debounce` dan `& throttle` di atas. Berikut yang lain:

```html
<!-- Signal: refresh binding secara manual (misal setelah locale ganti) -->
<span>${price | currency & signal:'refresh-prices'}</span>
```

```typescript
import { ISignaler, resolve } from 'aurelia';

export class PriceList {
  private signaler = resolve(ISignaler);

  refreshPrices() {
    // Semua binding dengan & signal:'refresh-prices' akan di-evaluasi ulang
    this.signaler.dispatch('refresh-prices');
  }
}
```

**Force binding mode:**

```html
<!-- Paksa one-time meskipun di elemen form -->
<input value.bind="staticValue & oneTime">

<!-- Paksa two-way di custom element -->
<custom-input value.bind="data & twoWay">

<!-- Paksa to-view -->
<span textcontent.bind="computed & toView">
```

**`& attr` — bind ke attribute, bukan property:**

```html
<!-- Berguna untuk ARIA, data-* attributes -->
<div aria-busy.bind="isLoading & attr"></div>
<div data-project-id.bind="project.id & attr"></div>
```

Tanpa `& attr`, Aurelia bind ke DOM property (seperti `ariaBusy`). Dengan `& attr`, Aurelia set HTML attribute asli (`aria-busy`), yang lebih kompatibel dengan screen reader dan library pihak ketiga.

---

## 14. `containerless`: Komponen Tanpa Wrapper

Secara default, custom element Aurelia merender tag host-nya di DOM:

```html
<!-- Sebelum render: -->
<user-card name="Alice"></user-card>

<!-- Setelah render: -->
<user-card name="Alice">
  <div class="card">...</div>
</user-card>
```

Tag `<user-card>` tetap ada sebagai wrapper. Kadang ini yang kamu mau (untuk styling), kadang tidak (misalnya di dalam `<table>` atau `<select>` yang tidak boleh punya wrapper asing).

`containerless` menghilangkan tag host:

```typescript
// src/components/ui-separator.ts
import { customElement } from 'aurelia';

@customElement({
  name: 'ui-separator',
  template: '<div class="h-px bg-border"></div>',
  containerless: true, // ← tidak ada tag <ui-separator> di DOM
})
export class UISeparator {}
```

```html
<!-- Sebelum render: -->
<ui-separator></ui-separator>

<!-- Setelah render (TANPA wrapper): -->
<div class="h-px bg-border"></div>
```

**Kapan pakai `containerless`?**
- Komponen UI primitif (separator, badge, avatar) yang tidak butuh wrapper
- Komponen yang harus langsung menjadi child dari elemen HTML tertentu (`<td>`, `<option>`, `<li>`)
- Komponen yang hanya "mengembalikan" tag HTML asli

**Kapan JANGAN pakai:**
- Kalau parent perlu memberikan class ke host (`<user-card class="mb-4">`)
- Kalau kamu butuh `ref` ke custom element host
- Kalau komponen butuh Shadow DOM

---

## 15. `<let>`: Variabel Lokal di Template

Deklarasikan variabel langsung di template tanpa menambah properti di class:

```html
<!-- Hitung sekali, pakai berkali -->
<let total.bind="price * quantity"></let>
<p>Total: ${total}</p>
<p>Dengan PPN: ${total * 1.11}</p>
```

```html
<!-- String interpolation -->
<let full-name.bind="`${firstName} ${lastName}`"></let>
<span>${fullName}</span>
```

Catatan: nama variabel di `<let>` pakai kebab-case di HTML, tapi camelCase saat dipakai (`full-name` → `fullName`).

---

## Latihan Bab 05

### Latihan 1: Daftar Produk dengan Filter

Buat halaman yang menampilkan daftar produk dengan:
- Search input (debounce 300ms)
- Filter status (select dengan `change.trigger`)
- Tabel dengan `repeat.for` dan `key: id`
- Total jumlah produk ditampilkan

<details>
<summary>Solusi</summary>

```typescript
// src/pages/product-list.ts
export class ProductList {
  products: Product[] = [];
  filtered: Product[] = [];
  searchQuery = '';
  statusFilter = 'all';

  get total() {
    return this.filtered.length;
  }

  private searchTimer?: ReturnType<typeof setTimeout>;

  onSearchInput() {
    clearTimeout(this.searchTimer);
    this.searchTimer = setTimeout(() => this.applyFilters(), 300);
  }

  onFilterChange() {
    this.applyFilters();
  }

  private applyFilters() {
    this.filtered = this.products.filter(p => {
      const matchSearch = !this.searchQuery ||
        p.name.toLowerCase().includes(this.searchQuery.toLowerCase());
      const matchStatus = this.statusFilter === 'all' || p.status === this.statusFilter;
      return matchSearch && matchStatus;
    });
  }

  async binding() {
    // Simulasi load data
    this.products = [
      { id: '1', name: 'Produk A', status: 'active', price: 50000 },
      { id: '2', name: 'Produk B', status: 'inactive', price: 75000 },
    ];
    this.applyFilters();
  }

  detaching() {
    clearTimeout(this.searchTimer);
  }
}
```

```html
<!-- src/pages/product-list.html -->
<div>
  <input type="search" value.two-way="searchQuery" input.trigger="onSearchInput()">
  <select value.two-way="statusFilter" change.trigger="onFilterChange()">
    <option value="all">Semua</option>
    <option value="active">Aktif</option>
    <option value="inactive">Nonaktif</option>
  </select>

  <p>Total: ${total} produk</p>

  <table>
    <tbody>
      <tr repeat.for="product of filtered; key: id">
        <td>${product.name}</td>
        <td>${product.status}</td>
        <td>${product.price | currency}</td>
      </tr>
    </tbody>
  </table>
</div>
```

</details>

### Latihan 2: Value Converter untuk Status Badge

Buat value converter yang menerima status string (`active`, `inactive`, `pending`) dan return class CSS yang sesuai.

<details>
<summary>Solusi</summary>

```typescript
// src/resources/status-badge.ts
import { valueConverter } from 'aurelia';

@valueConverter('statusBadge')
export class StatusBadgeValueConverter {
  toView(status: string): string {
    const classes: Record<string, string> = {
      active: 'bg-green-100 text-green-800',
      inactive: 'bg-gray-100 text-gray-800',
      pending: 'bg-yellow-100 text-yellow-800',
    };
    return classes[status] ?? 'bg-gray-100 text-gray-800';
  }
}
```

```html
<import from="../resources/status-badge"></import>
<span class="${product.status | statusBadge} px-2 py-1 rounded">${product.status}</span>
```

</details>

---

## Ringkasan

| Binding | Sintaks | Kapan |
|---------|---------|-------|
| Interpolasi | `${nilai}` | Tampilkan teks, ekspresi sederhana |
| Auto mode | `prop.bind` | Default — Aurelia pilih mode yang tepat |
| Satu arah ke template | `prop.to-view` | Data hanya ditampilkan |
| Dua arah | `prop.two-way` | Form input, data bolak-balik |
| Dari template | `prop.from-view` | Jarang — capture output komponen |
| Event | `event.trigger` | Handle interaksi user |
| Event + prevent | `event.trigger:prevent` | Form submit — wajib pakai ini |
| Kondisional | `if.bind` / `show.bind` | Tampil/sembunyi |
| Loop | `repeat.for` | Render list |
| Referensi DOM | `ref` | Akses elemen langsung |
| Modifikasi binding | `& debounce`, `& throttle` | Optimasi performa |
| Value converter | `${value | converter}` | Transformasi tampilan data |
| Binding behavior | `value.bind="x & behavior"` | Modifikasi runtime binding |
| `containerless` | `@customElement({ containerless })` | Hilangkan wrapper tag |
| `<let>` | `<let x.bind="expr">` | Variabel lokal template |

---

## Selanjutnya

Kamu sekarang tahu cara menghubungkan data ke template dengan lengkap. Langkah berikutnya: **memecah UI menjadi komponen-komponen yang bisa dipakai ulang** dan cara mereka berkomunikasi satu sama lain.

→ [Bab 06: Komponen & Komunikasi](../bab-06/README.md)
→ [Bab 05b: Forms & Validation](../bab-05b/README.md)
