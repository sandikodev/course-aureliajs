# Framework Positioning: Kenapa Perbandingan Lintas-Budaya Menyesatkan

> **Dokumen referensi untuk course ini.** Baca sebelum membuat perbandingan antar framework.
> **Dibuat:** 2026-11-18 — berdasarkan analisis paradigma dan data versi terkini.

---

## 1. Data Versi Terkini (November 2026)

| Framework | Versi | Status | Paradigma |
|-----------|-------|--------|-----------|
| Angular | 22.2.6 | Stabil, Google-backed | Enterprise OOP + DI |
| Aurelia | 2.0.0-rc.2 | Release candidate (belum stable) | Enterprise OOP + DI |
| React | 19.x | Stabil, Meta-backed | Functional re-render + VDOM |
| Preact | 11.2.0 | Stabil | React-lite + VDOM |
| SolidJS | 2.0.0 | Stabil | Fine-grained signals, no VDOM |
| Svelte | 5.57.3 | Stabil (Runes era) | Compiler-first SFC |
| **Vue** | **3.5.45** | **Stabil** | **Pragmatic Reactivity (Proxy-based SFC)** |
| Qwik | 1.0.57 | Stabil | Resumability, zero-hydration |
| Astro | 7.4.12 | Stabil | Islands, multi-framework host |
| Lit | 3.3.3 | Stabil | Web Components standard (W3C) |

**Meta-frameworks (ekosistem yang lebih lengkap):**

| Framework | Meta-Framework | Versi |
|-----------|---------------|-------|
| React | Next.js | 16.4.6 |
| Vue | Nuxt | 4.6.4 |
| Svelte | SvelteKit | (bundled dalam Svelte) |
| Solid | SolidStart | 2.2.3 |
| Angular | Analog.js (community) / Angular Universal (SSR built-in) | — |
| Aurelia | **Tidak ada yang matang** — `@hono/aurelia-ssr` masih PoC | — |

Ketiadaan meta-framework yang matang adalah salah satu gap terbesar Aurelia dibanding semua kompetitornya.

**Fakta yang harus diakui:** Aurelia masih RC. Semua kompetitor sudah stable. Angular sudah 22 major releases. Vue 3.5. Svelte 5. Solid 2.0. Dan semua sudah punya meta-framework produksi kecuali Aurelia.

---

## 2. Empat Klaster Paradigma

Framework-framework di atas tidak tinggal di satu "dunia frontend" — mereka tinggal di empat klaster budaya yang berbeda. Perbandingan lintas-klaster adalah perbandingan apel dengan jeruk.

### Klaster 1 — Functional Re-Render (React-sphere)

```
React, Preact, SolidJS, Qwik
```

**Budaya:** Komponen = fungsi. State berubah → fungsi dieksekusi ulang (React/Preact) atau signals yang memicu update langsung tanpa re-run (Solid/Qwik).

**Mental model:** UI = f(state). Developer berpikir dalam transformasi data, bukan dalam objek yang hidup. Fungsi adalah unit komposisi utama.

**Mekanisme rendering:**
- React/Preact: VDOM tree → diff → commit ke real DOM. Komponen dieksekusi ulang setiap state berubah.
- Solid: Fine-grained signals, komponen "run once" saat inisialisasi. Setiap signal update langsung menyentuh hanya node DOM yang terikat — tidak ada re-render komponen.
- Qwik: Resumability — tidak ada hydration. Event handler di-serialize langsung ke atribut DOM, di-download on-demand saat user berinteraksi.

**Sintaks:** JSX/TSX (markup ditulis di dalam JavaScript)

**Ironi di klaster ini:** React dan Solid keduanya disebut "functional" tapi mekanismenya berlawanan. React re-run fungsi terus menerus; Solid run fungsi sekali saja dan pakai signals untuk update. Svelte dan Solid lebih mirip mekanismenya (direct DOM update) meski sintaksnya berbeda.

