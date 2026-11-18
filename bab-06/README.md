# Bab 06 — Komponen & Komunikasi

> Aplikasi nyata tidak terdiri dari satu komponen raksasa. Kamu perlu memecah UI menjadi bagian-bagian kecil yang bisa dipakai ulang, lalu menghubungkan mereka. Bab ini membahas cara membuat komponen yang dapat menerima data dari luar, mengeluarkan event ke luar, dan berkomunikasi lintas komponen tanpa coupling yang ketat.

---

## Anatomi Komponen Aurelia

Komponen Aurelia yang lengkap terdiri dari dua file:

```
src/components/
├── user-card.ts    ← Logic (class TypeScript)
└── user-card.html  ← Tampilan (template HTML)
```

Nama file menentukan nama tag HTML secara otomatis: `user-card.ts` menjadi `<user-card>`.

---

## `@bindable` — Menerima Data dari Luar

`@bindable` adalah cara komponen menerima data dari parent-nya. Setara dengan `@Input()` di Angular atau `props` di React.

```typescript
// src/components/user-card.ts
import { bindable } from 'aurelia';

export class UserCard {
  @bindable name = '';
  @bindable email = '';
  @bindable role: 'owner' | 'admin' | 'member' = 'member';

  get roleLabel(): string {
    const labels = { owner: 'Pemilik', admin: 'Admin', member: 'Anggota' };
    return labels[this.role];
  }
}
```

```html
<!-- src/components/user-card.html -->
<div class="user-card">
  <strong>${name}</strong>
  <span>${email}</span>
  <span class="badge badge-${role}">${roleLabel}</span>
</div>
```

Pemakaian di parent:

```html
<!-- Di template halaman manapun yang sudah import komponen ini -->
<user-card
  name.bind="currentUser.name"
  email.bind="currentUser.email"
  role.bind="currentUser.role">
</user-card>
```

**Penamaan: camelCase di TypeScript, kebab-case di HTML**

```typescript
@bindable firstName = ''; // TypeScript
```

```html
<user-card first-name.bind="user.firstName"></user-card>
```

### Callback saat Bindable Berubah

Kalau kamu perlu bereaksi ketika nilai bindable berubah dari luar:

```typescript
import { bindable } from 'aurelia';

export class SearchBox {
  @bindable query = '';

  // Konvensi: nama properti + "Changed"
  queryChanged(newValue: string, oldValue: string) {
    if (newValue.length >= 3) {
      void this.search(newValue);
    }
  }

  private async search(term: string) {
    // jalankan pencarian
  }
}
```

---

## Mendaftarkan Komponen

Komponen tidak tersedia secara global secara otomatis. Kamu harus mendaftarkannya. Ada tiga cara:

### Cara 1: `<import>` di Template (Paling Umum)

```html
<!-- Di template yang mau pakai komponen -->
<import from="../components/user-card"></import>

<user-card name.bind="user.name" email.bind="user.email"></user-card>
```

### Cara 2: `dependencies` di Class

```typescript
import { customElement } from 'aurelia';
import { UserCard } from '../components/user-card';
import template from './members-page.html';

@customElement({
  name: 'members-page',
  template,
  dependencies: [UserCard], // ← daftarkan di sini
})
export class MembersPage {
  // ...
}
```

### Cara 3: Global di `main.ts`

```typescript
// main.ts
import { UserCard } from './components/user-card';

void Aurelia
  .register(
    RouterConfiguration.customize({ useUrlFragmentHash: false }),
    UserCard, // ← tersedia di seluruh aplikasi
  )
  .app({ host: document.querySelector('app-root')!, component: App })
  .start();
```

**Kapan pakai yang mana?**
- Komponen yang dipakai di banyak halaman: global di `main.ts`
- Komponen yang hanya dipakai di satu tempat: `<import>` atau `dependencies`
- Design system / UI kit: selalu global

---

## Content Projection dengan `<au-slot>`

`<au-slot>` memungkinkan parent menyisipkan konten ke dalam komponen. Setara dengan `<slot>` di Web Components atau `{children}` di React.

