---
title: "Praktikum: Implementasi Dasar Edge Computing"
description: "Modul praktikum untuk mempelajari arsitektur, deployment, dan evaluasi solusi Edge Computing melalui latihan terstruktur."
pubDatetime: 2026-04-26T06:48:56+08:00
tags:
  - edge-computing
  - praktikum
  - iot
  - lab
draft: false
---

Level: Menengah
Durasi: 2–4 jam
Prasyarat: Dasar jaringan, Docker/Container, Python atau Node.js, akses ke VM atau single-board computer (Raspberry Pi atau setara).

## Tujuan pembelajaran

Setelah menyelesaikan modul praktikum ini peserta diharapkan dapat:

1. Menjelaskan arsitektur lapisan Edge vs Cloud.
2. Mendeploy sebuah service sederhana di node edge (container).
3. Mengimplementasikan pipeline data ringan: akuisisi → pre-processing → inferensi sederhana.
4. Mengamankan komunikasi antara device dan edge gateway.
5. Mengevaluasi metrik latensi dan bandwidth serta membuat rekomendasi optimasi.

## Bahan dan alat

- 1 VM/PC sebagai cloud (atau akun cloud)
- 1 perangkat edge (Raspberry Pi atau VM yang bertindak sebagai gateway)
- Sensor virtual atau dataset (disediakan)
- Docker / Podman, dan docker-compose atau k3s (opsional)
- Python 3.9+ atau Node.js 18+
- Akses SSH antara mesin

## Ringkasan teori singkat

Ringkas konsep edge: pengertian, manfaat utama (latensi, bandwidth, privasi), serta trade-offs. Gambarkan arsitektur sederhana: Device → Edge Gateway → Cloud. Bahas keamanan dasar dan observability.

## Langkah praktikum

### Persiapan lingkungan (30 menit)

1. Siapkan dua mesin (cloud dan edge) dan pastikan dapat saling SSH.
2. Install Docker dan docker-compose pada kedua mesin.
3. Clone repository contoh (link diberikan oleh instruktur) yang berisi simulasi sensor dan service inferensi ringan.

### Langkah 1 — Menjalankan sensor simulasi (15 menit)

- Jalankan container sensor yang mem-publish data telemetri (mis. suhu) ke MQTT broker lokal di edge node.

Contoh (di edge node):

```bash
# jalankan mqtt broker (eclipse-mosquitto)
docker run -d --name mosquitto -p 1883:1883 eclipse-mosquitto
# jalankan sensor simulator
docker run -d --name sensor-sim --env MQTT_HOST=localhost example/sensor-sim
```

### Langkah 2 — Deploy pre-processing service (30 menit)

- Buat container yang subscribe ke topik MQTT, melakukan cleaning dan aggregasi, lalu menyimpan hasil ke storage lokal (file atau InfluxDB ringan).

Tugas:
- Siapkan Dockerfile sederhana untuk service preproc.
- Jalankan dan verifikasi data yang masuk tersimpan dan teragregasi.

### Langkah 3 — Inferensi ringan di edge (30 menit)

- Gunakan model sederhana (regresi atau threshold) untuk mendeteksi anomali. Model bisa berupa skrip Python yang membaca data teragregasi.
- Deploy sebagai container dan buat endpoint HTTP lokal yang mengembalikan status.

### Langkah 4 — Sinkronisasi ke Cloud & Analitik (30 menit)

- Konfigurasi service untuk mengirim ringkasan per 5 menit ke cloud (HTTP POST ke endpoint di mesin cloud).
- Di cloud, simpan ke database dan buat dashboard metrik sederhana (Grafana/Influx atau Prometheus/Grafana).

### Langkah 5 — Pengukuran dan evaluasi (15–30 menit)

- Ukur latency end-to-end (sensor → preproc → inferensi → respon).
- Ukur bandwidth yang digunakan terutama saat mengirim data mentah vs ringkasan.
- Catat temuan dan optimasi yang mungkin (compress, sampling, model quantization).

## Tugas akhir (penilaian)

1. Submit repository fork dengan Dockerfile dan instruksi deploy (README).
2. Jawab kuis singkat:
   - Jelaskan 3 trade-off utama Edge vs Cloud.
   - Sebutkan 4 teknik mitigasi keamanan di lingkungan edge.
3. Rekam video singkat (5 menit) yang menunjukkan alur kerja dan hasil pengukuran.

## Rubrik penilaian

- Setup dan dokumentasi: 30%
- Fungsi pipeline (sensor → preproc → inferensi → sink): 40%
- Keamanan dan konfigurasi: 10%
- Analisis metrik & rekomendasi: 20%

## Pertanyaan dan tantangan lanjutan

- Bagaimana menangani offline-first ketika koneksi sering terputus?
- Bagaimana mengelola update model aman di ratusan node edge?
- Optimalkan penggunaan energi pada perangkat berbasis baterai.

## Referensi dan sumber

- Materi ringkasan praktik edge (dokumen sumber)
- Contoh container: eclipse-mosquitto, example/sensor-sim
- Tools: Docker, k3s, Grafana, InfluxDB

---

Catatan: modul dapat dipecah menjadi beberapa sesi lab jika diperlukan. Jika ingin konversi ke MDX (menambahkan worksheet dan link file), beri tahu instruksi tambahan.