### Klaster 2 — Compiler-First SFC (Svelte-sphere)

```
Svelte, Astro
```

**Budaya:** Framework bukan runtime yang di-download, tapi compiler yang bekerja saat build. Sintaks khusus (`.svelte`, `.astro`) dikompilasi menjadi kode imperatif JavaScript yang langsung manipulasi DOM.

**Mental model:** Tulis HTML+JS+CSS dalam satu file sebagai unit semantik yang utuh, lalu compiler yang urus reaktivitas dan DOM mutation. Tidak ada runtime framework di browser (atau sangat sedikit).

**Mekanisme rendering:**
- Svelte 5 (Runes): `$state()`, `$derived()`, `$effect()` — compiler mengubah ini menjadi kode mutasi DOM imperatif. Runes adalah syntax sugar untuk signals yang dikompilasi, bukan runtime.
- Astro: Islands architecture — halaman default adalah HTML statis murni tanpa JavaScript. Komponen interaktif dari framework apapun (React, Vue, Svelte, Solid) disematkan sebagai "pulau" yang di-hydrate on-demand.

**Sintaks:** SFC proprietary (`.svelte`, `.astro`) — terlihat seperti HTML diperluas.

### Klaster 3 — Pragmatic Reactivity SFC (Vue-sphere)

```
Vue
```

Vue sering diposisikan sebagai "rival React" dalam diskusi populer. Ini pemahaman yang praktis tapi tidak akurat secara teknis. Vue lebih dekat ke Svelte daripada ke React — baik dalam cara penulisan (SFC `.vue`), model reaktivitas (Proxy-based, bukan immutability), maupun filosofi (komponen adalah unit stateful yang hidup, bukan fungsi yang re-run).

**Vue punya lapisan API yang memberi "rasa" berbeda tergantung yang dipakai:**

```
Vue 3 Options API       → terasa seperti Klaster 4 (OOP feel, mirip Angular era lama)
Vue 3 Composition API   → terasa seperti Klaster 1 (hooks feel, mirip React hooks)
Vue 3 <script setup>    → terasa seperti Klaster 2 (SFC feel, paling mirip Svelte)
```

Ini yang membuat Vue begitu adaptable — developer dari React, Angular, maupun Svelte semua bisa merasa "sudah familiar" saat baru menyentuh Vue. Tapi pendekatan teknis Vue berbeda dari ketiganya.

**Budaya:** Komponen adalah objek stateful yang diinisialisasi sekali. Reaktivitas bekerja via ES6 Proxy yang membungkus state — tidak ada VDOM overhead yang besar, tidak ada Zone.js, tidak ada compiler yang mengubah seluruh sintaks. Vue adalah "framework yang paling tidak memaksa opinionated approach".

**Mekanisme rendering:**
- State dibungkus Proxy (`ref()`, `reactive()`) — akses dan mutasi terdeteksi secara otomatis
- VDOM yang dioptimasi dengan patch flags — lebih efisien dari React karena Vue compiler bisa tandai node statis
- `<script setup>` dikompilasi ke kode yang lebih efisien, tapi ini optimasi, bukan perubahan paradigma seperti Svelte

**Sintaks:** SFC `.vue` dengan `<template>`, `<script setup>`, `<style>` — satu file, tapi tidak sepenuhnya dikompilasi seperti Svelte

**Perbandingan tepat untuk Vue:** Vue vs Svelte (SFC format, reaktivitas non-VDOM-heavy) lebih bermakna daripada Vue vs React (yang sering dipakai tapi kurang akurat secara teknis).

**Meta-framework:** Nuxt 4 — SSR, file-based routing, composables, hybrid rendering. Paling matang di kelasnya setelah Next.js.

### Klaster 4 — Enterprise Class OOP (Angular-sphere)

```
Angular, Aurelia
```