```typescript
// src/components/card.ts
export class Card {
  // Tidak perlu apapun di sini untuk komponen wrapper murni
}
```

```html
<!-- src/components/card.html -->
<div class="card rounded-lg border p-4 shadow-sm">
  <!-- Konten default kalau parent tidak isi slot header -->
  <div class="card-header">
    <au-slot name="header">
      <h3>Judul Default</h3>
    </au-slot>
  </div>
  <div class="card-body">
    <!-- Slot default (tanpa nama) -->
    <au-slot></au-slot>
  </div>
</div>
```

Pemakaian:

```html
<card>
  <!-- Isi slot header -->
  <span au-slot="header">Daftar Pengguna</span>

  <!-- Isi slot default (body) -->
  <user-card name.bind="user.name"></user-card>
</card>
```

---

## Mengirim Event ke Parent: `@observable` + EventAggregator

Ada dua cara komponen anak berkomunikasi ke parent.

### Cara 1: Custom Event via DOM

Cara paling langsung — dispatch event DOM standar:

```typescript
// src/components/confirm-button.ts
import { INode, resolve } from 'aurelia';

export class ConfirmButton {
  private element = resolve(INode) as HTMLElement;

  confirm() {
    // Dispatch custom event ke parent
    this.element.dispatchEvent(
      new CustomEvent('confirmed', {
        detail: { timestamp: Date.now() },
        bubbles: true, // naik ke parent
      })
    );
  }
}
```

```html
<!-- confirm-button.html -->
<button click.trigger="confirm()">Konfirmasi</button>
```

Tangkap di parent:

```html
<!-- Di template parent -->
<confirm-button confirmed.trigger="handleConfirm($event)"></confirm-button>
```

```typescript
// Di class parent
handleConfirm(event: CustomEvent) {
  console.log('Dikonfirmasi pada:', event.detail.timestamp);
}
```

### Cara 2: EventAggregator (Pub/Sub Lintas Komponen)

Untuk komunikasi antara komponen yang tidak punya hubungan langsung parent-child, Aurelia menyediakan `IEventAggregator` — sistem pub/sub bawaan.

```typescript
// src/services/app-events.ts
// Definisikan event sebagai class untuk type safety
export class ProjectCreated {
  constructor(
    public readonly projectId: string,
    public readonly projectName: string,
  ) {}
}

export class UserRoleChanged {
  constructor(
    public readonly userId: string,
    public readonly newRole: string,
  ) {}
}
```

**Publisher** — komponen yang mengirim event:

```typescript
// src/pages/projects-page.ts
import { resolve } from 'aurelia';
import { IEventAggregator } from 'aurelia';
import { ProjectCreated } from '../services/app-events';

export class ProjectsPage {
  private ea = resolve(IEventAggregator);

  async createProject(name: string) {
    const result = await this.api.createProject({ name });

    // Kirim event ke seluruh aplikasi
    this.ea.publish(new ProjectCreated(result.id, result.name));
  }
}
```

**Subscriber** — komponen yang mendengarkan event:

```typescript
// src/components/activity-feed.ts
import { resolve } from 'aurelia';
import { IEventAggregator } from 'aurelia';
import { ProjectCreated } from '../services/app-events';

export class ActivityFeed {
  private ea = resolve(IEventAggregator);
  private subscription?: { dispose(): void };

  activities: string[] = [];

  // Mulai mendengarkan saat komponen terikat
  binding() {
    this.subscription = this.ea.subscribe(ProjectCreated, (event) => {
      this.activities = [
        `Proyek "${event.projectName}" dibuat`,
        ...this.activities,
      ];
    });
  }

  // WAJIB: dispose saat komponen dilepas untuk mencegah memory leak
  unbinding() {
    this.subscription?.dispose();
  }
}
```

**Aturan penting:** Selalu dispose subscription di `unbinding()`. Lupa dispose adalah sumber memory leak yang umum.

---

## Contoh Nyata: ToastService dari Aurelia Ops

