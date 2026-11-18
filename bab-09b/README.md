# Bab 09b — Production Build & Deploy

> Aplikasi yang berjalan di `localhost` bukan produk. Bab ini membahas cara build Aurelia 2 untuk production dan deploy ke berbagai platform — dari yang paling sederhana sampai yang paling kompleks.

---

## Build Production

### Perintah Build

```bash
pnpm build
```

Ini menjalankan Vite build yang menghasilkan:

```
dist/
├── index.html         ← HTML shell (sudah inject script + css)
├── assets/
│   ├── app-[hash].js  ← Bundle JS (hashed untuk caching)
│   ├── app-[hash].css ← Bundle CSS
│   └── vendor-[hash].js ← Dependencies (kalau pakai manualChunks)
```

**Yang perlu di-perhatikan:**

1. **`experimentalDecorators: false`** di `tsconfig.json` — build akan error kalau salah
2. **`<base href="/">`** di `index.html` — tanpa ini, deep link tidak bekerja di production
3. **SPA fallback** — server harus mengembalikan `index.html` untuk semua route yang tidak match file statis

### Ukuran Bundle

Aurelia 2 production build:
- **Core runtime:** ~30 KB gzipped
- **Dengan router:** ~40 KB gzipped
- **Dengan Tailwind CSS 4:** tambah ~10 KB gzipped

Total untuk SPA sederhana: **~50–60 KB gzipped** — jauh lebih kecil dari React (~130 KB) atau Angular (~120 KB).

### Manual Chunks untuk Optimasi

Di `vite.config.ts`, kamu bisa kontrol code splitting:

```typescript
import { defineConfig } from 'vite';
import aurelia from '@aurelia/vite-plugin';

export default defineConfig({
  plugins: [aurelia()],
  build: {
    target: 'es2022',
    rolldownOptions: {
      output: {
        manualChunks(id) {
          if (id.includes('node_modules/aurelia') || id.includes('node_modules/@aurelia')) {
            return 'vendor-aurelia'; // pisahkan Aurelia core
          }
          if (id.includes('node_modules/zod')) {
            return 'vendor-zod'; // pisahkan Zod
          }
        },
      },
    },
  },
});
```

Hasilnya:
```
dist/assets/
├── app-[hash].js          ← Kode aplikasi
├── vendor-aurelia-[hash].js ← Aurelia core (cached lama)
├── vendor-zod-[hash].js    ← Zod (cached lama)
```

Bundle vendor jarang berubah antara deploy, jadi browser cache mereka lebih lama.

---

## Deploy ke Cloudflare Pages (Paling Mudah)

Cloudflare Pages cocok untuk SPA static — gratis, CDN global, otomatis.

1. Build project:
```bash
pnpm build
```

2. Deploy via Wrangler:
```bash
npx wrangler pages deploy dist --project-name aurelia-app
```

Atau connect repository GitHub ke Cloudflare Pages dashboard:
- **Build command:** `pnpm build`
- **Output directory:** `dist`
- **SPA fallback:** `/index.html` (ini penting — set di dashboard)

**SPA fallback** memastikan semua route (seperti `/about`, `/dashboard`) mengembalikan `index.html` agar router bisa resolve.

---

## Deploy ke Vercel

```bash
npm i -g vercel
vercel
```

Atau buat `vercel.json`:

```json
{
  "buildCommand": "pnpm build",
  "outputDirectory": "dist",
  "rewrites": [
    { "source": "/(.*)", "destination": "/index.html" }
  ]
}
```

Field `rewrites` adalah SPA fallback — semua route yang tidak match file statis return `index.html`.

---

## Deploy ke VPS dengan Nginx

### Build

```bash
pnpm build
```

### Upload ke Server

```bash
# Copy dist/ ke server
rsync -avz dist/ user@server:/var/www/aurelia-app/
```

### Nginx Config

```nginx
server {
    listen 80;
    server_name app.example.com;
    root /var/www/aurelia-app;
    index index.html;

    # SPA fallback — semua route return index.html
    location / {
        try_files $uri /index.html;
    }

    # Cache statis (hashed assets) — immutable
    location /assets/ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # Gzip
    gzip on;
    gzip_types text/plain text/css application/javascript application/json;
    gzip_min_length 1000;
}
```

**Penjelasan:**
- `try_files $uri /index.html` — kalau file tidak ada, return index.html (SPA fallback)
- `expires 1y` untuk `/assets/` — file sudah hashed, aman cache permanen
- `gzip on` — kompresi untuk transfer lebih cepat

---

## Deploy dengan Docker

### Dockerfile (Multi-stage)

```dockerfile
# Stage 1: Build
FROM node:22-alpine AS builder
WORKDIR /app

RUN npm install -g pnpm

COPY package.json pnpm-lock.yaml ./
RUN pnpm install --frozen-lockfile

COPY . .
RUN pnpm build

# Stage 2: Serve dengan nginx
FROM nginx:alpine

# Copy build output ke nginx
COPY --from=builder /app/dist /usr/share/nginx/html

# SPA fallback config
RUN echo 'server { \
    listen 80; \
    location / { \
        try_files $uri /index.html; \
    } \
    location /assets/ { \
        expires 1y; \
        add_header Cache-Control "public, immutable"; \
    } \
}' > /etc/nginx/conf.d/default.conf

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### docker-compose.yml

```yaml
version: '3.8'
services:
  web:
    build: .
    ports:
      - "8080:80"
    restart: unless-stopped
