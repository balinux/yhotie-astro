---
title: Setup Google Analytics Astro + Partytown
description: Pelajari cara setup Google Analytics di Astro menggunakan Partytown agar website lebih cepat, SEO-friendly, dan optimal untuk Core Web Vitals.
pubDatetime: 2026-05-06T16:00:00Z
tags:
  - astrojs
  - SEO
draft: false
---
# Setup Google Analytics dengan Partytown di Astro

Saat membuat website menggunakan Astro, performa biasanya menjadi prioritas utama. Astro dikenal ringan, cepat, dan sangat SEO-friendly. Namun ada satu hal yang sering membuat skor Lighthouse turun tanpa disadari:

> script pihak ketiga seperti Google Analytics.

Banyak developer langsung memasang Google Analytics menggunakan script biasa. Padahal script analytics dapat membebani main thread browser dan memperlambat rendering halaman.

Solusinya adalah menggunakan Partytown.

Di artikel ini kamu akan belajar:

- apa itu Partytown
    
- kenapa penting untuk Astro
    
- cara setup Google Analytics
    
- beberapa metode implementasi Partytown
    
- best practice SEO untuk website Astro modern
    

Artikel ini cocok untuk:

- pemula
    
- mahasiswa
    
- frontend developer
    
- backend developer
    
- pengguna Astro
    

---

# Apa Itu Partytown?