Proyek referensi menggunakan pendekatan yang sedikit berbeda — service sebagai shared state yang diobservasi:

```typescript
// Disederhanakan dari apps/web/src/services/toast.ts
export type ToastTone = 'success' | 'error' | 'info' | 'warning';

export type Toast = {
  id: number;
  message: string;
  tone: ToastTone;
};

export class ToastService {
  // Array ini diobservasi oleh ui-toaster
  toasts: Toast[] = [];
  private nextId = 1;

  success(message: string): void {
    this.add(message, 'success');
  }

  error(message: string): void {
    this.add(message, 'error');
  }

  dismiss(id: number): void {
    this.toasts = this.toasts.filter(t => t.id !== id);
  }

  private add(message: string, tone: ToastTone): void {
    const id = this.nextId++;
    this.toasts = [...this.toasts, { id, message, tone }];
    // Auto-dismiss setelah 4 detik
    setTimeout(() => this.dismiss(id), 4000);
  }
}
```

Komponen `ui-toaster` cukup bind ke `toasts`:

```html
<!-- ui-toaster.html (disederhanakan) -->
<div class="toast-container">
  <div
    repeat.for="toast of toastService.toasts; key: id"
    class="toast toast-${toast.tone}"
    click.trigger="toastService.dismiss(toast.id)">
    ${toast.message}
  </div>
</div>
```

Dan dari halaman manapun:

```typescript
// Di halaman apapun
export class MembersPage {
  private toast = resolve(ToastService);

  async inviteMember() {
    try {
      await this.api.invite(this.email);
      this.toast.success('Undangan berhasil dikirim!');
    } catch {
      this.toast.error('Gagal mengirim undangan.');
    }
  }
}
```

Pola ini lebih sederhana dari EventAggregator untuk kasus notifikasi global — service-nya adalah sumber kebenaran tunggal, dan komponen apapun bisa baca dan tulis ke sana.

---

## Membangun UI Kit Sendiri

Di Aurelia Ops, semua komponen UI didaftarkan secara global di `main.ts` lewat array `UI`:

```typescript
// apps/web/src/ui/index.ts (pola)
import UiButton from './button.aux';
import UiBadge from './badge.aux';
import UiCard from './card.aux';
// ... komponen lainnya

export const UI = [UiButton, UiBadge, UiCard, /* ... */];
```

```typescript
// main.ts
import { UI } from './ui';

void Aurelia
  .register(...UI, /* services */)
  .app(...)
  .start();
```

Dengan pola ini, semua template di seluruh aplikasi bisa pakai `<ui-button>`, `<ui-badge>`, dan lainnya tanpa import manual.

---

## Praktik: Komponen Daftar dengan Aksi

Buat komponen `member-row` yang:
1. Menerima data member via `@bindable`
2. Menampilkan nama, email, dan role
3. Mengirim event saat tombol "Hapus" diklik

```typescript
// src/components/member-row.ts
import { bindable } from 'aurelia';
import { INode, resolve } from 'aurelia';

type Member = { id: string; name: string; email: string; role: string };

export class MemberRow {
  @bindable member!: Member;

  private element = resolve(INode) as HTMLElement;

  remove() {
    this.element.dispatchEvent(
      new CustomEvent('remove', {
        detail: { memberId: this.member.id },
        bubbles: true,
      })
    );
  }
}
```

```html
<!-- src/components/member-row.html -->
<tr>
  <td>${member.name}</td>
  <td>${member.email}</td>
  <td><span class="badge">${member.role}</span></td>
  <td>
    <button click.trigger="remove()">Hapus</button>
  </td>
</tr>
```

Pemakaian:

```html
<!-- Di halaman members -->
<import from="../components/member-row"></import>

<table>
  <tbody>
    <tr
      is="member-row"
      repeat.for="member of members; key: id"
      member.bind="member"
      remove.trigger="handleRemove($event)">
    </tr>
  </tbody>
</table>
```

---

## Custom Attributes: Perilaku yang Bisa Dilepas

