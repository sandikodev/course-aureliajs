# Bab 05b — Forms & Validation

> Setiap aplikasi nyata punya form. Dan setiap form butuh validasi. Bab ini membahas cara Aurelia 2 menangani input form — dari binding dasar sampai validasi dengan `@aurelia/validation`.

---

## Binding untuk Form Controls

### Text Input

```html
<!-- .bind otomatis two-way pada elemen form -->
<input type="text" value.bind="name">
<textarea value.bind="bio"></textarea>
```

**Gotcha angka:** `value.bind` pada `<input type="number">` tetap menghasilkan **string**. Gunakan `value-as-number.bind`:

```html
<!-- ❌ age adalah string: "42" -->
<input type="number" value.bind="age">

<!-- ✅ age adalah number: 42 -->
<input type="number" value-as-number.bind="age">
```

### Checkbox

```html
<!-- Boolean tunggal -->
<input type="checkbox" checked.bind="agreeToTerms">

<!-- Array — pakai model.bind untuk nilai per item -->
<label repeat.for="p of products">
  <input type="checkbox" model.bind="p.id" checked.bind="selectedIds"> ${p.name}
</label>
```

### Radio

```html
<label repeat.for="role of roles">
  <input type="radio" name="role" model.bind="role" checked.bind="selectedRole">
  ${role}
</label>
```

### Select

```html
<!-- String values -->
<select value.bind="selectedStatus">
  <option value="">Semua</option>
  <option value="active">Aktif</option>
  <option value="inactive">Nonaktif</option>
</select>

<!-- Object values — WAJIB pakai model.bind, bukan value -->
<select value.bind="selectedProduct" matcher.bind="byId">
  <option model.bind="null">Pilih…</option>
  <option repeat.for="p of products" model.bind="p">${p.name}</option>
</select>
```

```typescript
// matcher untuk object equality (bukan reference equality)
byId = (a: Product, b: Product) => a?.id === b?.id;
```

**Kenapa `model.bind` bukan `value`?** Karena atribut `value` di HTML selalu coerce ke string. Kalau kamu bind object ke `value`, yang tersimpan adalah `"[object Object]"`. `model.bind` menyimpan object asli.

### File Input

```html
<input type="file" multiple change.trigger="onFiles($event)">
```

```typescript
onFiles(e: Event) {
  const input = e.target as HTMLInputElement;
  this.files = input.files ? Array.from(input.files) : [];
}
```

---

## Submit Form dengan Benar

**Aturan paling penting:** `submit.trigger` TIDAK mencegah default browser. Tanpa `:prevent`, halaman akan reload.

```html
<!-- ❌ Browser reload halaman -->
<form submit.trigger="save()">

<!-- ✅ :prevent mencegah reload -->
<form submit.trigger:prevent="save()">
  <input value.bind="form.name">
  <button type="submit" disabled.bind="isSaving">
    ${isSaving ? 'Menyimpan…' : 'Simpan'}
  </button>
</form>
```

---

## Validasi dengan `@aurelia/validation`

Install dulu:

```bash
pnpm add @aurelia/validation @aurelia/validation-html
```

Daftarkan di `main.ts`:

```typescript
import { ValidationHtmlConfiguration } from '@aurelia/validation-html';

void Aurelia
  .register(ValidationHtmlConfiguration)
  .app(MyApp)
  .start();
```

### Definisi Rules

```typescript
import { resolve } from 'aurelia';
import { IValidationRules } from '@aurelia/validation';

export class SignupForm {
  private rules = resolve(IValidationRules);

  username = '';
  email = '';
  age: number | null = null;

  constructor() {
    this.rules
      .on(this)
      .ensure('username')
        .required()
        .minLength(3).withMessage('Minimal 3 karakter')
      .ensure('email')
        .required()
        .matches(/^[^\s@]+@[^\s@]+\.[^\s@]+$/).withMessage('Email tidak valid')
      .ensure('age')
        .satisfies(v => v == null || v >= 18).withMessage('Harus 18 tahun ke atas');
  }
}
```

### Controller untuk Form

Buat validation controller scoped ke form ini:

```typescript
import { resolve, newInstanceForScope } from 'aurelia';
import { IValidationController } from '@aurelia/validation-html';

export class SignupForm {
  private controller = resolve(newInstanceForScope(IValidationController));
  private rules = resolve(IValidationRules);

  username = '';
  email = '';
  errors: string[] = [];

  constructor() {
    this.rules.on(this)
      .ensure('username').required().minLength(3)
      .ensure('email').required();
  }

  async submit() {
    const result = await this.controller.validate();
    if (result.valid) {
      // Kirim ke API
      await this.api.signup({ username: this.username, email: this.email });
    } else {
      this.errors = result.results
        .filter(r => !r.valid)
        .map(r => r.message);
    }
  }
}
```

### Template dengan `& validate`

```html
<form submit.trigger:prevent="submit()">
  <div>
    <label>Username</label>
    <input value.bind="username & validate">
    <!-- & validate akan trigger validasi sesuai default (focusout) -->
  </div>

  <div>
    <label>Email</label>
    <input value.bind="email & validate:'blur'">
  </div>

  <!-- Tampilkan error -->
  <ul if.bind="errors.length">
    <li repeat.for="err of errors" class="text-red-500">${err}</li>
  </ul>

  <button type="submit">Daftar</button>
</form>
```