**Budaya:** Komponen = class TypeScript dengan lifecycle yang jelas. Dependency Injection sebagai arsitektur inti. Template HTML terpisah dari logic. Tidak ada JSX, tidak ada SFC, tidak ada "fungsi yang re-run", tidak ada Proxy. Hanya class dan observasi direktif.

**Mental model:** Kelas yang hidup, properti yang diobservasi, service yang diinjeksi. Ini adalah cara berpikir backend (Spring, NestJS, .NET) yang dipindahkan ke frontend. Developer yang nyaman dengan backend enterprise akan langsung klik.

**Mekanisme rendering:**
- Angular 22: Zone.js (legacy, bisa di-disable) + Signals native (modern). Change detection tree dari root, tapi bisa dioptimasi dengan `OnPush` atau fully zoneless.
- Aurelia 2: ObserverLocator — observasi properti secara bedah via getter/setter. Tidak ada Zone.js, tidak ada change detection tree. Hanya properti yang terikat ke template yang diobservasi.

**Sintaks:** HTML murni (file `.html` terpisah dari `.ts`). Binding via atribut (`.bind`, `.trigger`, `.for`). Tidak ada tag khusus selain elemen HTML valid.

---

## 3. Rivalitas Populer vs Realita Teknis

Dua pasangan rivalitas yang populer di komunitas tapi kurang akurat secara teknis:

### React vs Vue — Rivalitas yang Menyesatkan

Di komunitas, React dan Vue sering disebut "pesaing langsung". Di job market, keduanya bersaing untuk posisi "frontend developer". Tapi secara teknis, pendekatan mereka berbeda:

| Aspek | React (Klaster 1) | Vue (Klaster 3) |
|-------|-------------------|-----------------|
| Komponen | Fungsi yang re-run | Objek stateful yang hidup |
| Reaktivitas | Immutability + VDOM diff | Proxy-based, mutation langsung |
| Sintaks | JSX (markup di dalam JS) | SFC `.vue` (HTML diperluas) |
| Mental model | Transformasi data | Komponen reaktif yang hidup |
| Template | Tidak ada (JSX adalah kode) | Terpisah dari logic (`<template>`) |

**Yang lebih mirip Vue secara teknis:** Svelte. Keduanya SFC, keduanya state sebagai objek yang bisa dimutasi (bukan immutable), keduanya lebih "HTML-first" daripada React. Perbedaannya: Svelte compiler mengubah seluruh reaktivitas jadi kode imperatif; Vue runtime Proxy yang menangani reaktivitas.

### React vs Svelte — Klaster Berbeda tapi Sering Dibandingkan

Ini juga comparison populer tapi tidak apple-to-apple. React adalah runtime yang re-run fungsi; Svelte adalah compiler yang eliminasi runtime. Membandingkan mereka seperti membandingkan interpretasi vs kompilasi — bukan masalah "mana lebih baik" tapi "trade-off apa yang kamu pilih".

---

## 4. Konvergensi Signals — Semua Framework Menuju Arah yang Sama

Tren paling signifikan dalam 3 tahun terakhir: **semua framework sedang konvergen ke signals** sebagai primitif reaktivitas.

```
React     → useSignal() di React 19 experimental (belum masuk core)
Vue       → ref() / reactive() — sudah signals sejak Vue 3 (2020)
Angular   → signal(), computed(), effect() — native sejak Angular 16 (2023), matang di Angular 22
Svelte 5  → $state Runes — semangatnya signals, dikompilasi
Solid     → createSignal() — pelopor, dari awal (2019)
Qwik      → useSignal() — dari awal
Preact    → @preact/signals — library terpisah
```

**Posisi Aurelia dalam tren ini:**

Aurelia sejak awal sudah pakai konsep yang setara signals melalui **ObserverLocator** — getter/setter yang mendeteksi perubahan properti secara bedah dan langsung update hanya binding yang relevan. Ini secara konseptual identik dengan fine-grained signals Solid atau Vue's reactivity.