Custom attribute adalah perilaku yang bisa ditempel ke elemen HTML manapun. Berbeda dari custom element (yang membuat tag baru), custom attribute memodifikasi elemen yang sudah ada.

```typescript
// src/attributes/tooltip.ts
import { customAttribute, INode, bindable, resolve } from 'aurelia';

@customAttribute({ name: 'tooltip', defaultProperty: 'text' })
export class TooltipCustomAttribute {
  @bindable text = '';

  private element = resolve(INode) as HTMLElement;

  attached() {
    this.element.title = this.text;
  }

  textChanged(newValue: string) {
    this.element.title = newValue;
  }
}
```

Pemakaian:

```html
<import from="../attributes/tooltip"></import>

<!-- defaultProperty = 'text' → cukup tulis string langsung -->
<button tooltip="Klik untuk simpan">Simpan</button>

<!-- Atau bind dinamis -->
<button tooltip.bind="dynamicTooltipText">Simpan</button>
```

**Contoh nyata — custom attribute untuk autofocus:**

```typescript
// src/attributes/autofocus.ts
import { customAttribute, INode, resolve } from 'aurelia';

@customAttribute('autofocus')
export class AutofocusCustomAttribute {
  private element = resolve(INode) as HTMLElement;

  attached() {
    this.element.focus();
  }
}
```

```html
<import from="../attributes/autofocus"></import>
<input autofocus placeholder="Ketik di sini...">
```

**Perbedaan custom element vs custom attribute:**

| Custom Element | Custom Attribute |
|----------------|------------------|
| Membuat tag baru (`<my-card>`) | Memodifikasi tag yang ada (`<button tooltip="...">`) |
| Punya template sendiri | Tidak punya template — bekerja pada elemen host |
| Menerima `@bindable` | Menerima `@bindable` + `defaultProperty` |

---

## HTML-Only Components: Komponen Tanpa TypeScript

Untuk komponen visual sederhana yang tidak butuh logic, kamu tidak perlu file `.ts` sama sekali. Cukup satu file HTML.

```html
<!-- src/components/status-badge.html -->
<bindable name="status"></bindable>
<bindable name="message"></bindable>
<span class="badge badge-${status}">${message}</span>
```

Pemakaian:

```html
<import from="../components/status-badge.html"></import>
<status-badge status="success" message="Selesai"></status-badge>
<status-badge status="error" message="Gagal"></status-badge>
```

Ini berguna untuk:
- Badge, tag, label visual
- Separator, divider
- Wrapper markup tanpa logic
- Icon SVG yang dipakai berkali-kali

---

## `containerless` untuk UI Primitives

Saat membangun UI kit, beberapa komponen harus merender langsung tanpa tag wrapper. Contoh dari Aurelia Ops — separator:

```typescript
// Pola dari apps/web/src/ui/separator.aux
import { customElement } from 'aurelia';

@customElement({
  name: 'ui-separator',
  template: '<div class="h-px w-full bg-border" role="separator"></div>',
  containerless: true,
})
export class UISeparator {}
```

Tanpa `containerless`, DOM akan jadi:
```html
<ui-separator><div class="h-px bg-border"></div></ui-separator>
```

Dengan `containerless`:
```html
<div class="h-px bg-border"></div>
```

**Aturan praktis:**
- UI primitives (separator, badge, avatar, icon) → `containerless: true`
- Komponen yang butuh class dari parent → JANGAN `containerless`
- Komponen yang parent butuh `ref` ke host → JANGAN `containerless`

---

## `<au-compose>`: Komponen Dinamis

Kalau kamu perlu merender komponen yang ditentukan di runtime:

```html
<au-compose
  component.bind="currentComponent"
  model.bind="currentModel">
</au-compose>
```

```typescript
export class DynamicPage {
  currentComponent = null;
  currentModel = null;

  showChart() {
    this.currentComponent = () => import('./chart-component');
    this.currentModel = { data: this.chartData };
  }

  showTable() {
    this.currentComponent = () => import('./table-component');
    this.currentModel = { rows: this.tableData };
  }
}
```

---

