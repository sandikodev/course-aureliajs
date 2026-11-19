# Bab 01 — Kenapa Aurelia?

> Sebelum belajar *cara* pakai Aurelia, kamu perlu tahu *kenapa* Aurelia ada. Karena kalau kamu tahu sejarahnya, kamu akan paham kenapa framework ini terasa berbeda dari yang lain — dan kenapa itu justru kelebihannya.

---

## Drama di Balik Lahirnya Aurelia

Tahun 2012, seorang developer bernama Rob Eisenberg membuat framework bernama **Durandal**. Waktu itu React belum populer, Angular masih versi 1.x, dan Durandal adalah salah satu framework SPA paling elegan yang pernah ada — menggabungkan KnockoutJS dengan RequireJS dalam pola MVVM yang bersih.

Google melihat potensi Rob dan merekrutnya masuk ke tim inti Angular. Tugasnya: ikut merancang **Angular 2** dari nol.

Rob bergabung. Dan di sinilah konflik dimulai.

### Dua Visi yang Bertabrakan

Rob punya visi Angular 2 yang sangat jelas:
- Berbasis standar web terbuka (ES6, Web Components)
- Convention over configuration — penamaan file yang cerdas, bukan konfigurasi yang bertele-tele
- Tidak ada monkey-patching pada API browser
- Arsitektur yang bisa diuji (testable) tanpa setup rumit

Tim Google punya visi berbeda:
- Bahasa baru buatan Google: AtScript (kemudian dimerger ke TypeScript)
- Zone.js: sistem yang mem-patch semua API asynchronous browser secara global
- NgModule: lapisan registrasi komponen yang kaku
- Decorator yang sangat verbose untuk hal-hal sederhana

Dua visi ini tidak bisa disatukan.

**Desember 2014:** Rob Eisenberg mengundurkan diri dari tim Angular Google.

**Januari 2015:** Rob merilis **Aurelia** — framework yang menurutnya seharusnya menjadi Angular 2 jika Google tidak terjebak dalam kompleksitas enterprise.

> *"Aurelia is what Angular 2 should have been."*
> — Banyak developer yang pernah pakai keduanya

---

## Apa yang Membuat Aurelia Berbeda

Setelah tahu sejarahnya, sekarang kita lihat secara teknis apa yang membuat Aurelia terasa berbeda.

### 1. Tidak Ada Virtual DOM

React, Vue, dan banyak framework modern menggunakan Virtual DOM — representasi JavaScript dari DOM asli yang kemudian dibandingkan (di-diff) untuk menentukan apa yang perlu diupdate.

Aurelia tidak melakukan itu.

Aurelia bekerja **langsung di DOM asli**. Ketika kamu mengubah properti:

```typescript
// user-profile.ts
export class UserProfile {
  name = 'Alice';

  updateName() {
    this.name = 'Bob'; // ← Aurelia langsung update DOM yang terikat ke this.name
  }
}
```

Aurelia mendeteksi perubahan pada `this.name` dan **langsung memperbarui hanya node teks yang terikat** — tanpa membuat representasi virtual, tanpa diff, tanpa rekonsiliasi.

Hasilnya: performa setara Svelte atau SolidJS, tapi dengan DX yang terasa seperti Angular.

### 2. Tidak Ada Zone.js

Angular menggunakan Zone.js untuk mendeteksi kapan sesuatu berubah. Zone.js bekerja dengan cara mem-patch semua API asynchronous browser:
- `setTimeout`
- `Promise`
- `fetch`
- `addEventListener`

Dan banyak lagi. Setiap kali salah satunya dipanggil, Angular tahu ada kemungkinan state berubah dan menjalankan change detection.

Masalahnya: ini overhead yang nyata, sulit di-debug, dan sering jadi sumber bug tersembunyi.

Aurelia tidak butuh Zone.js. Sistemnya bernama **ObserverLocator** — ia memantau properti secara bedah, hanya properti yang benar-benar terikat ke template. Tidak ada global patching.

### 3. HTML adalah HTML

Di React, kamu menulis JSX:

```tsx
// React
function UserCard({ name, email }) {
  return (
    <div className="card">
      <h3>{name}</h3>
      <p>{email}</p>
    </div>
  );
}
```

JSX bukan HTML. Ini JavaScript yang terlihat seperti HTML. Kamu tidak bisa validasi dengan HTML linter, kamu tidak bisa format dengan Prettier HTML, dan desainer yang tidak familiar JavaScript akan kebingungan.

Di Aurelia, template adalah file HTML biasa:

```html
<!-- user-card.html -->
<div class="card">
  <h3>${name}</h3>
  <p>${email}</p>
</div>
```

Dan logic-nya di file TypeScript terpisah:

```typescript
// user-card.ts
import { bindable } from 'aurelia';

export class UserCard {
  @bindable name = '';
  @bindable email = '';
}
```

File HTML bisa dibuka di browser, divalidasi HTML linter, dan dibaca oleh siapapun yang tahu HTML — termasuk desainer.

