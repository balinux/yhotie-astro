---
title: auto startup container podman
description: tutorial untuk menjalankan container Podman dengan systemd, sehingga container dapat berjalan otomatis setelah server restart
pubDatetime: 2026-01-28T16:00:00Z
tags:
  - nodejs
  - podman
  - systemd
draft: false
---
Berikut adalah langkah-langkah untuk menjalankan **container Podman dengan systemd**, sehingga container dapat berjalan otomatis setelah server restart.

---

### **1. Buat Container dengan Nama Unik**

Pastikan Anda sudah memiliki container Podman yang dibuat dengan nama unik. Jika belum, buat container dengan nama (misalnya `nodejs-container`):

```bash
podman run -d --name nodejs-container -p 3000:3000 nodejs-app:alpine

```

---

### **2. Generate File Unit systemd**

Podman memiliki fitur untuk menghasilkan file unit systemd. Gunakan perintah berikut:

```bash
podman generate systemd --name nodejs-container --files --new

```

- **`-name nodejs-container`**: Nama container Anda.
- **`-files`**: Membuat file unit systemd di direktori kerja Anda.
- **`-new`**: Restart container sebagai instance baru jika container mati.

Perintah ini akan menghasilkan file `container-nodejs-container.service`.

---

### **3. Pindahkan File Unit ke systemd**

Pindahkan file unit yang telah dibuat ke direktori systemd (`~/.config/systemd/user/` untuk rootless, atau `/etc/systemd/system/` untuk root):

### Jika menggunakan rootless:

```bash
mkdir -p ~/.config/systemd/user
mv container-nodejs-container.service ~/.config/systemd/user/

```

### Jika menggunakan root:

```bash
sudo mv container-nodejs-container.service /etc/systemd/system/

```

---

### **4. Reload systemd Daemon**

Setelah file unit dipindahkan, reload daemon systemd agar mengenali file baru:

### Jika rootless:

```bash
systemctl --user daemon-reload

```

### Jika root:

```bash
sudo systemctl daemon-reload

```

---

### **5. Enable dan Start Service**

Aktifkan service agar berjalan otomatis saat server restart, lalu jalankan service:

### Jika rootless:

```bash
systemctl --user enable container-nodejs-container
systemctl --user start container-nodejs-container

```

### Jika root:

```bash
sudo systemctl enable container-nodejs-container
sudo systemctl start container-nodejs-container

```

---

### **6. Verifikasi**

Cek status service untuk memastikan berjalan dengan baik:

### Jika rootless:

```bash
systemctl --user status container-nodejs-container

```

### Jika root:

```bash
sudo systemctl status container-nodejs-container

```

Output akan menunjukkan service aktif dan container berjalan.

---

### **7. Test Restart Server**

Untuk memastikan container berjalan otomatis setelah restart, lakukan reboot:

```bash
sudo reboot

```

Setelah restart, periksa kembali status container:

### Jika rootless:

```bash
systemctl --user status container-nodejs-container

```

### Jika root:

```bash
sudo systemctl status container-nodejs-container

```

---

### **Catatan**

1. **Rootless Podman**: Gunakan `-user` untuk rootless setup.
2. **Log Service**: Jika ada masalah, cek log service:

atau

````
```bash
journalctl --user -u container-nodejs-container

```

```bash
sudo journalctl -u container-nodejs-container

```
````

Dengan setup ini, container Anda akan otomatis berjalan saat server menyala ulang. 🚀

Podman memiliki kemampuan untuk mengintegrasikan container dengan **systemd** berkat desain arsitekturnya yang unik. Berikut penjelasan mengapa Podman bisa melakukan ini dan apa yang membuatnya berbeda:

---

### **1. Podman Tidak Membutuhkan Daemon**

- Podman **tidak menggunakan daemon** seperti Docker yang memiliki proses `dockerd`.
- Setiap container yang dijalankan oleh Podman adalah **proses Linux independen** yang dikelola langsung oleh kernel.
- Karena itu, Podman dapat memanfaatkan **systemd** secara langsung untuk mengelola proses-proses container.

**Perbedaannya:**

- **Docker** membutuhkan daemon (background process) untuk mengelola container.
- **Podman** menjalankan container tanpa daemon, sehingga lebih sesuai untuk integrasi langsung dengan sistem init seperti `systemd`.

---

### **2. Dukungan bawaan untuk Systemd**

Podman memiliki fitur **`podman generate systemd`**, yang mempermudah pembuatan file unit systemd dari container. Fitur ini bekerja dengan:

- Membaca konfigurasi container (seperti port, volume, environment).
- Menghasilkan file unit systemd yang dapat langsung digunakan untuk mengelola container.

Dengan ini, Podman memungkinkan:

- **Restart otomatis container saat server reboot**.
- Manajemen container menggunakan perintah `systemctl`.

---

### **3. Rootless Containers**

Podman dirancang untuk mendukung **rootless mode** secara bawaan. Artinya, pengguna biasa (non-root) dapat menjalankan container tanpa perlu hak istimewa (privilege).

Integrasi dengan systemd di mode rootless dilakukan dengan:

- Menyimpan file unit systemd di direktori pengguna: `~/.config/systemd/user/`.
- Menjalankan systemd dengan opsi `-user`.

---

### **4. Tidak Bergantung pada Proses Utama**

- Podman tidak memerlukan proses utama (`dockerd`) untuk mengelola container.
- Systemd dapat mengontrol container secara langsung karena container Podman adalah **proses biasa** di sistem Linux.

Ini berbeda dari Docker, di mana systemd harus mengontrol `dockerd` sebagai perantara untuk mengelola container.

---

### **5. Keamanan Lebih Baik**

Karena Podman mendukung rootless mode secara penuh:

- Container berjalan sebagai proses milik pengguna biasa.
- Mengurangi risiko eskalasi privilege jika ada kerentanan di dalam container.

Selain itu, setiap integrasi dengan systemd tetap memanfaatkan prinsip **least privilege**.

---

### **6. Fitur Tambahan dengan Podman Pods**

Podman juga memiliki konsep **pods** (mirip dengan Kubernetes Pods). File unit systemd bisa dibuat untuk mengelola **seluruh pod**, sehingga:

- Banyak container dalam satu pod bisa dikelola sebagai satu unit systemd.

---

### **Ringkasan: Mengapa Podman Mendukung Systemd dengan Baik?**

1. **Tanpa daemon**: Container langsung dikelola sebagai proses Linux.
2. **Integrasi bawaan**: `podman generate systemd` otomatis membuat file unit systemd.
3. **Rootless mode**: Container aman berjalan tanpa hak istimewa.
4. **Keamanan lebih baik**: Tidak ada daemon, sehingga lebih sulit dieksploitasi.
5. **Pods**: Kemampuan mengelola grup container sekaligus dengan systemd.

Dengan semua keunggulan ini, Podman menjadi pilihan yang ideal untuk sistem yang membutuhkan integrasi erat dengan `systemd`, terutama dalam lingkungan server yang stabil dan aman. 🚀