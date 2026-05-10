---
title: Restart Service Podman dengan Systemd Timer
description: Dokumentasi ini menjelaskan cara menjadwalkan restart service Podman secara otomatis menggunakan systemd timer. Dengan konfigurasi ini, Podman akan direstart pada waktu yang telah ditentukan.
pubDatetime: 2026-02-04T16:00:00Z
tags:
  - podman
  - nodejs
  - systemd
draft: false
---
# **Pendahuluan**

Dokumentasi ini menjelaskan cara menjadwalkan restart service Podman secara otomatis menggunakan **systemd timer**. Dengan konfigurasi ini, Podman akan direstart pada waktu yang telah ditentukan.

---

## **1. Membuat Systemd Timer**

Systemd timer digunakan untuk menjadwalkan restart service Podman. Untuk pengguna rootless Podman, file konfigurasi diletakkan di **`~/.config/systemd/user/`**.

### **1.1 Buat File Timer**

Buka terminal dan buat file berikut:

```
nano ~/.config/systemd/user/restart-podman-container.timer

```

Isi file:

```
[Unit]
Description=Restart Podman container at a specific time
Requires=restart-podman-container.service

[Timer]
OnCalendar=daily 02:00:00
Persistent=true

[Install]
WantedBy=timers.target

```

**Penjelasan:**

- `OnCalendar=daily 02:00:00` → Timer akan berjalan setiap hari pada pukul **02:00**.
- `Persistent=true` → Jika timer terlewat karena sistem mati, maka akan langsung dijalankan saat sistem hidup kembali.

---

## **2. Membuat Systemd Service**

Systemd service digunakan untuk menjalankan perintah restart Podman.

### **2.1 Buat File Service**

```
nano ~/.config/systemd/user/restart-podman-container.service

```

Isi file:

```
[Unit]
Description=Restart Podman Container
After=podman.service

[Service]
Type=oneshot
ExecStart=/bin/systemctl --user restart podman.service

```

**Penjelasan:**

- `Type=oneshot` → Service hanya berjalan satu kali.
- `ExecStart=/bin/systemctl --user restart podman.service` → Perintah untuk merestart service Podman.

---

## **3. Mengaktifkan Systemd Timer**

Setelah membuat file timer dan service, jalankan perintah berikut:

```
systemctl --user daemon-reload
systemctl --user enable --now restart-podman-container.timer

```

**Penjelasan:**

- `daemon-reload` → Memuat ulang konfigurasi systemd.
- `enable --now` → Mengaktifkan dan menjalankan timer secara langsung.

---

## **4. Memeriksa Status Timer**

Untuk memastikan timer berjalan dengan benar:

```
systemctl --user list-timers --all

```

Atau cek status spesifik:

```
systemctl --user status restart-podman-container.timer

```

Jika terjadi error, periksa log dengan:

```
journalctl --user -xe

```

---

## **5. Menonaktifkan Systemd Timer**

Jika ingin menonaktifkan timer:

```
systemctl --user stop restart-podman-container.timer
systemctl --user disable restart-podman-container.timer

```

---

## **Kesimpulan**

Dengan konfigurasi ini, service Podman akan otomatis direstart setiap hari pada pukul **02:00** tanpa perlu intervensi manual. Systemd timer adalah solusi ringan dan efisien untuk menangani tugas otomatisasi di Linux.

---

### **Referensi**

- `man systemd.timer`
- `man systemd.service`