### 4. TypeScript Murni, Tanpa Magic

Angular menggunakan decorators dengan metadata yang dikompilasi secara khusus. React pakai JSX transform. Vue pakai SFC parser tersendiri.

Aurelia menggunakan **TypeScript standard** — stage-3 decorators sesuai spesifikasi TC39. Tidak ada transform khusus, tidak ada plugin kompilasi tambahan untuk fitur dasar.

```typescript
// Ini valid TypeScript murni, tidak ada magic
import { customElement, bindable } from 'aurelia';

@customElement('user-card')
export class UserCard {
  @bindable name = '';
}
```

---

## Empat Klaster Paradigma Frontend (Penting untuk Dipahami)

Sebelum lanjut, kamu perlu pahami sesuatu yang jarang dibahas di komunitas frontend: framework-framework modern tidak tinggal di satu "dunia" yang sama. Mereka tinggal di **empat klaster budaya yang berbeda**, dan membandingkan lintas-klaster adalah perbandingan apel dengan jeruk.

**Klaster 1 — Functional Re-Render (React, Preact, Solid, Qwik)**

Komponen = fungsi. State berubah → fungsi dieksekusi ulang → Virtual DOM (React/Preact) atau signals (Solid/Qwik) yang memicu update langsung tanpa re-run. Mental model: UI = f(state). JSX adalah sintaks utama — markup hidup di dalam JavaScript.

**Klaster 2 — Compiler-First SFC (Svelte, Astro)**

Framework bukan runtime yang di-download, tapi compiler yang bekerja saat build. Sintaks khusus (`.svelte`, `.astro`) dikompilasi menjadi kode imperatif. Tidak ada runtime framework di browser. Mental model: tulis dalam satu file, compiler urus reaktivitas.

**Klaster 3 — Pragmatic Reactivity SFC (Vue)**

Vue sering dianggap rival React, tapi secara teknis Vue lebih dekat ke Svelte. Komponen adalah objek stateful yang hidup (bukan fungsi yang re-run). Reaktivitas via ES6 Proxy — mutation langsung terdeteksi tanpa VDOM overhead besar. Format SFC `.vue` mirip Svelte, tapi tidak dikompilasi jadi kode imperatif — reactivity tetap runtime. Vue punya tiga "rasa" API (Options, Composition, `<script setup>`), yang membuat developer dari React, Angular, maupun Svelte semua merasa "familiar" — tapi pendekatan teknis Vue berbeda dari ketiganya.

**Klaster 4 — Enterprise Class OOP (Angular, Aurelia)**

Komponen = class TypeScript dengan lifecycle yang jelas. Dependency Injection sebagai arsitektur inti. Template HTML terpisah dari logic. Tidak ada JSX, tidak ada SFC, tidak ada "fungsi yang re-run", tidak ada Proxy. Hanya class dan observasi. Mental model: kelas yang hidup, properti yang diobservasi — cara berpikir backend enterprise yang dipindahkan ke frontend.

**Kenapa ini penting?**

Ketika kamu membandingkan Aurelia dengan React atau Vue, kamu tidak membandingkan "mana yang lebih baik" — kamu membandingkan budaya yang berbeda. Developer yang tumbuh di React tidak akan nyaman di Aurelia bukan karena Aurelia lebih buruk, tapi karena mental model-nya fundamental berbeda. Developer Vue mungkin lebih cepat adaptasi karena Vue punya "rasa OOP" lewat Options API, tapi mekanisme reaktivitasnya tetap berbeda.

Sebaliknya, developer Angular yang pindah ke Aurelia langsung klik karena budayanya sama.

Perbandingan yang adil untuk Aurelia adalah dengan Angular — karena mereka berada di klaster yang sama. Dan di sana, Angular lebih terbukti: versi 22 (stabil, Google-backed), ekosistem matang, tooling lengkap. Aurelia masih versi 2.0.0-rc.2 (release candidate, belum stable). Aurelia menang di simplisitas dan ukuran bundle, tapi belum punya bukti produksi skala seperti Angular.

Ini bukan alasan untuk tidak belajar Aurelia — tapi ini realita yang harus kamu tahu sebelum menjual skill ini ke klien.

> Dokumen lengkap tentang posisi framework termasuk sejarah Vue dan signals konvergensi: lihat `FRAMEWORK_POSITIONING.md`

---

## Mengapa Ini Penting untuk Kariermu

Kamu mungkin bertanya: kalau Aurelia sebagus ini, kenapa tidak sepopuler React?

Jawabannya sederhana: **marketing dan ekosistem**, bukan kualitas teknis. Tapi juga: Aurelia masih versi RC (release candidate), bukan stable release. Angular sudah versi 22. React sudah versi 19. Vue sudah 3.5. Semua kompetitor sudah matang.

