# Aurelia 2: Framework yang Berbeda

> Course dalam Bahasa Indonesia tentang Aurelia 2 — framework frontend yang dirancang untuk menyelesaikan masalah Angular dengan cara yang lebih elegan.

## Tentang Course Ini

Aurelia 2 bukan framework baru yang harus kamu pelajari dari nol. Ini adalah penyempurnaan dari ide-ide terbaik Angular, dikerjakan oleh orang yang sama yang ikut merancang Angular 2 di Google — Rob Eisenberg.

Satu hal yang harus kamu tahu dari awal: **Aurelia 2 masih release candidate (RC.2)**. Angular sudah versi 22. Semua framework populer lain sudah stable. Ini bukan alasan untuk tidak belajar Aurelia — tapi ini realita yang harus dipahami supaya kamu bisa menjual skill ini dengan jujur.

Course ini untuk kamu yang:
- Sudah kenal Angular/AngularJS dan mau tahu kenapa Aurelia terasa lebih bersih
- Sudah bisa React/Vue dan penasaran dengan pendekatan class-based tanpa virtual DOM
- Ingin masuk ke niche freelance yang kompetisinya sangat rendah di platform seperti Upwork

## Struktur Course

| Chapter | Topik |
|---------|-------|
| Bab 01 | Kenapa Aurelia? Sejarah, filosofi, dan tiga klaster paradigma frontend |
| Bab 02 | Mental model baru: komponen = class biasa (perbandingan dengan React) |
| Bab 03 | Untuk ex-Angular: persamaan, perbedaan, dan evaluasi jujur Angular 22 vs Aurelia RC.2 |
| Bab 04 | Setup project dengan npx makes aurelia + latihan |
| Bab 05 | Binding lengkap: .bind, .trigger, value converters, binding behaviors, containerless + latihan |
| Bab 05b | Forms & Validation: semua input types, `@aurelia/validation`, error display + latihan |
| Bab 06 | Komponen: @bindable, custom attributes, HTML-only, au-slot, EventAggregator + latihan |
| Bab 07 | Dependency Injection: resolve(), HTTP client, interceptors, service pattern + latihan |
| Bab 07b | Testing: createFixture, Vitest, mocks, routing tests + latihan |
| Bab 08 | Routing: @route, nested routing, canLoad, 404 handling, route metadata + latihan |
| Bab 09 | Studi kasus: multi-tenant SaaS dari Aurelia Ops + latihan |
| Bab 09b | Production Build & Deploy: Cloudflare, Vercel, VPS/Nginx, Docker + latihan |
| Bab 10 | Aurelia di dunia kerja: Upwork strategy, proposal template, offer ladder + latihan |
| Bab 11 | Portfolio builder: fork, modifikasi, deploy Aurelia Ops + latihan |

## Cara Menggunakan

Pilih jalur baca sesuai background kamu:

**Ex-Angular/AngularJS:** Bab 1 → 3 → 5 → 7 → 9 → 10 → 11

**React/Vue developer:** Bab 1 → 2 → 4 → 5 → 05b → 6 → 8 → 9

**Freelance hunter:** Bab 1 → 10 → 11, lalu pilih salah satu jalur di atas

**Jalur lengkap (semua bab berurutan):** Disarankan untuk pembaca yang mau menguasai semua aspek — teknis, testing, deploy, dan karier.

## Dokumen Referensi

| File | Isi |
|------|-----|
| `MASTER_PLAN.md` | Panduan pengerjaan course, keputusan desain, status semua bab |
| `FRAMEWORK_POSITIONING.md` | **Wajib baca** — analisis tiga klaster paradigma frontend, kenapa Aurelia tidak bisa dibandingkan dengan React/Svelte, perbandingan jujur Angular 22 vs Aurelia RC.2 |

## Referensi Kode

Semua contoh kode di course ini diambil dari atau terinspirasi proyek nyata:

**[Aurelia Ops Dashboard](https://github.com/sandikodev/aureliajs)** — SaaS admin dashboard production-ready dengan Aurelia 2 (RC.2), Hono, Drizzle ORM, PostgreSQL, dan Stripe. Multi-tenant, 1.000 baris data seeded, auth, billing, team management. Dipakai sebagai referensi studi kasus di Bab 09 dan panduan portfolio di Bab 11.

## Penulis

**Sandikodev** — TypeScript/Aurelia engineer  
TikTok & Facebook: @sandikodev  
Produk digital: lynk.id/sandikodev

---

> Panduan pengerjaan dan keputusan desain: lihat `MASTER_PLAN.md`  
> Analisis positioning framework: lihat `FRAMEWORK_POSITIONING.md`