[Partytown](https://partytown.builder.io/?utm_source=chatgpt.com) adalah library dari Builder.io yang memungkinkan script pihak ketiga berjalan di Web Worker.

Biasanya script seperti:

- Google Analytics
    
- Google Tag Manager
    
- Facebook Pixel
    

akan berjalan di browser utama (main thread).

Akibatnya:

- UI bisa terasa lebih berat
    
- interaksi melambat
    
- Core Web Vitals turun
    

Partytown memindahkan proses tersebut ke worker thread sehingga website tetap cepat.

---

# Kenapa Astro Cocok dengan Partytown?

[Astro](https://astro.build/?utm_source=chatgpt.com) memiliki filosofi:

> “Ship less JavaScript”

Astro sangat fokus pada:

- performa
    
- SEO
    
- static rendering
    
- optimasi JavaScript
    

Karena itu, menggabungkan Astro dengan Partytown menjadi kombinasi yang sangat powerful.

Cocok untuk:

- blog
    
- portfolio
    
- company profile
    
- landing page
    
- memorial website
    
- dokumentasi
    

---

# Cara Install Partytown di Astro

Gunakan command berikut:

```bash
pnpm astro add partytown
```

Command ini otomatis:

- install package
    
- update konfigurasi Astro
    
- setup integrasi Partytown
    

---

# Konfigurasi Astro

Buka file:

```bash
astro.config.mjs
```

Lalu ubah menjadi:

```js
import { defineConfig } from 'astro/config';
import partytown from '@astrojs/partytown';

export default defineConfig({
  integrations: [
    partytown({
      config: {
        forward: ['dataLayer.push'],
      },
    }),
  ],
});
```

---

# Kenapa `forward: ['dataLayer.push']` Penting?

Google Analytics menggunakan:

```js
dataLayer.push()
```

untuk tracking event.

Tanpa konfigurasi ini:

- analytics bisa tidak berjalan
    
- page tracking gagal
    
- event tidak terkirim
    

---

# Cara 1 — Menggunakan Component GoogleAnalytics

Ini adalah metode yang paling umum digunakan.

## Membuat Component

Buat file:

```bash
src/components/GoogleAnalytics.astro
```

Isi:

```astro
---
const GA_ID = import.meta.env.PUBLIC_GA_ID;
---

<script
  type="text/partytown"
  async
  src={`https://www.googletagmanager.com/gtag/js?id=${GA_ID}`}
></script>

<script type="text/partytown">
  window.dataLayer = window.dataLayer || [];

  window.gtag = function () {
    dataLayer.push(arguments);
  };

  gtag('js', new Date());

  gtag('config', '{GA_ID}', {
    page_path: window.location.pathname,
  });
</script>
```

---

## Tambahkan ke Layout

Contoh:

```astro
---
import GoogleAnalytics from '@/components/GoogleAnalytics.astro';
---

<html>
  <head>
    <GoogleAnalytics />
  </head>

  <body>
    <slot />
  </body>
</html>
```

---

# Cara 2 — Menggunakan BaseHead Langsung

Metode kedua adalah memasukkan script analytics langsung ke file `BaseHead`.

Pendekatan ini cocok jika:

- ingin semua meta SEO dalam satu file
    
- ingin setup lebih sederhana
    
- menggunakan BaseHead global
    

---

## Contoh Implementasi

```astro
<!-- Google Analytics (GA4) via Partytown -->
<script
  type="text/partytown"
  src={`https://www.googletagmanager.com/gtag/js?id=${GA_ID}`}
  async
></script>

<script
  type="text/partytown"
  set:html={`window.dataLayer = window.dataLayer || [];
function gtag(){dataLayer.push(arguments);}
gtag('js', new Date());
gtag('config', '${GA_ID}');`}
></script>
```

---

# Penjelasan Metode Kedua

Pada implementasi ini:

- script analytics langsung dimasukkan ke `<head>`
    
- menggunakan `set:html`
    
- tidak perlu component tambahan
    

Ini sangat cocok untuk:

- blog sederhana
    
- personal website
    
- static site
    
- memorial website
    

---

# Kelebihan Cara Kedua

## Struktur Lebih Ringkas

Semua:

- SEO meta tag
    
- Open Graph
    
- Twitter card
    
- Google Analytics
    

berada dalam satu file.

---

## Lebih Mudah Dikelola

Tidak perlu:

- import component tambahan
    
- setup file terpisah
    

---

## Cocok untuk Static Website

Terutama untuk:

- Astro static output
    
- blog
    
- portfolio
    

---

# Cara 3 — Manual GA4 Integration dengan Astro ClientRouter

Selain menggunakan component terpisah atau langsung memasukkan script ke `BaseHead`, ada juga pendekatan yang lebih advanced dan production-ready.

Metode ini biasanya digunakan pada:

- blog modern
    
- dokumentasi developer
    
- website dengan View Transition
    
- Astro SPA navigation
    
- website dengan analytics tracking yang lebih kompleks
    

Pendekatan ini disebut:

> Manual GA4 Integration dengan Astro ClientRouter SPA Tracking

---

# Kenapa Metode Ini Berbeda?

Pada implementasi sebelumnya, Google Analytics hanya berjalan pada initial page load.

Namun saat menggunakan:

```astro
<ClientRouter />
```

Astro bekerja seperti SPA (Single Page Application).

Artinya:

- halaman berpindah tanpa full reload
    
- browser tidak refresh total
    
- Google Analytics tidak otomatis mendeteksi perpindahan halaman
    

Karena itu dibutuhkan:

> manual page view tracking

---

# Contoh Implementasi Production Ready

Berikut contoh implementasi modern yang mendukung:

- Partytown
    
- SPA tracking
    
- Astro transitions
    
- manual page view
    
- environment variable
    

```astro
{
  PUBLIC_GA_MEASUREMENT_ID && (
    <>
      <script
        type="text/partytown"
        async
        src={`https://www.googletagmanager.com/gtag/js?id=${PUBLIC_GA_MEASUREMENT_ID}`}
      ></script>

      <script
        type="text/partytown"
        define:vars={{ gaId: PUBLIC_GA_MEASUREMENT_ID }}
      >
        {`
          window.dataLayer = window.dataLayer || [];

          function gtag() {
            window.dataLayer.push(arguments);
          }

          window.gtag = window.gtag || gtag;

          gtag("js", new Date());

          gtag("config", gaId, {
            send_page_view: true,
          });

          let lastTrackedPath =
            window.location.pathname + window.location.search;

          document.addEventListener("astro:page-load", () => {
            const currentPath =
              window.location.pathname + window.location.search;

            if (currentPath === lastTrackedPath) return;

            lastTrackedPath = currentPath;

            gtag("event", "page_view", {
              page_path: currentPath,
              page_title: document.title,
              page_location: window.location.href,
            });
          });
        `}
      </script>
    </>
  )
}
```

---

# Apa yang Dilakukan Kode Ini?

Implementasi ini melakukan beberapa hal penting.

---

## 1. Menjalankan Google Analytics di Web Worker

Bagian:

```astro
type="text/partytown"
```

menandakan script dijalankan melalui:

- Partytown
    
- Web Worker
    
- bukan main thread browser
    

Hasilnya:

- website lebih ringan
    
- interaksi lebih cepat
    
- Core Web Vitals lebih baik
    

---

## 2. Mendukung SPA Navigation Astro

Bagian:

```js
document.addEventListener("astro:page-load", () => {
```

digunakan untuk mendeteksi:

- perpindahan halaman
    
- route transition
    
- navigation tanpa reload
    

Tanpa ini:  
Google Analytics hanya membaca halaman pertama saja.

---

## 3. Menghindari Duplicate Tracking

Bagian:

```js
if (currentPath === lastTrackedPath) return;
```

digunakan agar:

- page view tidak tercatat dua kali
    
- analytics tetap akurat
    

---

# Perbandingan Ketiga Metode

|Metode|Tingkat|Cocok Untuk|
|---|---|---|
|Component GoogleAnalytics|Mudah|Pemula|
|BaseHead Inline Script|Menengah|Blog sederhana|
|ClientRouter + SPA Tracking|Advanced|Production modern website|

---

# Cara Mengecek Apakah Analytics Berhasil

Buka:

- Chrome DevTools
    
- tab Network
    

Cari request:

- `googletagmanager`
    
- `collect?v=2`
    

Kalau muncul:  
✅ Google Analytics aktif.

---

# Cara Memastikan Partytown Berjalan

Saat production build dijalankan, akan muncul folder:

```bash
/~partytown/
```

Ini menandakan script dijalankan melalui Web Worker.

---

# Tips SEO Tambahan untuk Astro

## Gunakan Absolute URL untuk Gambar

Contoh yang benar:

```html
<meta property="og:image" content="https://domain.com/image.png" />
```

Jangan gunakan:

```html
../../image.png
```

---

## Gunakan Ukuran OG Image Ideal

Rekomendasi:

- 1200x630
    
- format WebP
    
- ukuran kecil tapi berkualitas
    

---

## Gunakan Meta Description yang Natural

Hindari keyword stuffing.

Gunakan deskripsi yang:

- jelas
    
- menarik
    
- mengundang klik
    

---

# Kelebihan Menggunakan Partytown

Berikut manfaat utama:

- website lebih cepat
    
- main thread lebih ringan
    
- SEO meningkat
    
- Lighthouse score lebih baik
    
- analytics tetap berjalan
    
- user experience lebih nyaman
    

---

# Kekurangan Partytown

Walaupun sangat bagus, ada beberapa hal yang perlu diperhatikan:

- tidak semua script compatible
    
- debugging sedikit berbeda
    
- kadang perlu konfigurasi tambahan
    

Namun untuk Google Analytics, Partytown sudah sangat stabil digunakan.

---

# FAQ

## Apakah Partytown cocok untuk pemula?

Ya. Astro sudah menyediakan integrasi resmi sehingga setup menjadi jauh lebih mudah.

---

## Apakah Google Analytics tetap akurat?

Ya. Selama `forward: ['dataLayer.push']` digunakan, tracking tetap normal.

---

## Mana yang lebih baik, component atau langsung di BaseHead?

Keduanya bagus.

Gunakan:

- component jika ingin reusable
    
- BaseHead jika ingin lebih sederhana
    

---

## Kapan harus menggunakan ClientRouter tracking?

Gunakan jika website kamu menggunakan:

- Astro transitions
    
- SPA navigation
    
- dynamic page transitions
    

---

# Penutup

Menggunakan Google Analytics secara biasa memang mudah, tetapi bisa berdampak pada performa website.

Dengan Partytown, kamu bisa:

- tetap menggunakan analytics
    
- menjaga website tetap cepat
    
- meningkatkan pengalaman pengguna
    
- mempertahankan SEO score
    

Astro dan Partytown adalah kombinasi modern yang sangat cocok untuk website cepat dan SEO-friendly.

Kalau kamu sedang membangun blog, portfolio, atau memorial website menggunakan Astro, implementasi ini sangat direkomendasikan.

---

# Internal Linking Suggestion

Tambahkan internal link ke artikel berikut:

- Cara Setup SEO Meta Tag di Astro
    
- Cara Membuat Open Graph di Astro
    
- Cara Deploy Astro ke Vercel
    
- Optimasi Lighthouse Score Astro
    
- Cara Membuat Sitemap Otomatis di Astro
    

