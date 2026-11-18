# Bab 10 — Aurelia di Dunia Kerja: Strategi Upwork

> Ini bukan bab tentang cara "menjadi kaya dengan freelance". Ini bab tentang cara memposisikan skill Aurelia yang baru kamu pelajari agar bisa mendapat pekerjaan nyata — dengan ekspektasi yang realistis dan strategi yang konkret.

---

## Realita yang Harus Kamu Akui Dulu

Sebelum strategi, ada tiga fakta yang harus kamu internalisasi:

**1. Aurelia 2 masih release candidate.** Per November 2026, versi terbaru adalah `2.0.0-rc.2`. Itu artinya belum stable release. Angular sudah versi 22. Svelte 5. Solid 2.0. Semua kompetitor sudah matang. Klien enterprise yang ragu pakai RC punya alasan yang valid.

**2. 15–30 job adalah jumlah kecil.** Jangan pernah bilang ke klien "banyak job Aurelia di Upwork". Bilang yang sebenarnya: "job Aurelia ada, dan hampir tidak ada kompetitor yang qualified." Itu dua pernyataan yang berbeda.

**3. Aurelia bukan pengganti React atau Angular.** Aurelia adalah niche spesialis untuk tim yang sudah familiar dengan pola Angular dan mau sesuatu yang lebih ringan. Posisikan diri sebagai spesialis, bukan generalis yang kebetulan bisa Aurelia.

---

## Realita Pasar Aurelia di Upwork

Mari mulai dengan angka yang jujur. Setiap saat, ada sekitar **15–30 job aktif** di Upwork yang spesifik menyebut Aurelia. Itu kecil — jauh lebih kecil dari React (ribuan job) atau Angular (ratusan job).

Tapi itu justru keunggulannya.

Dari 15–30 job itu, berapa banyak kandidat yang benar-benar qualified? Rata-rata 2–5 orang. Dibanding niche React yang satu job bisa dilamar 50–200 orang, peluangmu jauh lebih besar di Aurelia.

**Jenis job yang paling sering muncul:**

| Jenis | Deskripsi | Frekuensi |
|-------|-----------|-----------|
| Rescue/debugging | App Aurelia 1 yang rusak atau lambat, butuh diperbaiki | ★★★★ |
| Migration | Aurelia 1 → Aurelia 2, atau Angular 1 → Aurelia | ★★★ |
| Feature tambahan | Perusahaan yang sudah punya app Aurelia, butuh fitur baru | ★★★ |
| Greenfield | Membangun app baru dari nol dengan Aurelia | ★★ |
| Konsultasi | Review arsitektur, mentoring tim internal | ★ |

**Fokus ke rescue dan migration** — di sana peluang paling besar. Klien yang punya app Aurelia yang rusak tidak mau diajarkan; mereka mau orang yang langsung bisa kerja.

**Catatan jujur tentang RC status:** Untuk job rescue (Aurelia 1), status RC Aurelia 2 tidak relevan — klien butuh orang yang paham Aurelia 1. Untuk job migration (Aurelia 1 → 2), klien sudah aware mereka pindah ke RC dan sudah terima risikonya. Untuk greenfield, kamu harus jujur: "Aurelia 2 masih RC, tapi roadmap ke stable sudah jelas dan codebase yang ditulis sekarang akan kompatibel."

---

## Positioning yang Benar

Kesalahan paling umum adalah memposisikan diri sebagai "Aurelia Developer". Ini membatasi kamu ke pool job yang sudah kecil, dan klien baru yang belum kenal Aurelia tidak akan search dengan keyword itu.

Posisi yang lebih kuat:

> "TypeScript/frontend engineer yang mengkhususkan diri pada framework enterprise — termasuk Aurelia, Angular, dan class-based architecture. Kalau framework kamu langka dan butuh penanganan orang yang benar-benar paham, itulah tempat saya."

Perbedaannya:
- "Aurelia Developer" → klien harus sudah tahu dia butuh Aurelia
- "Engineer yang bisa masuk ke codebase Aurelia yang sudah ada" → klien yang punya masalah dengan app yang ada

**Profil Upwork — bagian yang paling penting dibaca:**

```
Judul (120 karakter):
TypeScript Frontend Engineer — Angular, Aurelia & Enterprise SPA Specialist

Overview (500 karakter pertama yang terlihat sebelum "See more"):
Kalau tim kamu punya aplikasi Aurelia atau Angular yang butuh
diselamatkan, dimigrasi, atau dikembangkan lebih lanjut — itulah
yang saya kerjakan.

Saya masuk ke codebase yang sudah ada, memahami arsitekturnya,
dan menyelesaikan masalah tanpa harus rewrite semuanya dari nol.
```

---