Tapi Aurelia tidak mendapat credit untuk ini karena:
1. Namanya berbeda ("ObserverLocator", bukan "signals")
2. Tidak ada marketing atau blog post yang menjelaskan kesetaraan ini
3. Komunitas kecil tidak punya suara yang cukup keras

Ini bisa diinterpretasikan dua cara:
- **"Aurelia ahead of its time"** — sudah ada konsep ini sebelum signals menjadi tren
- **"Branding yang lemah"** — tidak berhasil mengkomunikasikan keunggulan ini saat momentumnya ada

Keduanya benar sekaligus.

---

## 5. Kemampuan Agnostik: Host Framework Lain

Beberapa framework bisa "menjalankan" framework lain di atasnya, tapi dengan mekanisme dan filosofi yang berbeda:

### Astro — Islands Architecture

Bisa host React, Vue, Svelte, Solid, Preact sebagai island interaktif di dalam halaman statis.

**Mekanisme:** Setiap island adalah komponen dari framework tersebut, di-bundle terpisah, dan hanya di-hydrate saat visible. Astro tidak "menjalankan" React — Astro menyertakan React runtime hanya untuk island yang membutuhkannya.

**Filosofi:** Content-first. Halaman default adalah HTML statis. Interaktivitas adalah pengecualian, bukan aturan.

### Lit — Web Components Standard

Menghasilkan Custom Elements yang bisa dipakai di framework apapun karena mematuhi standar W3C.

**Mekanisme:** Lit mengkompilasi komponen menjadi Custom Element class yang terdaftar di browser. Framework apapun bisa pakai `<my-lit-component>` seperti elemen HTML biasa — React, Vue, Svelte, Angular, semua bisa.

**Filosofi:** Standar web sebagai jembatan. Tidak ada framework lock-in karena outputnya adalah W3C Custom Elements.

### Vue — `defineCustomElement()`

Vue 3 juga bisa kompilasi komponen menjadi W3C Custom Elements via `defineCustomElement()`.

**Mekanisme:** Sama seperti Lit — output adalah Custom Element yang bisa dipakai di framework manapun.

**Catatan:** Kemampuan ini ada di Vue tapi tidak sering digunakan karena ekosistem Vue sudah cukup lengkap untuk tidak perlu keluar dari Vue.

### Aurelia — `Aurelia.enhance()`

Bisa augmentasi HTML yang sudah ada di DOM dengan behavior Aurelia.

**Mekanisme:** Aurelia parse template di existing DOM, pasang binding instructions, dan aktifkan observasi. Berguna untuk progressive enhancement di halaman yang sudah server-rendered.

**Filosofi:** Enhancement, bukan replacement. Ini bukan untuk "menjalankan React di atas Aurelia".

**Penting:** Kemampuan ini lebih terbatas dari yang sering diklaim. Tidak sama dengan Astro islands atau Lit Web Components.

### Angular — Angular Elements (`@angular/elements`)

Menghasilkan Custom Elements dari komponen Angular yang bisa dipakai di framework lain.

**Mekanisme:** Membawa seluruh Angular runtime di dalam setiap custom element — overhead besar.

**Filosofi:** Kompatibilitas via W3C standard, tapi dengan cost yang signifikan.

### Qwik — `qwikify$()`

Bisa "host" komponen React, Vue, Svelte dan menjadikannya resumable.

**Mekanisme:** Qwik membungkus komponen dari framework lain. Experimental.

**Filosofi:** Resumability untuk semua — tapi belum matang.

**Kesimpulan:** Kemampuan agnostik ada di hampir semua framework, tapi mekanisme dan trade-off-nya berbeda. Astro paling matang dan paling praktis. Lit dan Vue paling standar (W3C). Aurelia dan Angular paling berat. Qwik paling eksperimental.

---

## 6. Aurelia vs Angular: Satu-Satunya Perbandingan yang Adil

Karena mereka berada di klaster yang sama, comparison ini bermakna:

| Dimensi | Angular 22 | Aurelia 2.0.0-rc.2 |
|---------|------------|---------------------|
| Versi stabil terkini | 22.2.6 (major releases reguler) | RC.2 (belum stable) |
| Backing | Google + komunitas besar | Komunitas kecil |
| DI | Matang, `@Injectable`, hierarchical | `resolve()`, lebih sederhana |
| Change detection | Zone.js (legacy) + Signals native (v22) | ObserverLocator, no Zone.js |
| Zoneless mode | Tersedia sejak Angular 18, stabil di v22 | Tidak relevan (tidak ada Zone.js dari awal) |
| Bundle size | ~60–120 KB | ~30 KB |
| Signals | Native sejak Angular 16, matang di Angular 22 | ObserverLocator (konsep ekuivalen, nama berbeda) |
| SSR / Meta-framework | Angular Universal + Analog.js (community) | `@hono/aurelia-ssr` PoC — belum ada meta-framework matang |
| Testing | Matang (Jasmine, Jest, Vitest + TestBed) | `createFixture`, terbatas dokumentasinya |
| CLI | `ng` — scaffold, generate, serve, test, deploy, migrate | `npx makes aurelia` — minimal |
| Component library | Angular Material, PrimeNG, NG-Zorro, Clarity | Tidak ada resmi (port manual) |
| DevTools | Angular DevTools (browser extension) | Tidak ada |
| Docs | Sangat lengkap, terawat, contoh banyak | Cukup, ada gap untuk use case lanjut |
| Job market | Ratusan job aktif di seluruh dunia | 15–30 job aktif, mayoritas rescue/migration |
| Stability | Production-proven di ribuan enterprise global | Belum ada bukti skala production |
| Konvergensi signals | Terdepan — signals sudah matang di v22 | Konsep awal ada, tapi tidak di-evolusikan ke signals modern |

**Kesimpulan jujur:** Angular lebih terbukti di hampir semua dimensi kecuali ukuran bundle dan simplisitas setup awal. Aurelia menang di cleanliness dan zero-boilerplate, tapi cleanliness tanpa bukti produksi skala adalah hipotesis, bukan fakta.

Satu pergeseran penting: **keunggulan "tidak ada Zone.js" yang dulu jadi alasan utama pilih Aurelia di atas Angular, sekarang sudah kehilangan eksklusivitasnya.** Angular 22 sudah stabil dengan zoneless mode. Developer Angular modern sudah bisa dapat reaktivitas yang bersih tanpa Zone.js — sambil tetap mendapat ekosistem Angular yang jauh lebih matang.

---

## 7. Sejarah Koneksi: Semua Berakar dari Ketidakpuasan terhadap AngularJS

Ada benang merah historis yang jarang disebut — tiga framework besar lahir sebagai reaksi terhadap AngularJS:

**Vue — Evan You, 2014:**
Evan You bekerja di Google menggunakan AngularJS. Vue lahir sebagai "AngularJS yang saya mau pakai sendiri" — lebih ringan, lebih pragmatis, tanpa overhead DI dan module system yang kompleks. Vue berhasil mendapat adopsi masif karena timing, marketing, dan dokumentasi yang jauh lebih baik dari Aurelia.

**Aurelia — Rob Eisenberg, 2015:**
Rob Eisenberg ikut merancang Angular 2 di Google. Ketika visi keduanya bentrok (Rob ingin standards-first, Google pilih zone.js dan NgModule), Rob keluar dan membuat Aurelia — "Angular 2 yang seharusnya".

**React — Jordan Walke, 2013:**
React bukan reaksi terhadap Angular — lahir dari kebutuhan Facebook menangani UI yang kompleks. Tapi popularitas React mempercepat kematian AngularJS sebagai pilihan utama.

