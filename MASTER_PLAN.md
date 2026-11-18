# MASTER PLAN — Course Aurelia 2: Framework yang Berbeda

> **Dokumen ini adalah satu-satunya referensi untuk seluruh pekerjaan course ini.**
> Baca ini sebelum mengerjakan apapun. Update dokumen ini setiap ada keputusan baru.
>
> **Lokasi project:** `/home/dev/bukudigital/aureliajs/`
> **Referensi codebase nyata:** `/home/dev/project/aureliajs/` (Aurelia Ops Dashboard)
> **Penulis:** Sandikodev (@sandikodev)
> **Terakhir diupdate:** 2026-11-18 — Semua 15 chapter selesai ✅ (11 bab asli + 4 bab tambahan)

---

## 1. Visi & Tujuan

Membuat course Aurelia 2 dalam Bahasa Indonesia yang:

1. **Dijual di lynk.id** sebagai produk digital (ebook PDF + starter kit)
2. **Dipecah menjadi konten pendek** untuk TikTok dan Facebook @sandikodev sebagai funnel gratis
3. **Memposisikan pembaca** bisa mendapat pekerjaan freelance via Upwork dengan niche Aurelia, bahkan tanpa pengalaman real project Aurelia sebelumnya — asalkan familiar dengan Angular atau frontend umum

Ini bukan sekadar tutorial framework. Ini course strategi karier dengan Aurelia sebagai alatnya.

---

## 2. Produk & Harga

| Produk | Format | Harga |
|--------|--------|-------|
| Ebook PDF "Aurelia 2: Framework yang Berbeda" | PDF + project starter | Rp 97.000–149.000 |
| Konten TikTok/Facebook | Video pendek gratis | Funnel ke lynk.id |

**Platform distribusi:** lynk.id  
**Konten promosi:** TikTok @sandikodev, Facebook @sandikodev

---

## 3. Target Pembaca (3 Persona)

### Persona A — "Angular Refugee" ⭐ PRIORITAS UTAMA
- Background Angular/AngularJS, frustrasi dengan zone.js dan NgModule
- Langsung klik dengan Aurelia karena DNA sama (Rob Eisenberg)
- **Jalur baca:** Bab 1 → 3 → 5 → 7 → 9 → 10 → 11
- **Hook:** "Aurelia adalah Angular yang seharusnya"

### Persona B — "Frontend Generalist"
- Bisa React/Vue tapi belum paham DI dan class-based architecture
- Butuh jembatan mental model: kenapa `this.name = 'x'` langsung update DOM
- **Jalur baca:** Bab 1 → 2 → 4 → 5 → 6 → 8 → 9
- **Hook:** "Frontend tanpa useState, tanpa virtual DOM, tanpa magic"

### Persona C — "Upwork Hunter"
- Sengaja cari niche langka untuk kompetisi rendah
- Belum tentu pengalaman Angular, tapi strategis
- **Jalur baca:** Bab 1 → 10 → 11 → (pilih Track A atau B)
- **Hook:** "15–30 job Aurelia di Upwork setiap saat, hampir nol kompetitor qualified"

---

## 4. Pesan Kunci yang Harus Konsisten di Semua Chapter

1. **Aurelia dan Angular berada di klaster yang sama** (Enterprise Class OOP) — perbandingan dengan Angular valid, perbandingan dengan React/Svelte/Solid adalah perbandingan lintas-budaya yang menyesatkan
2. **Direct-DOM bukan kelemahan, tapi keunggulan** — tidak ada virtual DOM overhead, tidak ada zone.js, tidak ada magic
3. **Aurelia masih RC.2** — jangan sembunyikan status ini. Angular sudah versi 22, Aurelia belum stable. Kejujuran tentang ini justru membangun kepercayaan
4. **Niche ini bukan soal banyaknya job, tapi absennya kompetitor** — 15-30 job adalah kecil, tapi hampir nol yang qualified
5. **Angular lebih terbukti di hampir semua dimensi** — ekosistem, tooling, stability, job market. Aurelia menang di simplisitas, tapi cleanliness tanpa bukti produksi skala adalah hipotesis
6. **Referensi nyata tersedia:** repo `/home/dev/project/aureliajs/` adalah SaaS dashboard production-ready
7. **Lihat `FRAMEWORK_POSITIONING.md`** untuk analisis lengkap tentang tiga klaster paradigma dan aturan perbandingan

---

## 5. Outline Chapter Lengkap