## Menulis Proposal yang Menang

Ini template yang bisa langsung dipakai. Maksimal 180 kata, karena klien membaca di mobile.

```
Hi [nama],

Kamu menyebutkan [parafrase masalah dengan kata-kata klien].
Bagian yang paling penting untuk diselesaikan adalah [identifikasi
risiko spesifik yang disebutkan atau tersirat].

Rencana saya untuk minggu pertama:
1. [Langkah teknis konkret — bisa diverifikasi]
2. [Langkah kedua]
3. [Test atau deliverable yang bisa dilihat klien]

Satu pertanyaan sebelum saya bisa estimasi dengan tepat:
- [Pertanyaan teknis spesifik tentang stack/versi/arsitektur]

Kalau berguna, saya bisa kirim 1 halaman rencana + estimasi dalam
24 jam setelah saya lihat repository-nya — atau 20 menit call dan
saya kasih tahu langsung apakah saya bisa bantu atau tidak.

[Nama kamu]
[Link portfolio/demo]
```

**Mengapa format ini bekerja:**
- Parafrase masalah klien → bukti kamu membaca, bukan copas
- Rencana minggu pertama → konkret, bisa diverifikasi
- Satu pertanyaan teknis → sinyal kamu paham domain-nya
- Penawaran "atau saya bilang tidak bisa" → menurunkan risiko klien

**Yang TIDAK perlu ada di proposal:**
- "I am a passionate developer with 8 years of experience..."
- Daftar 20 teknologi yang kamu kuasai
- "100% satisfaction guaranteed"
- Link ke semua portfolio tanpa konteks

---

## Proposal untuk Job Rescue (Contoh Nyata)

Misalnya ada job posting seperti ini:
> "Our Aurelia 1.x app has memory leak issues and very slow performance after adding new features. Need someone who can diagnose and fix."

Proposal yang tepat:

```
Hi [nama],

Memory leak di Aurelia 1 biasanya ada di salah satu dari tiga
tempat: subscription EventAggregator yang tidak di-dispose saat
komponen detach, interval/timeout yang berjalan terus, atau
komponen yang di-destroy tapi masih di-reference.

Rencana minggu pertama:
1. Profiling dengan Chrome Memory Profiler — identifikasi retained
   objects setelah navigate antar halaman
2. Audit semua komponen yang subscribe ke EventAggregator, cek
   apakah ada yang tidak dispose() di detached()
3. Kirim laporan: penyebab pasti, estimasi effort fix, dan apakah
   ada quick win yang bisa diselesaikan dalam 1–2 hari

Pertanyaan: Apakah masalah terjadi setelah navigasi ke halaman
tertentu, atau setelah buka halaman berulang kali? Ini penting
untuk menentukan apakah masalahnya di lifecycle atau di service.

Kalau mau, saya bisa mulai dengan audit 3 jam (fixed rate) untuk
memberikan diagnosis lengkap sebelum kita putuskan scope perbaikan.

[Nama] — link ke repo Aurelia Ops sebagai bukti kerja
```

---

## Penetapan Harga

Jangan langsung tawarkan "cheapest rate". Strategi yang lebih efektif:

### Offer Ladder: 3 Ukuran

**Tingkat 1 — Rescue Audit (pintu masuk)**
- Durasi: 3–5 hari kerja
- Harga: $400–800
- Isi: baca repo, identifikasi 5–10 masalah nyata, laporan 2–3 halaman + 30 menit call
- Ini hampir selalu berlanjut ke pekerjaan lebih besar

**Tingkat 2 — Feature Sprint**
- Durasi: 1–3 minggu
- Harga: $1.000–4.000
- Isi: satu fitur vertical lengkap — bisa diklik, ada test, ada dokumentasi singkat

**Tingkat 3 — Embedded (retainer)**
- Durasi: 3–6 bulan, 10–20 jam/minggu
- Harga: $1.500–3.500/bulan
- Ini target akhir — kamu jadi bagian dari tim, bukan vendor

**Untuk job pertama:** Mulai dari Tingkat 1. Harga rendah, risiko klien rendah, kamu dapat project nyata untuk portfolio. Setelah selesai dengan bagus, Tingkat 2 dan 3 jadi keputusan mudah bagi klien.

---

## Ritme Kerja yang Membuat Klien Memperpanjang Kontrak

Ini bagian yang paling diabaikan tapi paling besar dampaknya. Developer yang bisa kerja sama "seperti anggota tim" jauh lebih mudah diperpanjang kontraknya daripada yang kerja bagus tapi tidak komunikasi.

**Setiap Senin — 5 baris update:**
```
Minggu lalu: [X] selesai, [Y] masih dalam progress
Minggu ini: target [Z]
Risiko: [ada/tidak, jelaskan kalau ada]
Butuh dari kamu: [kalau ada yang perlu dari klien]
```