**Trigger options untuk `& validate`:**
- `& validate` — default (focusout)
- `& validate:'blur'` — saat blur
- `& validate:'change'` — saat nilai berubah
- `& validate:'changeOrBlur'` — change atau blur
- `& validate:'manual'` — hanya saat `controller.validate()` dipanggil

### Error Per-Field

```html
<div validation-errors.from-view="usernameErrors">
  <input value.bind="username & validate">
  <span class="error" repeat.for="err of usernameErrors">${err.result.message}</span>
</div>
```

---

## Contoh Nyata: Form Create Project dari Aurelia Ops

```typescript
// Disederhanakan dari apps/web/src/pages/projects-page.ts
export class ProjectsPage {
  form = {
    name: '',
    ownerName: '',
    status: 'planning',
    budget: '',
    dueDate: '',
  };
  formError = '';
  isCreating = false;

  async saveProject(event?: Event): Promise<void> {
    event?.preventDefault();
    this.formError = '';
    this.isCreating = true;

    try {
      await api.createProject({
        name: this.form.name.trim(),
        ownerName: this.form.ownerName.trim(),
        status: this.form.status,
        budgetCents: Math.round(parseFloat(this.form.budget || '0') * 100),
        dueDate: this.form.dueDate || null,
      });
      this.dialogEl.close();
      this.toast.success('Proyek dibuat!');
      await this.loadProjects();
    } catch (error) {
      if (error instanceof ApiError) {
        if (error.code === 'plan_limit_reached') {
          this.formError = 'Limit plan tercapai. Upgrade dulu.';
        } else if (error.issues.length > 0) {
          this.formError = error.issues.map(i => i.message).join(', ');
        } else {
          this.formError = error.message;
        }
      } else {
        this.formError = 'Gagal menyimpan.';
      }
    } finally {
      this.isCreating = false;
    }
  }
}
```

```html
<form submit.trigger:prevent="saveProject()">
  <label>Nama Proyek</label>
  <input value.bind="form.name" placeholder="Mis: Redesign Website">

  <label>PIC</label>
  <input value.bind="form.ownerName" placeholder="Nama pemilik">

  <label>Status</label>
  <select value.bind="form.status">
    <option value="planning">Planning</option>
    <option value="active">Active</option>
    <option value="paused">Paused</option>
    <option value="done">Done</option>
  </select>

  <label>Budget (Rp)</label>
  <input type="number" value.bind="form.budget" placeholder="50000000">

  <label>Jatuh Tempo</label>
  <input type="date" value.bind="form.dueDate">

  <p if.bind="formError" class="text-red-500">${formError}</p>

  <button type="submit" disabled.bind="isCreating">
    ${isCreating ? 'Menyimpan…' : 'Simpan'}
  </button>
</form>
```

---

## Latihan

### Latihan 1: Form dengan Validasi Sederhana

Buat form registrasi dengan field name, email, password. Validasi:
- Name: required, min 2 karakter
- Email: required, format email
- Password: required, min 8 karakter

Tampilkan error per field.

<details>
<summary>Solusi</summary>

```typescript
import { resolve, newInstanceForScope } from 'aurelia';
import { IValidationRules } from '@aurelia/validation';
import { IValidationController } from '@aurelia/validation-html';

export class RegisterForm {
  private controller = resolve(newInstanceForScope(IValidationController));
  private rules = resolve(IValidationRules);

  name = '';
  email = '';
  password = '';
  submitted = false;

  constructor() {
    this.rules.on(this)
      .ensure('name').required().minLength(2)
      .ensure('email').required()
        .matches(/^[^\s@]+@[^\s@]+\.[^\s@]+$/).withMessage('Format email tidak valid')
      .ensure('password').required().minLength(8);
  }

  async submit() {
    const result = await this.controller.validate();
    if (result.valid) {
      this.submitted = true;
    }
  }
}
```

```html
<form submit.trigger:prevent="submit()">
  <div validation-errors.from-view="nameErrors">
    <label>Nama</label>
    <input value.bind="name & validate">
    <span class="error" repeat.for="e of nameErrors">${e.result.message}</span>
  </div>

  <div validation-errors.from-view="emailErrors">
    <label>Email</label>
    <input value.bind="email & validate">
    <span class="error" repeat.for="e of emailErrors">${e.result.message}</span>
  </div>

  <div validation-errors.from-view="passwordErrors">
    <label>Password</label>
    <input type="password" value.bind="password & validate">
    <span class="error" repeat.for="e of passwordErrors">${e.result.message}</span>
  </div>

  <p if.bind="submitted" class="text-green-600">Registrasi berhasil!</p>

  <button type="submit">Daftar</button>
</form>
```

</details>

---

## Ringkasan

- `value.bind` pada `<input type="number">` menghasilkan string — gunakan `value-as-number.bind`
- Checkbox array wajib pakai `model.bind`, bukan `value`
- Select dengan object wajib pakai `model.bind` + `matcher.bind`
- `submit.trigger:prevent` wajib — tanpa `:prevent` halaman reload
- `@aurelia/validation` + `@aurelia/validation-html` untuk validasi yang terintegrasi
- `& validate` binding behavior mengaktifkan validasi di template
- `newInstanceForScope(IValidationController)` membuat controller per form

---

## Selanjutnya

Form dan validasi sudah lengkap. Sekarang kembali ke komponen — kita perdalam dengan custom attributes dan HTML-only components.

→ [Bab 06: Komponen & Komunikasi](../bab-06/README.md)