```
bab-01/  Kenapa Aurelia? (Sejarah, Rob Eisenberg, Direct-DOM)
bab-02/  Mental Model Baru: Komponen = Class Biasa
bab-03/  Untuk Ex-Angular: Perbedaan & Persamaan (head-to-head)
bab-04/  Setup Project (npx makes aurelia, Vite, TypeScript) + latihan
bab-05/  Binding: .bind, .trigger, repeat.for, if.bind + value converters + containerless + latihan
bab-05b/ Forms & Validation (input types, validation, & validate, error display) + latihan
bab-06/  Komponen & Komunikasi (@bindable, EventAggregator) + custom attributes + HTML-only + containerless + latihan
bab-07/  Dependency Injection: resolve(), interface, singleton + HTTP Client mendalam + latihan
bab-07b/ Testing dengan Vitest + createFixture (mocks, routing tests) + latihan
bab-08/  Routing: @route, canLoad, navigate, parameter + nested routing + 404 + latihan
bab-09/  Studi Kasus: Dashboard SaaS sederhana (dari Aurelia Ops)
bab-09b/ Production Build & Deploy (Cloudflare, Vercel, VPS, Docker)
bab-10/  Aurelia di Dunia Kerja: Upwork Strategy, proposal template
bab-11/  Portfolio Builder: Pakai Aurelia Ops sebagai bukti kerja
```

**Total target:** ~180–220 halaman PDF

---

## 6. Format Penulisan

### Gaya Bahasa
- Bahasa Indonesia informal tapi profesional (tidak terlalu formal, tidak slang)
- Sapaan: "kamu" (bukan "Anda", bukan "lo")
- Nama variabel, file, kode: tetap English
- Tidak ada basa-basi panjang — langsung ke poin

### Struktur Setiap Chapter
```
1. Pembuka (masalah yang akan diselesaikan, 1–2 paragraf)
2. Konsep inti (penjelasan, bisa dengan analogi)
3. Kode contoh (selalu ada, selalu bisa dijalankan)
4. Perbandingan (dengan React/Angular/Vue kalau relevan)
5. Praktik (latihan singkat yang bisa dikerjakan)
6. Ringkasan (3–5 poin bullet)
7. Penutup chapter (teaser bab berikutnya)
```

### Format Kode
- Semua kode harus bisa dijalankan (bukan pseudocode)
- Gunakan TypeScript strict
- Selalu sertakan nama file di atas blok kode
- Annotasi inline untuk baris penting

---

## 7. Referensi Codebase (Aurelia Ops)

Repo `/home/dev/project/aureliajs/` dipakai sebagai referensi nyata di chapter 9 dan 11.

File kunci yang relevan untuk course:
| File | Relevan untuk |
|------|---------------|
| `apps/web/src/app.ts` | Bab 2, 8 — lifecycle, routing |
| `apps/web/src/config/routes.ts` | Bab 8 — routing config |
| `apps/web/src/pages/projects-page.ts` | Bab 5, 6 — binding, komponen |
| `apps/web/src/ui/*.aux` | Bab 6 — SFC format .aux |
| `apps/web/src/services/auth.ts` | Bab 7 — DI service |
| `apps/api/src/middleware/tenant.ts` | Bab 9, 11 — studi kasus |
| `packages/shared/src/schemas.ts` | Bab 9 — shared types |

Versi Aurelia yang dipakai di repo: `2.0.0-rc.2`
TypeScript: stage-3 decorators (`experimentalDecorators: false`)

---

## 8. Status Pengerjaan