Jadi secara historis, Vue dan Aurelia sama-sama "lahir dari frustrasi terhadap AngularJS" — tapi dengan output yang sangat berbeda. Vue memilih pragmatisme dan berhasil. Aurelia memilih kemurnian arsitektur dan tetap niche.

---

## 8. Posisi Aurelia di Pasar

Aurelia bukan pengganti Angular. Aurelia bukan alternatif React atau Vue. Aurelia adalah **niche spesialis**:

- Rescue/maintenance dari Aurelia 1.x yang sudah deployed
- Migration dari Angular 1 / AngularJS ke modern framework ringan
- Greenfield project untuk tim yang sudah familiar Angular tapi mau sesuatu yang lebih ringan
- Perusahaan enterprise di Eropa/Australia yang sudah committed ke Aurelia

**Ini posisi yang jujur.** Bukan "Aurelia is better than React/Vue" (menyesatkan, lintas-budaya). Bukan "Aurelia is the new Angular" (Angular sudah 22, Aurelia masih RC). Tapi "Aurelia adalah pilihan spesialis untuk tim tertentu dengan kebutuhan tertentu."

---

## 9. Konteks Ekosistem Indonesia — Job Aurelia adalah Job Export

Ini konteks penting yang sering diabaikan:

**Di Indonesia, job market frontend didominasi:**
- React / Next.js — startup, e-commerce, fintech lokal
- Vue / Nuxt — startup menengah, banyak di Vietnam dan datang ke Indonesia
- Angular — enterprise perbankan dan telko (legacy sistem, bukan greenfield)
- Aurelia — **hampir tidak ada secara lokal**

Artinya: **belajar Aurelia untuk job lokal Indonesia adalah strategi yang salah.** Aurelia adalah skill yang dijual ke luar negeri — Eropa, Australia, Amerika — via platform seperti Upwork, Toptal, atau remote job boards.

Implikasi praktis untuk pembaca course ini:
1. **Bahasa Inggris** wajib — proposal Upwork, komunikasi dengan klien, dokumentasi PR
2. **Timezone** — klien Eropa (UTC+1/+2) atau Australia (UTC+8/+10/+11), bukan Indonesia (UTC+7)
3. **Pricing dalam USD/EUR** — tarif lokal tidak berlaku. Rescue Audit $400–800 adalah wajar untuk klien Eropa
4. **Profil Upwork harus dalam Inggris** — tidak ada benefit menulis profil dalam Bahasa Indonesia
5. **Invoice dan kontrak** — perlu memahami basic freelance contract dalam Inggris

Ini bukan kekurangan — ini realita yang perlu diterima sejak awal supaya strategi karier bisa dibuat dengan tepat.

---

## 10. Aturan untuk Course Ini

Berdasarkan analisis di atas, berikut aturan yang harus diikuti di semua chapter course:

1. **Jangan bilang "Aurelia lebih baik dari React/Vue/Svelte"** — itu perbandingan lintas-budaya yang menyesatkan
2. **Bisa bilang "Aurelia lebih bersih dari Angular"** — itu perbandingan dalam-klaster yang valid
3. **Vue bukan rival React secara teknis** — Vue lebih dekat ke Svelte. Rivalry React-Vue adalah persepsi pasar, bukan realita teknis
4. **Selalu sebut status RC Aurelia** — jangan sembunyikan bahwa Aurelia 2 belum stable
5. **Jangan oversell job market** — 15–30 job adalah kecil, kelebihannya adalah absennya kompetitor
6. **Angular 22 sudah punya zoneless mode** — keunggulan "tidak ada Zone.js" Aurelia tidak lagi eksklusif
7. **Aurelia adalah skill export, bukan skill lokal Indonesia** — job hampir pasti dari klien luar negeri
8. **Kemampuan agnostik disebut dengan konteks mekanismenya** — "enhance" Aurelia berbeda dari "islands" Astro
9. **Signals konvergensi adalah konteks penting** — Aurelia ahead of its time, tapi tidak mendapat credit karena branding lemah