**Setiap PR/commit:**
- Deskripsi singkat: apa yang diubah, kenapa, cara test
- Screenshot kalau ada perubahan UI
- Mention kalau ada keputusan yang kamu buat dan alasannya

**Setiap 2 minggu:**
- Demo singkat 10 menit (bisa async via Loom)
- Klien lihat progress nyata, bukan hanya laporan teks

Kalau kamu melakukan tiga hal ini secara konsisten, dalam 2–3 minggu klien mulai bertanya "menurut kamu sebaiknya bagaimana?" — itu tanda kamu sudah dianggap partner, bukan vendor.

---

## Screening Klien: Siapa yang Harus Diterima

Tidak semua klien layak diterima. Berikut tanda klien yang bagus vs yang akan membuang waktumu:

**Klien yang bagus:**
- Bisa menjelaskan masalah dengan spesifik
- Punya repo yang bisa dilihat (atau mau share setelah NDA)
- Sudah punya developer internal, butuh bantuan spesifik
- Budget sesuai scope yang dibicarakan
- Mau interview/call teknis sebelum kontrak

**Hindari (atau harga 3x lipat):**
- "Buat aplikasi seperti X, budget $200, deadline 3 hari"
- Tidak mau kasih akses repo sebelum kontrak tanda tangan
- Sudah ganti 3 developer dan semua dibilang "tidak kompeten"
- Scope berubah setiap pesan
- Tanya harga sebelum menjelaskan masalah

---

## 5 Pertanyaan untuk Interview Teknis

Kalau ada call dengan klien potensial, pertanyaan ini membantu kamu evaluate situasi dan sekaligus menunjukkan kompetensi:

1. **"Apa yang membuat kamu cari bantuan sekarang, bukan 3 bulan lalu?"** — cari tahu urgensi nyata
2. **"Siapa yang akan review PR saya?"** — ada tim teknis atau kamu sendirian?
3. **"Kalau pekerjaan ini selesai bagus, apa yang terjadi selanjutnya?"** — ada kemungkinan kontrak berlanjut?
4. **"Versi Aurelia yang dipakai di production sekarang apa?"** — kompetensi langsung terlihat dari caramu bertanya ini
5. **"Apa yang sudah dicoba dan gagal?"** — hindari mengulang kesalahan yang sama

---

## Rencana 30/60/90 Hari

**Hari 1–30: Bangun Bukti**
- [ ] Deploy demo Aurelia Ops (atau fork yang sudah dimodifikasi) ke URL publik
- [ ] README 1 layar: cara jalan, akun demo, link case study
- [ ] Profil Upwork: judul, overview, 1 portfolio item dengan konteks
- [ ] Kirim 10 proposal yang ditulis dengan format di atas, bukan template
- [ ] Target: minimal 3 balasan

**Hari 31–60: Konversi Percakapan**
- [ ] 3 call dengan klien potensial, gunakan 5 pertanyaan screening
- [ ] 1 Rescue Audit terjual — ini yang paling penting
- [ ] Jalankan ritme update mingguan untuk klien pertama

**Hari 61–90: Naik Level**
- [ ] 1 Sprint atau retainer dari Rescue Audit pertama
- [ ] Naikkan harga 20–30% untuk klien baru
- [ ] Minta 1 testimoni yang menyebut hasil nyata (bukan "orangnya baik")

---

## Latihan Bab 10

### Latihan: Tulis Proposal Pertamamu

Ambil job posting Aurelia nyata dari Upwork (atau buat skenario sendiri), lalu tulis proposal maksimal 180 kata menggunakan format di bab ini.

**Skenario contoh:**
> "We have a legacy Aurelia 1.9 app with performance issues after adding new features. Users complain about slow navigation. Need help diagnosing and fixing. Budget: $500–$1500. Timeline: ASAP."

<details>
<summary>Contoh proposal yang baik</summary>

```
Hi [nama],

Kamu menyebut navigasi lambat setelah fitur baru ditambahkan.
Yang paling sering jadi penyebab di Aurelia 1 adalah subscription
EventAggregator yang tidak di-dispose, atau komponen yang masih
hidup di memori setelah navigate pergi.

Rencana minggu pertama:
1. Profiling dengan Chrome Memory Profiler — identifikasi retained
   objects setelah navigate antar halaman (1–2 jam)
2. Audit komponen yang subscribe ke EventAggregator tanpa dispose
   di detached() (2–3 jam)
3. Laporan: penyebab pasti + estimasi effort fix

Satu pertanyaan: masalah terjadi setelah navigate ke halaman
tertentu, atau makin lambat setelah buka berulang kali?
Ini menentukan apakah root cause-nya di lifecycle atau di service.

Saya bisa mulai dengan Rescue Audit 2 hari ($400) untuk diagnosis
lengkap sebelum kita putuskan scope perbaikan.

[Nama] — link demo Aurelia Ops sebagai referensi kerja
```