```

### Build & Run

```bash
docker build -t aurelia-app .
docker run -p 8080:80 aurelia-app
```

Buka `http://localhost:8080`.

---

## Deploy SPA + API Bersamaan (Pola Aurelia Ops)

Aurelia Ops deploy sebagai satu container yang menyajikan API + static web:

```dockerfile
# Multi-stage: build web + build API + serve together
FROM node:22-alpine AS web-builder
WORKDIR /app
RUN npm install -g pnpm
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml ./
COPY apps/web apps/web
COPY packages packages
RUN pnpm install --frozen-lockfile
RUN pnpm --filter @aureliajs/web build

FROM node:22-alpine AS api-builder
WORKDIR /app
RUN npm install -g pnpm
COPY . .
RUN pnpm install --frozen-lockfile
RUN pnpm --filter @aureliajs/api build

FROM node:22-alpine
WORKDIR /app
COPY --from=api-builder /app/apps/api/dist ./dist
COPY --from=web-builder /app/apps/web/dist ./public

ENV NODE_ENV=production
EXPOSE 3000
CMD ["node", "dist/server.js"]
```

API server menyajikan:
- `/api/*` — JSON API
- `/assets/*` — file statis dari build web
- `/*` — SPA fallback (return `index.html`)

---

## Checklist Sebelum Deploy

- [ ] `pnpm build` berhasil tanpa error
- [ ] `pnpm typecheck` lulus
- [ ] `<base href="/">` ada di `index.html`
- [ ] Server dikonfigurasi dengan SPA fallback
- [ ] Environment variable production sudah diset
- [ ] Tidak ada `console.log` yang tertinggal di kode production
- [ ] Tidak ada secret/API key di kode (pakai env variable)
- [ ] Favicon ada
- [ ] Meta tags (title, description) sudah benar

---

## Ringkasan

- `pnpm build` menghasilkan `dist/` dengan file JS dan CSS yang sudah hashed
- SPA fallback wajib di server — `try_files $uri /index.html` (nginx) atau rewrite rules
- Cloudflare Pages dan Vercel paling mudah untuk SPA statis
- Nginx untuk VPS — dengan gzip dan cache immutable untuk assets
- Docker multi-stage untuk deploy terisolasi
- Bundle Aurelia 2: ~50–60 KB gzipped untuk app sederhana
- Manual chunks di Vite untuk pisahkan vendor dari kode aplikasi

---

## Latihan Bab 09b

### Latihan: Checklist Deploy Mandiri

Sebelum deploy proyek Aurelia pertamamu ke production, buat checklist pribadi berdasarkan bab ini. Jawab tiap pertanyaan:

1. Di mana kamu akan deploy? (Cloudflare Pages / Vercel / VPS / Docker) — dan kenapa?
2. Bagaimana kamu konfigurasi SPA fallback di platform pilihan itu?
3. Apa yang kamu set sebagai environment variable (jangan hardcode)?
4. Bagaimana kamu verifikasi bundle size setelah `pnpm build`?

<details>
<summary>Panduan jawaban</summary>

**1. Pilihan platform:**
- Cloudflare Pages — gratis, CDN global, cocok untuk SPA statis tanpa server
- Vercel — gratis tier cukup untuk portfolio, integrasi GitHub mudah
- VPS + Nginx — kalau butuh kontrol penuh atau punya server sendiri
- Docker — kalau mau isolasi dan portabilitas, cocok untuk monorepo SPA+API

**2. SPA fallback:**
- Cloudflare Pages: set "Not found behavior" → `/index.html` di dashboard
- Vercel: tambah `"rewrites": [{"source": "/(.*)", "destination": "/index.html"}]` di `vercel.json`
- Nginx: `try_files $uri /index.html;` di blok `location /`
- Docker+Nginx: sudah ada di contoh Dockerfile bab ini

**3. Environment variables yang umum:**
- `VITE_API_BASE_URL` — URL API kalau deploy terpisah
- `DATABASE_URL` — koneksi database (hanya di API, jangan di frontend)
- `SESSION_SECRET` — secret untuk session cookie
- `STRIPE_SECRET_KEY` — API key Stripe
- Semua yang ada di `.env.example` di repo referensi

**4. Verifikasi bundle size:**
```bash
pnpm build
# Lihat output — Vite menampilkan ukuran tiap chunk
# Gunakan bundlephobia.com untuk estimasi dependency size
# Target: < 100 KB gzipped untuk SPA sederhana
```

</details>

---

## Selanjutnya

Aplikasi sudah bisa deploy. Sekarang kita bahas bagaimana menggunakan semua skill ini untuk mendapat pekerjaan nyata di Upwork.

→ [Bab 10: Aurelia di Dunia Kerja](../bab-10/README.md)
