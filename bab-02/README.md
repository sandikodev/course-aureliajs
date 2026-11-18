# Bab 02 — Mental Model Baru: Komponen = Class Biasa

> Kalau kamu datang dari React, pergeseran terbesar yang perlu kamu lakukan adalah ini: **berhenti berpikir bahwa komponen harus berupa fungsi yang dijalankan ulang setiap kali state berubah.** Di Aurelia, komponen adalah class yang hidup sekali — dan sistem observasi-nya yang bekerja di belakang layar.
>
> **Catatan tentang perbandingan:** Bab ini memperbandingkan Aurelia dengan React dan Vue untuk membantu memahami perbedaan mental model. Ini bukan perbandingan "mana yang lebih baik" — ini penjelasan "bagaimana cara kerjanya berbeda". Aurelia (Enterprise OOP), React (Functional Re-Render), dan Vue (Pragmatic Reactivity SFC) hidup di tiga klaster budaya yang berbeda. Analisis lengkap empat klaster ada di [`FRAMEWORK_POSITIONING.md`](../FRAMEWORK_POSITIONING.md).

---

## React, Vue, dan Aurelia: Tiga Cara Berbeda Menjawab Masalah yang Sama

Sebelum masuk ke kode, pahami dulu perbedaan fundamental:

**React (Klaster Functional Re-Render):** Komponen adalah fungsi yang dijalankan ulang setiap kali state berubah. Mental model: UI = f(state). React pakai Virtual DOM untuk mengetahui apa yang perlu diupdate setelah re-render.

**Aurelia (Klaster Enterprise Class OOP):** Komponen adalah class yang diinstansiasi sekali dan hidup. Properti diobservasi secara bedah. Tidak ada re-render, tidak ada Virtual DOM. Mental model: class yang hidup, observasi yang bekerja.

Kedua pendekatan ini valid untuk masalah yang berbeda. React cocok untuk tim yang berpikir dalam fungsi dan transformasi. Aurelia cocok untuk tim yang berpikir dalam class, service, dan DI — mirip cara kerja backend enterprise.

## Masalah dengan "Re-render Everything" (Sudut Pandang Aurelia)

Di React, cara kerjanya seperti ini:

```tsx
// React — fungsi ini dieksekusi ULANG setiap kali state berubah
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(count + 1)}>
      Diklik {count} kali
    </button>
  );
}
```

Setiap kali tombol diklik:
1. `setCount` dipanggil
2. React menjalankan ulang seluruh fungsi `Counter()` dari baris pertama
3. React membuat representasi Virtual DOM baru
4. React membandingkan dengan yang lama (diffing)
5. React memperbarui DOM nyata hanya pada bagian yang berbeda

Untuk komponen sederhana ini, overhead-nya tidak kelihatan. Tapi di komponen yang besar, dengan banyak props dan children, proses ini terus berulang — bahkan untuk perubahan yang sangat kecil.

React punya solusi untuk ini: `React.memo`, `useMemo`, `useCallback`. Tapi itu berarti kamu harus secara manual memberitahu React *apa* yang perlu di-cache dan *kapan* harus diperbarui. Itu cognitive overhead yang nyata.

---

## Cara Kerja Aurelia: Komponen Hanya Dibuat Sekali

Di Aurelia, pendekatan dasarnya berbeda:

```typescript
// counter.ts — class ini hanya diinstansiasi SEKALI
export class Counter {
  count = 0;

  increment() {
    this.count++; // ← Aurelia langsung update teks di DOM
  }
}
```

```html
<!-- counter.html -->
<button click.trigger="increment()">
  Diklik ${count} kali
</button>
```

Ketika tombol diklik:
1. `increment()` dipanggil
2. `this.count` berubah dari 0 ke 1
3. Aurelia mendeteksi perubahan pada `count` melalui ObserverLocator
4. **Hanya node teks yang menampilkan `count`** yang diperbarui
5. Selesai — tidak ada re-render, tidak ada diffing

Class `Counter` tidak pernah dijalankan ulang dari awal. Ia tetap hidup, dan properti-propertinya diobservasi secara bedah.

---

## Mengapa Ini Bukan Keterbatasan

Ketika developer React pertama kali melihat pendekatan ini, reaksi umum adalah: "Tapi gimana cara Aurelia tahu kalau ada yang berubah?"

Pertanyaan yang bagus. Jawabannya ada dua mekanisme:

### Mekanisme 1: Getter/Setter Observasi

