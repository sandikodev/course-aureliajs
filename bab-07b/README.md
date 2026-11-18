# Bab 07b — Testing dengan Vitest + createFixture

> Kalau kamu mau dapat job Aurelia di Upwork, kemampuan menulis test adalah pembeda utama. Job "rescue" hampir selalu minta test untuk mencegah regresi. Bab ini membahas cara test komponen Aurelia dengan `createFixture` dari `@aurelia/testing`.

---

## Setup Vitest

Install dulu:

```bash
pnpm add -D vitest @aurelia/testing jsdom
```

Buat `vitest.config.ts` yang merge dengan Vite config yang sudah ada:

```typescript
// vitest.config.ts
import { fileURLToPath } from 'node:url';
import { mergeConfig, defineConfig, configDefaults } from 'vitest/config';
import viteConfig from './vite.config';

export default mergeConfig(
  viteConfig,
  defineConfig({
    test: {
      environment: 'jsdom',
      watch: false,
      exclude: [...configDefaults.exclude, 'e2e/*'],
      root: fileURLToPath(new URL('./', import.meta.url)),
    },
  }),
);
```

---

## `createFixture` — Dasar

`createFixture` membuat instance komponen lengkap dengan DOM, binding, dan lifecycle aktif.

```typescript
import { createFixture } from '@aurelia/testing';

// Test paling sederhana
const { component, assertText, started, stop } = createFixture(
  '<div>${message}</div>',
  class { message = 'Halo Dunia'; },
);

await started; // tunggu lifecycle selesai
assertText('Halo Dunia'); // assert teks di DOM
await stop(true); // cleanup + hapus dari DOM
```

**Aturan penting:**
- Selalu `await started` sebelum assert — async lifecycle hook belum selesai sebelum ini
- Selalu `await stop(true)` di akhir — mencegah leak antar test
- Assert via helper bawaan, bukan querySelector manual

---

## Assertion Helpers

```typescript
const { assertText, assertHtml, assertValue, assertClass, assertAttr, trigger } = createFixture(
  '<input value.bind="name" class.bind="isActive ? \"active\" : \"\">',
  class { name = 'Alice'; isActive = true; },
);

await started;

assertValue('input', 'Alice');           // cek value elemen
assertClass('input', 'active');          // cek class ada
assertAttr('input', 'value', 'Alice');   // cek attribute
```

---

## Trigger Events

```typescript
const { trigger, assertText, started } = createFixture(
  '<button click.trigger="count++">${count}</button>',
  class { count = 0; },
);

await started;
assertText('0');

trigger.click('button'); // klik tombol
await tasksSettled();     // tunggu Aurelia update DOM
assertText('1');
```

**`tasksSettled`** dari `@aurelia/runtime` — wajib dipanggil setelah mutasi state sebelum assert. Tanpa ini, DOM mungkin belum terupdate.

```typescript
import { tasksSettled } from '@aurelia/runtime';
```

---

## Test dengan Input

```typescript
import { createFixture } from '@aurelia/testing';
import { tasksSettled } from '@aurelia/runtime';

test('input mengubah properti', async () => {
  const { component, type, assertValue, started } = createFixture(
    '<input type="text" value.two-way="name">',
    class { name = ''; },
  );

  await started;

  await type('input', 'Budi');
  await tasksSettled();

  expect(component.name).toBe('Budi');
  assertValue('input', 'Budi');
});
```

---

## Test dengan Mock Service

Register mock service via argumen ketiga `createFixture`:

```typescript
import { createFixture, Registration } from '@aurelia/testing';
import { tasksSettled } from '@aurelia/runtime';

// Mock AuthService
const mockAuth = {
  isAuthenticated: true,
  user: { id: '1', name: 'Test User', email: 'test@example.com' },
  refresh: vi.fn().mockResolvedValue(true),
};

test('halaman menampilkan nama user dari AuthService', async () => {
  const { assertText, started } = createFixture(
    '<div if.bind="auth.isAuthenticated">Halo, ${auth.user.name}</div>',
    class {},
    [Registration.instance(AuthService, mockAuth)],
  );

  await started;
  assertText('Halo, Test User');
  expect(mockAuth.refresh).not.toHaveBeenCalled();
});
```

---

## Test Komponen dengan `@bindable`

```typescript
import { createFixture } from '@aurelia/testing';
import { tasksSettled } from '@aurelia/runtime';

// Komponen yang dites
import { UserCard } from '../src/components/user-card';

test('UserCard menampilkan nama dan email', async () => {
  const { assertText, started } = createFixture(
    '<user-card name.bind="nama" email.bind="email"></user-card>',
    class { nama = 'Alice'; email = 'alice@example.com'; },
    [UserCard],
  );

  await started;
  assertText('Alice');
  assertText('alice@example.com');
});

test('UserCard bereaksi saat bindable berubah', async () => {
  const { component, assertText, started } = createFixture(
    '<user-card name.bind="nama"></user-card>',
    class { nama = 'Alice'; },
    [UserCard],
  );

  await started;
  assertText('Alice');

  component.nama = 'Bob';
  await tasksSettled();
  assertText('Bob');
});
```

---

## Test Routing

Gunakan `historyStrategy: 'none'` agar test tidak mengubah browser history:

```typescript
import { createFixture } from '@aurelia/testing';
import { route, IRouter, RouterConfiguration } from '@aurelia/router';

@route({
  routes: [
    { path: '', component: () => import('./home') },
    { path: 'users/:id', component: () => import('./user-detail') },
  ],
})
class App {}

test('navigasi ke /users/42 render UserDetail', async () => {
  const { appHost, container, started, stop } = createFixture(
    '<au-viewport></au-viewport>',
    App,
    [RouterConfiguration.customize({ historyStrategy: 'none' })],
  );

  await started;

  const router = container.get(IRouter);
  await router.load('users/42');

  expect(appHost.textContent).toContain('User 42');
  await stop(true);
});
```

---

## Pola Test yang Direkomendasikan

### 1. Test Service Secara Isolasi

Service adalah class biasa — test tanpa DOM:

```typescript
import { describe, test, expect } from 'vitest';
import { CartService } from '../src/services/cart';

describe('CartService', () => {
  test('total menghitung jumlah harga * qty', () => {
    const cart = new CartService();
    cart.addItem({ id: '1', name: 'A', price: 10000, qty: 2 });
    cart.addItem({ id: '2', name: 'B', price: 25000, qty: 1 });
    expect(cart.total).toBe(45000);
  });

  test('removeItem menghapus item yang benar', () => {
    const cart = new CartService();
    cart.addItem({ id: '1', name: 'A', price: 10000, qty: 1 });
    cart.addItem({ id: '2', name: 'B', price: 25000, qty: 1 });
    cart.removeItem('1');
    expect(cart.items).toHaveLength(1);
    expect(cart.items[0].id).toBe('2');
  });
});
```

### 2. Test Value Converter Secara Isolasi

```typescript
import { CurrencyValueConverter } from '../src/resources/currency';

test('CurrencyValueConverter format cents ke rupiah', () => {
  const vc = new CurrencyValueConverter();
  expect(vc.toView(5000000)).toContain('50.000.000');
});
```

### 3. Test Komponen dengan createFixture

```typescript
test('counter bertambah saat tombol diklik', async () => {
  const { trigger, assertText, started } = createFixture(
    '<button click.trigger="increment()">${count}</button>',
    class { count = 0; increment() { this.count++; } },
  );

  await started;
  trigger.click('button');
  await tasksSettled();
  assertText('1');
});
```

---

## Pitfalls yang Umum

| Masalah | Penyebab | Solusi |
|---------|----------|--------|
| Assert gagal padahal kode benar | Tidak `await started` | Selalu await sebelum assert |
| Test kedua fail karena DOM leak | Tidak `stop(true)` | Selalu stop + dispose |
| DOM tidak update setelah mutasi | Tidak `await tasksSettled()` | Panggil setelah ubah state |
| `ref` undefined | Diakses sebelum `bound()` | Assert di `started` atau setelahnya |
| Mock tidak terpakai | Salah registration key | Pakai `Registration.instance(IAuthService, mock)` |

---

## Latihan

### Latihan: Test Komponen Toggle

Buat test untuk komponen yang punya tombol toggle yang mengubah visibilitas konten.

<details>
<summary>Solusi</summary>

```typescript
import { createFixture } from '@aurelia/testing';
import { tasksSettled } from '@aurelia/runtime';

// Komponen
export class TogglePanel {
  isOpen = false;
  toggle() { this.isOpen = !this.isOpen; }
}

// Template: '<button click.trigger="toggle()">Toggle</button><div if.bind="isOpen">Konten</div>'

test('panel tersembunyi saat awal', async () => {
  const { appHost, assertText, started } = createFixture(
    '<button click.trigger="toggle()">Toggle</button><div if.bind="isOpen">Konten</div>',
    new TogglePanel(),
    [TogglePanel],
  );

  await started;
  expect(appHost.textContent).not.toContain('Konten');
});

test('panel muncul setelah toggle', async () => {
  const { trigger, appHost, started } = createFixture(
    '<button click.trigger="toggle()">Toggle</button><div if.bind="isOpen">Konten</div>',
    new TogglePanel(),
    [TogglePanel],
  );

  await started;
  trigger.click('button');
  await tasksSettled();
  expect(appHost.textContent).toContain('Konten');
});

test('panel tersembunyi lagi setelah toggle kedua', async () => {
  const { trigger, appHost, started } = createFixture(
    '<button click.trigger="toggle()">Toggle</button><div if.bind="isOpen">Konten</div>',
    new TogglePanel(),
    [TogglePanel],
  );

  await started;
  trigger.click('button'); // buka
  await tasksSettled();
  trigger.click('button'); // tutup
  await tasksSettled();
  expect(appHost.textContent).not.toContain('Konten');
});
```

</details>

---

## Ringkasan

- `createFixture` membuat komponen lengkap dengan DOM dan lifecycle aktif
- Selalu `await started` sebelum assert, `await stop(true)` di akhir
- `tasksSettled()` dari `@aurelia/runtime` — wajib setelah mutasi state sebelum assert DOM
- Mock service via `Registration.instance(Interface, mock)` di argumen ketiga
- Test service dan value converter secara isolasi (tanpa createFixture)
- Test routing dengan `historyStrategy: 'none'`
- Pitfalls umum: lupa `await started`, lupa `stop(true)`, lupa `tasksSettled()`

---

## Selanjutnya

Testing sudah lengkap. Sekarang kembali ke routing untuk perdalam dengan nested routing dan 404 handling.

→ [Bab 07: Dependency Injection](../bab-07/README.md) ← kembali ke induk
→ [Bab 08: Routing](../bab-08/README.md)