## Pola UI Kit yang Dipakai Aurelia Ops

Aurelia Ops mendaftarkan semua komponen UI secara global lewat array:

```typescript
// apps/web/src/ui/index.ts
import UiButton from './button.aux';
import UiBadge from './badge.aux';
import UiCard from './card.aux';
import UiInput from './input.aux';
import UiSelect from './select.aux';
// ... dan lainnya

export const UI = [UiButton, UiBadge, UiCard, UiInput, UiSelect, /* ... */];
```

```typescript
// main.ts
import { UI } from './ui';

void Aurelia
  .register(...UI, /* services */)
  .app(...)
  .start();
```

Dengan pola ini, template di seluruh aplikasi bisa pakai `<ui-button>`, `<ui-badge>`, dll. tanpa import manual per halaman.

---

## Latihan Bab 06

### Latihan 1: Custom Attribute untuk Loading State

Buat custom attribute `loading` yang menambahkan class `is-loading` ke elemen saat nilai true, dan menghapus saat false.

<details>
<summary>Solusi</summary>

```typescript
// src/attributes/loading.ts
import { customAttribute, INode, bindable, resolve } from 'aurelia';

@customAttribute('loading')
export class LoadingCustomAttribute {
  @bindable value = false;
  private element = resolve(INode) as HTMLElement;

  valueChanged(newValue: boolean) {
    this.element.classList.toggle('is-loading', newValue);
    this.element.setAttribute('aria-busy', String(newValue));
  }

  attached() {
    this.valueChanged(this.value);
  }
}
```

```html
<import from="../attributes/loading"></import>
<button loading.bind="isSubmitting" disabled.bind="isSubmitting">
  ${isSubmitting ? 'Menyimpan…' : 'Simpan'}
</button>
```

</details>

### Latihan 2: HTML-Only Icon Component

Buat HTML-only component yang menampilkan icon SVG berdasarkan nama.

<details>
<summary>Solusi</summary>

```html
<!-- src/components/ui-icon.html -->
<bindable name="name"></bindable>
<bindable name="size"></bindable>

<svg if.bind="name === 'check'" width.bind="size || 16" height.bind="size || 16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
  <polyline points="20 6 9 17 4 12"></polyline>
</svg>

<svg if.bind="name === 'x'" width.bind="size || 16" height.bind="size || 16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
  <line x1="18" y1="6" x2="6" y2="18"></line>
  <line x1="6" y1="6" x2="18" y2="18"></line>
</svg>
```

Pemakaian:
```html
<import from="../components/ui-icon.html"></import>
<ui-icon name="check" size="20"></ui-icon>
<ui-icon name="x"></ui-icon>
```

</details>

---

## Ringkasan

- `@bindable` membuat komponen menerima data dari parent — nama camelCase di TypeScript, kebab-case di HTML
- `fooChanged()` dipanggil otomatis saat bindable `foo` berubah dari luar
- Daftarkan komponen via `<import>`, `dependencies`, atau global di `main.ts`
- `<au-slot>` untuk content projection
- Custom Event DOM untuk komunikasi parent-child
- `IEventAggregator` untuk komunikasi lintas komponen — selalu dispose di `unbinding()`
- Shared service (seperti `ToastService`) untuk state global
- Custom attribute memodifikasi elemen yang ada — bukan membuat tag baru
- HTML-only component cukup satu file `.html` dengan `<bindable>` — tidak perlu TypeScript
- `containerless: true` untuk UI primitives yang tidak boleh punya wrapper
- Daftarkan UI kit secara global di `main.ts` lewat array

---

## Selanjutnya

Komponen sudah bisa bicara satu sama lain. Tapi kode akan mulai berantakan kalau setiap komponen harus buat sendiri koneksi ke API, service, atau state. Di bab berikutnya kita bahas **Dependency Injection** — cara Aurelia mengelola ketergantungan antar bagian aplikasi secara bersih.

→ [Bab 07: Dependency Injection](../bab-07/README.md)
→ [Bab 05b: Forms & Validation](../bab-05b/README.md)