Untuk properti biasa, Aurelia menggunakan teknik getter/setter di belakang layar. Ketika kamu tulis `this.count++`, sebenarnya setter yang sudah dipasang Aurelia yang dipanggil — dan setter itu tahu siapa yang perlu dinotifikasi.

Ini transparan. Kamu tidak perlu melakukan apapun khusus. Cukup tulis properti biasa.

### Mekanisme 2: `@observable` untuk Callback Eksplisit

Kalau kamu butuh bereaksi saat properti berubah:

```typescript
import { observable } from 'aurelia';

export class UserProfile {
  @observable name = '';

  // Dipanggil otomatis setiap kali name berubah
  nameChanged(newValue: string, oldValue: string) {
    console.log(`Nama berubah dari "${oldValue}" ke "${newValue}"`);
  }
}
```

Pola ini mirip dengan `watch` di Vue atau `ngOnChanges` di Angular — tapi lebih sederhana karena konvensinya jelas: nama method = nama properti + "Changed".

---

## Lifecycle: Komponen Punya Tahapan Hidup

Karena komponen hanya dibuat sekali, ia punya siklus hidup yang jelas:

```
Urutan lifecycle hooks (dari atas ke bawah):

  constructor()     ← injeksi service, setup awal
       │
  binding()         ← bindable sudah di-assign, bisa load data (async OK)
       │
  bound()           ← template aktif, ref tersedia
       │
  attaching()       ← DOM insertion sedang terjadi
       │
  attached()        ← komponen & semua child sudah di DOM ← setup 3rd-party lib
       │
  ···· user berinteraksi ····
       │
  detaching()       ← sebelum keluar DOM ← cleanup DOM listeners
       │
  unbinding()       ← bindings dilepas ← dispose subscriptions
       │
  dispose()         ← teardown permanen (view di-cache tidak sampai sini)
```

```typescript
export class MyComponent {
  // 1. Properti bindable sudah di-assign, tapi template belum aktif
  binding() {
    console.log('binding — saat untuk load data awal');
  }

  // 2. Template sudah aktif, ref tersedia
  bound() {
    console.log('bound — akses ref di sini');
  }

  // 3. Komponen sudah masuk DOM
  attached() {
    console.log('attached — saat untuk setup library pihak ketiga');
  }

  // 4. Komponen akan keluar dari DOM
  detaching() {
    console.log('detaching — cleanup');
  }

  // 5. Template dilepas
  unbinding() {
    console.log('unbinding — dispose subscriptions');
  }
}
```

Kamu tidak harus implement semua. Implement hanya yang dibutuhkan.

**Kapan pakai apa:**
- `binding()` — load data yang dibutuhkan sebelum render (bisa return Promise)
- `attached()` — init library yang butuh akses DOM nyata (chart, map, dll)
- `unbinding()` — dispose event listener, interval, subscription

---

## Perbandingan Mental Model

Tabel ini membandingkan Aurelia dengan React karena React adalah framework terpopuler saat ini dan banyak developer datang dari sana. Tapi ingat: **ini perbandingan lintas-klaster**. Aurelia (Klaster 4 — Enterprise Class OOP) dan React (Klaster 1 — Functional Re-Render) hidup di budaya yang berbeda. Ini bukan soal mana lebih baik, tapi soal perbedaan mental model yang fundamental.

Vue perlu disebut tersendiri karena sering disalahposisikan. Vue lebih dekat ke Svelte daripada ke React — keduanya SFC, keduanya reaktivitas via mutasi langsung, keduanya komponen sebagai objek stateful. Vue punya "rasa OOP" via Options API yang sekilas mirip Angular/Aurelia, tapi mekanisme reaktivitasnya (ES6 Proxy) berbeda dari keduanya.

| Konsep | React (Klaster 1) | Vue (Klaster 3) | Aurelia (Klaster 4) |
|--------|-------------------|-----------------|---------------------|
| Komponen adalah | Fungsi yang re-run | Objek stateful (Proxy) | Class yang hidup sekali |
| State berubah → | Fungsi dijalankan ulang | Proxy terdeteksi, patch DOM | Observasi bedah, binding update |
| Menyimpan state | `useState` hook | `ref()` / `reactive()` | Properti class biasa |
| Bereaksi pada perubahan | `useEffect` + dep array | `watch()` / computed | `@observable` + `fooChanged()` |
| Akses DOM | `useRef` | Template `ref` attribute | `ref` binding di template |
| Data fetching timing | `useEffect` setelah render | `onMounted()` | `binding()` sebelum render |
| Sintaks template | JSX (markup di JS) | SFC `.vue` (HTML diperluas) | File HTML murni terpisah |
| DI / Service | Context API + hooks | Provide/Inject | `resolve()` — DI container penuh |