Ada pelajaran dari sejarah di sini. Dua framework lahir dari frustrasi terhadap AngularJS di tahun yang berdekatan:
- **Vue (2014)** — Evan You, bekas engineer Google AngularJS, membangun Vue sebagai "AngularJS yang lebih pragmatis". Vue berhasil mendapat adopsi masif karena timing, dokumentasi luar biasa, dan marketing yang tepat.
- **Aurelia (2015)** — Rob Eisenberg, bekas arsitek Angular 2 di Google, membangun Aurelia sebagai "Angular 2 yang seharusnya". Aurelia tetap niche meski kualitas teknis-nya unggul di banyak aspek.

Dua proyek, satu motivasi, outcome yang sangat berbeda. Pelajarannya: kualitas teknis saja tidak cukup. Marketing, timing, dan ekosistem menentukan adopsi.

React punya Facebook. Angular punya Google. Vue punya komunitas yang sangat aktif. Aurelia punya... komunitas kecil tapi sangat loyal.

Dan justru di situlah peluangnya — tapi dengan realita yang jujur.

Coba cek Upwork sekarang. Search "Aurelia developer". Kamu akan menemukan 15–30 job aktif, kebanyakan berupa:
- Rescue project: app Aurelia 1 yang perlu diupgrade atau diperbaiki
- Migration: Angular 1 ke Aurelia
- Greenfield: perusahaan enterprise di Eropa/Australia yang sudah committed ke Aurelia

Sekarang lihat kandidatnya. Sangat sedikit. Hampir tidak ada yang punya pengalaman nyata.

**Kamu tidak bersaing melawan 50 developer Aurelia. Kamu bersaing melawan 2–3.**

Tapi jujur: 15–30 job adalah jumlah kecil. Ini bukan niche yang akan membuatmu kaya. Ini niche yang akan memberi kamu akses ke proyek yang hampir tidak ada kompetitor — dengan trade-off bahwa proyek tersebut jarang muncul.

Dan dengan menyelesaikan course ini, kamu sudah masuk ke kelompok yang sangat kecil itu.

---

## Aurelia di Dunia Nyata

Supaya ini tidak terasa teoritis, berikut gambaran nyata dari proyek yang akan jadi referensi course ini:

**Aurelia Ops** adalah SaaS admin dashboard production-ready dengan:
- Multi-tenant workspace dengan Stripe billing
- 1.000 baris data seeded, server-side pagination dan filtering
- Auth dengan session cookie (Argon2id), team invitation, role management
- Dibangun dengan Aurelia 2 + Hono + Drizzle + Postgres

Kode yang kamu pelajari di course ini bukan teori — ini kode yang sudah berjalan di production shape.

---

## Ringkasan

- Aurelia lahir dari konflik antara Rob Eisenberg dan Google soal arah Angular 2
- Perbedaan teknis utama: Direct-DOM (tidak ada Virtual DOM), tidak ada Zone.js, HTML murni, TypeScript standard
- Aurelia bukan framework yang lebih populer — tapi justru itu yang membuatnya menarik sebagai niche karier
- Course ini tidak hanya mengajarkan framework, tapi juga cara monetisasi skill ini

---

## Selanjutnya

Di bab berikutnya, kita akan membongkar mental model paling penting yang perlu kamu pahami sebelum menulis satu baris pun kode Aurelia: **mengapa komponen = class biasa itu bukan keterbatasan, tapi kekuatan.**

→ [Bab 02: Mental Model Baru](../bab-02/README.md)


---

## Mengapa Course Ini Berbeda dari Tutorial Aurelia yang Lain

Ada beberapa tutorial Aurelia di internet — kebanyakan sudah outdated atau fokus ke versi 1. Course ini berbeda karena:

**1. Berbasis proyek nyata.**
Semua contoh kode diambil dari Aurelia Ops — SaaS admin dashboard yang dibangun di production shape. Bukan todo app. Bukan counter. Aplikasi dengan multi-tenancy, billing Stripe, auth Argon2id, dan 1.000 baris data seeded.

**2. Jujur tentang keterbatasan.**
Aurelia 2 masih RC.2. Angular sudah versi 22. Job Aurelia di Upwork hanya 15–30 aktif. Course ini tidak menyembunyikan realita ini — justru memanfaatkannya sebagai bagian dari strategi yang jujur.

**3. Framework positioning yang benar.**
Aurelia bukan rival React atau Vue — mereka hidup di klaster paradigma yang berbeda. Course ini menjelaskan ini dari awal supaya kamu bisa berbicara tentang Aurelia dengan positioning yang akurat, bukan dengan klaim yang menyesatkan.

**4. Target karier yang spesifik.**
Bukan "pelajari Aurelia karena bagus." Tapi "pelajari Aurelia karena ini niche dengan demand nyata dan hampir tidak ada kompetitor qualified."

Kalau kamu datang ke sini dengan ekspektasi belajar framework yang akan menggantikan React — kamu datang ke tempat yang salah. Kalau kamu datang dengan ekspektasi memahami framework yang punya ceruk pasar yang nyata dan spesifik dengan kompetisi yang sangat rendah — kamu di tempat yang tepat.