| Chapter | Status | Catatan |
|---------|--------|---------|
| MASTER_PLAN.md | ✅ Done | Dokumen ini — diperbarui dengan aturan positioning |
| FRAMEWORK_POSITIONING.md | ✅ Done | Tiga klaster paradigma, Angular 22 vs Aurelia RC.2, kemampuan agnostik, aturan perbandingan |
| README.md | ✅ Done | Overview course untuk pembaca |
| bab-01/ | ✅ Done | Kenapa Aurelia + tiga klaster paradigma + framing RC status + peluang karier jujur |
| bab-02/ | ✅ Done | Mental Model Baru + catatan "perbedaan budaya bukan mana lebih baik" |
| bab-03/ | ✅ Done | Untuk Ex-Angular + tabel Angular 22 vs Aurelia RC.2 + Angular signals & keunggulan |
| bab-04/ | ✅ Done | Setup Project + latihan buat halaman baru |
| bab-05/ | ✅ Done | Binding + value converters + binding behaviors + containerless + latihan |
| bab-05b/ | ✅ Done | Forms & Validation — input types, model.bind, validation, error display + latihan |
| bab-06/ | ✅ Done | Komponen + custom attributes + HTML-only + containerless + au-compose + latihan |
| bab-07/ | ✅ Done | DI + HTTP Client (IHttpClient, interceptors, retry, AbortController, pola fetch) + latihan |
| bab-07b/ | ✅ Done | Testing — createFixture, assertions, trigger events, mocks, routing tests + latihan |
| bab-08/ | ✅ Done | Routing + nested routing + 404 handling + route metadata + latihan |
| bab-09/ | ✅ Done | Studi Kasus — satu seam tenancy, DB schema, GET+POST /projects, shared types, error contract |
| bab-09b/ | ✅ Done | Production Build & Deploy — Cloudflare Pages, Vercel, VPS/Nginx, Docker, manual chunks |
| bab-10/ | ✅ Done | Upwork Strategy + 3 fakta jujur (RC status, job volume, posisi bukan pengganti Angular) |
| bab-11/ | ✅ Done | Portfolio Builder — fork+modif+deploy Aurelia Ops, case study, jawaban interview teknis |

---

## 9. Konten TikTok/Facebook (Mapping ke Chapter)

Setiap chapter menghasilkan 2–3 konten pendek:

| Chapter | Ide Konten Pendek |
|---------|-------------------|
| Bab 1 | "Kenapa Angular 2 gagal jadi Angular yang seharusnya" |
| Bab 1 | "Rob Eisenberg: orang yang keluar dari Google dan bikin framework sendiri" |
| Bab 1 | "3 klaster paradigma frontend: mana yang cocok untuk kamu?" |
| Bab 2 | "Frontend tanpa useState — class biasa bisa reactive?" |
| Bab 3 | "Angular vs Aurelia: 5 perbedaan yang langsung kelihatan" |
| Bab 3 | "Angular 22 vs Aurelia RC.2: jujur, siapa yang lebih terbukti?" |
| Bab 5 | "Binding di Aurelia: ini bukan React, ini budaya yang berbeda" |
| Bab 7 | "Dependency Injection di frontend: ini bukan cuma Angular" |
| Bab 10 | "15–30 job Aurelia di Upwork, hampir nol yang bisa handle" |
| Bab 10 | "Aurelia masih RC — bagaimana menjual ini tanpa berbohong?" |
| Bab 11 | "Cara pakai satu repo GitHub jadi portofolio yang menjual" |

---

## 10. Instruksi untuk Sesi Baru

Kalau kamu (AI/Kiro) membuka sesi baru dan diminta melanjutkan course ini:

1. Baca file ini dulu: `/home/dev/bukudigital/aureliajs/MASTER_PLAN.md`
2. Cek tabel status di bagian 8 untuk tahu apa yang sudah selesai
3. Baca chapter terakhir yang sudah selesai sebagai referensi gaya penulisan
4. Ikuti format chapter di bagian 6
5. Referensi kode selalu dari `/home/dev/project/aureliajs/` — jangan karang sendiri
6. Update tabel status di bagian 8 setelah selesai mengerjakan chapter

---

## 11. Keputusan yang Sudah Final (Jangan Diubah Tanpa Diskusi)

- Bahasa: Indonesia
- Format: Markdown → PDF
- Harga: Rp 97.000–149.000
- Platform: lynk.id
- Versi Aurelia: 2.0.0-rc.2 (sesuai repo referensi)
- Gaya sapaan: "kamu"
- Tidak ada bab tentang .aux format (terlalu eksperimental, bisa bingungkan pemula)
- Bab 10 (Upwork) harus jujur soal volume job — tidak oversell
- Jangan pernah bilang "Aurelia lebih baik dari React/Vue/Svelte" — itu perbandingan lintas-budaya yang menyesatkan (lihat FRAMEWORK_POSITIONING.md)
- Selalu sebut status RC Aurelia saat relevan — terutama di Bab 03 dan Bab 10
- Angular 22 (versi stabil saat ini) lebih terbukti dari Aurelia RC.2 — akui ini secara jujur

## 12. Dokumen Referensi Tambahan

| File | Isi |
|------|-----|
| `MASTER_PLAN.md` | Panduan utama course ini |
| `FRAMEWORK_POSITIONING.md` | Analisis tiga klaster paradigma frontend — wajib baca sebelum membuat perbandingan |
| `README.md` | Overview course untuk pembaca |