**Kesimpulan dari tabel:** Aurelia paling dekat ke Vue dalam hal "komponen sebagai objek stateful yang hidup", tapi berbeda dalam cara template ditulis (HTML murni vs SFC) dan cara services dikelola (DI container vs provide/inject). Aurelia paling jauh dari React dalam hampir semua aspek — ini memang dua budaya yang berbeda, bukan versi yang lebih baik atau lebih buruk.

---

## Contoh Nyata: Komponen dari Aurelia Ops

Berikut ini komponen login yang sesungguhnya dari proyek referensi course ini. Perhatikan betapa sederhana kodenya:

```typescript
// apps/web/src/pages/login-page.ts
import { resolve } from 'aurelia';
import { IRouter } from '@aurelia/router';
import { AuthService } from '../services/auth';

export class LoginPage {
  private router = resolve(IRouter);
  private auth = resolve(AuthService);

  email = '';
  password = '';
  error = '';
  submitting = false;

  async submit(event: Event) {
    event.preventDefault();
    this.error = '';
    this.submitting = true;

    try {
      await this.auth.login(this.email, this.password);
      await this.router.load('overview');
    } catch {
      this.error = 'Email atau password salah.';
    } finally {
      this.submitting = false;
    }
  }
}
```

Perhatikan:
- Tidak ada `useState` — `email`, `password`, `error`, `submitting` adalah properti biasa
- Tidak ada `useEffect` — tidak ada yang perlu di-setup dengan side effect
- Tidak ada `useCallback` — `submit` adalah method biasa
- Tidak ada JSX — template ada di file `.html` terpisah
- DI (Dependency Injection) via `resolve()` — kita bahas di Bab 7

Ini class TypeScript murni. Bisa di-unit test tanpa render ke DOM sama sekali.

---

## Praktik

Coba pahami perbedaan ini dengan latihan mental sederhana. Tanpa menulis kode dulu, bayangkan kamu ingin membuat komponen yang:

1. Menampilkan daftar nama
2. Punya tombol "Tambah" yang menambahkan nama baru ke daftar
3. Menampilkan jumlah total nama

Di React, kamu akan butuh:
- `useState` untuk array nama
- `useState` atau `useMemo` untuk jumlah total
- Handler untuk tombol

Di Aurelia, coba pikirkan:
- Properti apa yang perlu ada di class?
- Method apa yang perlu ada?
- Apakah kamu butuh decorator apapun?

```typescript
// Jawaban Aurelia — semua terasa "biasa saja"
export class NameList {
  names: string[] = ['Alice', 'Bob'];

  get total() {
    return this.names.length; // getter biasa, Aurelia otomatis observasi
  }

  addName() {
    this.names = [...this.names, `Orang ${this.total + 1}`];
  }
}
```

```html
<!-- name-list.html -->
<div>
  <p>Total: ${total} orang</p>
  <ul>
    <li repeat.for="name of names">${name}</li>
  </ul>
  <button click.trigger="addName()">Tambah</button>
</div>
```

Selesai. Tidak ada hook, tidak ada memo, tidak ada callback yang perlu di-manage.

---

## Ringkasan

- Di Aurelia, komponen adalah class TypeScript biasa yang diinstansiasi sekali
- Properti class diobservasi secara otomatis — perubahan langsung memperbarui DOM terkait
- Lifecycle hooks tersedia untuk kontrol lebih: `binding`, `bound`, `attached`, `detaching`, `unbinding`
- Getter dan computed property bekerja secara otomatis tanpa perlu `useMemo`
- Kode lebih mudah di-test karena tidak bergantung pada React runtime

---

## Selanjutnya

Kalau kamu dari background Angular, bab berikutnya akan sangat familiar sekaligus mengejutkan — kita akan lihat secara berdampingan apa yang sama dan apa yang berbeda antara Angular dan Aurelia.

Kalau kamu bukan dari Angular, kamu tetap bisa lanjut ke bab berikutnya, tapi bab 4 (Setup Project) mungkin lebih cocok sebagai langkah berikutnya.

→ [Bab 03: Untuk Ex-Angular](../bab-03/README.md)
→ [Bab 04: Setup Project](../bab-04/README.md)