**Yang benar di proposal ini:**
- Parafrase masalah klien dengan kata teknis yang tepat (bukan copy paste)
- Rencana konkret yang bisa diverifikasi
- Satu pertanyaan teknis yang menunjukkan kamu paham domain
- Offer ladder dimulai dari Rescue Audit — risiko rendah bagi klien

</details>

---

## Konteks Penting: Job Aurelia adalah Job Export, Bukan Job Lokal

Ini satu hal yang jarang dibahas tapi sangat berpengaruh pada strategi karier.

**Di Indonesia, Aurelia hampir tidak ada secara lokal.** Job market frontend Indonesia didominasi React (startup, e-commerce, fintech), Vue (startup menengah), dan Angular (enterprise perbankan dan telko legacy). Aurelia di Indonesia? Hampir nol.

Artinya: **belajar Aurelia untuk job lokal Indonesia adalah strategi yang salah.** Aurelia adalah skill yang dijual ke luar negeri — Eropa Barat, Australia, Amerika Utara — via platform seperti Upwork, Toptal, atau remote job boards.

Implikasi praktis yang perlu kamu siapkan:

**Bahasa Inggris adalah wajib, bukan opsional.**
Proposal Upwork, komunikasi dengan klien, comment di kode, deskripsi PR, laporan mingguan — semua dalam Inggris. Tidak perlu sempurna, tapi harus bisa dipahami. Klien yang bayar $400–800 untuk Rescue Audit akan membaca dengan teliti.

**Timezone adalah realita yang harus dikelola.**
- Klien Eropa Barat: UTC+1/+2 (selisih 5–6 jam dari WIB)
- Klien Australia Timur: UTC+10/+11 (selisih 3–4 jam dari WIB, lebih mudah)
- Klien Amerika: UTC-5 sampai UTC-8 (selisih 12–15 jam, paling sulit)

Dalam proposal dan kontrak, selalu klarifikasi overlap jam kerja. Minimal 2–3 jam overlap sehari adalah standar yang wajar.

**Pricing dalam USD atau EUR, bukan IDR.**
Rate freelance Upwork untuk niche spesialis seperti Aurelia: $45–80/jam untuk yang baru mulai, $80–150/jam setelah punya 2–3 proyek sukses. Rescue Audit $400–800 adalah wajar untuk klien Eropa.

Jangan konversi ke IDR saat menetapkan harga — itu mindset yang salah. Hargai skill berdasarkan pasar klien, bukan pasar lokal.

**Profil Upwork harus dalam Inggris.**
Tidak ada benefit menulis profil dalam Bahasa Indonesia. Klien yang cari Aurelia developer adalah klien internasional.

**Invoice dan basic contract perlu dipahami.**
Minimal tahu cara buat invoice sederhana dalam USD, pahami term seperti "net 15", "50% upfront", dan cara kerja Upwork escrow. Ini bukan hal yang sulit tapi perlu dipelajari sebelum kontrak pertama.

---

## Ringkasan

- 15–30 job Aurelia aktif di Upwork setiap saat — kecil, tapi hampir tidak ada kompetitor qualified
- Aurelia masih RC.2 — jangan sembunyikan, tapi frame dengan konteks yang tepat per jenis job
- **Job Aurelia adalah job export** — hampir pasti dari klien Eropa, Australia, atau Amerika, bukan Indonesia
- Posisikan diri sebagai "engineer yang bisa masuk ke codebase yang ada", bukan "Aurelia developer"
- Proposal: parafrase masalah klien, rencana minggu pertama, satu pertanyaan teknis — maksimal 180 kata
- Offer ladder: Rescue Audit (pintu masuk) → Sprint → Retainer
- Ritme komunikasi (Senin update, PR deskripsi, demo 2 mingguan) yang membedakan vendor dari partner
- Bahasa Inggris, timezone management, USD pricing adalah prasyarat — bukan skill bonus

> Untuk memahami posisi Aurelia di ekosistem frontend yang lebih luas termasuk konteks ekosistem Indonesia: baca `FRAMEWORK_POSITIONING.md`.

---

## Selanjutnya

Strategi sudah ada. Yang tersisa adalah **bukti**. Di bab terakhir kita bahas cara menggunakan Aurelia Ops — proyek yang sudah kamu pelajari sepanjang course ini — sebagai portfolio yang menjual.

→ [Bab 11: Portfolio Builder](../bab-11/README.md)
