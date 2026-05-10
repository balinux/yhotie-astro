---
title: "Setup Android SDK di Ubuntu dengan Mudah: Panduan Singkat untuk Developer"
description: Panduan singkat menyiapkan Android SDK di Ubuntu, mulai dari instalasi command line tools, konfigurasi environment variables, hingga pemasangan komponen penting dengan sdkmanager agar sistem siap untuk build dan menjalankan emulator Android versi terbaru.
pubDatetime: 2025-09-25T16:00:00Z
tags:
  - android
  - sdk
  - ubuntu
draft: false
---
## Pendahuluan & istilah

- `sdkmanager` adalah alat baris perintah (command line) untuk melihat, memasang, memperbarui, dan menghapus paket Android SDK. ([Android Developers](https://developer.android.com/tools/sdkmanager?utm_source=chatgpt.com))
- `cmdline-tools` (command line tools) adalah paket yang menyertakan `sdkmanager`, `avdmanager`, dan alat-alat pengelolaan SDK lainnya. ([Android Developers](https://developer.android.com/tools?utm_source=chatgpt.com))
- Struktur direktori yang benar sangat penting agar `sdkmanager` bisa mendeteksi root SDK dan berfungsi tanpa error “Could not determine SDK root”.

---

## 📁 Struktur direktori yang disarankan

Misalkan kamu ingin menyimpan seluruh SDK di:

```
/home/nama_user/Android   ← ini jadi root SDK kamu

```

Maka struktur idealnya:

```
/home/nama_user/Android/
  cmdline-tools/
    latest/
      bin/
      lib/
      source.properties
      … (file & folder lain dari paket cmdline tools)
  platform-tools/
  build-tools/
  platforms/
  system-images/
  emulator/    (opsional jika memakai emulator)
  …

```

Catat:

- Folder `cmdline-tools` harus berada **di dalam** root SDK.
- Di dalam `cmdline-tools`, folder versi (misalnya `latest`) berisi `bin/`, `lib/`, dsb.
- `sdkmanager` harus ada di `…/cmdline-tools/latest/bin/`.

---

## 👣 Langkah setup lengkap

Berikut langkah demi langkah yang bisa kamu tulis di blog:

### 1. Install Java (OpenJDK)

Android SDK memerlukan Java agar dapat menjalankan alat-alatnya.

```bash
sudo apt update
sudo apt install openjdk-11-jdk   # atau versi lain yang kompatibel

```

Cek:

```bash
java -version
javac -version

```

Jika sistem memiliki beberapa versi Java, kamu mungkin perlu menggunakan `update-alternatives` untuk memilih yang benar.

---

### 2. Download paket Command Line Tools dari Google

- Kunjungi halaman resmi Android (Android Studio → _Command line tools only_) dan unduh untuk Linux. ([Android Developers](https://developer.android.com/tools?utm_source=chatgpt.com))
- File akan berupa `zip` (misalnya `commandlinetools-linux-<versi>.zip`).

---

### 3. Ekstrak & atur struktur direktori

Misalnya kamu menyimpan SDK di:

```
~/Android

```

Langkah:

```bash
cd ~
mkdir Android
cd Android

# Misal file zip ada di ~/Downloads
unzip ~/Downloads/commandlinetools-linux-*.zip
# biasanya akan menghasilkan folder bernama “cmdline-tools” atau “tools” tergantung versi

# Siapkan struktur:
mkdir -p cmdline-tools
mv cmdline-tools/* cmdline-tools/latest   # kalau zip menghasilkan folder cmdline-tools
# Atau kalau zip menghasilkan “tools”, sesuaikan:
# mv tools cmdline-tools/latest

# Setelah itu, pastikan isi folder bin ada di:
ls cmdline-tools/latest/bin
# harus ada sdkmanager, avdmanager, dll

```

Jika kamu sebelumnya memiliki struktur yang berbeda (misalnya `latest/cmdline-tools`), kamu bisa memindahkan agar menjadi `cmdline-tools/latest`.

---

### 4. Atur variabel lingkungan (environment variables)

Edit file `~/.bashrc` atau `~/.profile` (tergantung shell yang kamu gunakan) dan tambahkan:

```bash
# root SDK Android
export ANDROID_SDK_ROOT="$HOME/Android"
export ANDROID_HOME="$ANDROID_SDK_ROOT"

# Tambahkan path ke alat-alat SDK agar bisa dipanggil dari mana saja
export PATH="$PATH:$ANDROID_SDK_ROOT/cmdline-tools/latest/bin"
export PATH="$PATH:$ANDROID_SDK_ROOT/platform-tools"
export PATH="$PATH:$ANDROID_SDK_ROOT/emulator"   # opsional, kalau kamu punya folder emulator

```

Setelah disimpan, muat ulang:

```bash
source ~/.bashrc

```

---

### 5. Beri izin (permissions) agar SDK bisa ditulis

Supaya `sdkmanager` dapat mengunduh dan menulis file ke dalam folder SDK, berikan izin tulis:

```bash
chmod -R a+w ~/Android

```

(atau jika SDK diletakkan di lokasi lain, sesuaikan path-nya)

---

### 6. Verifikasi `sdkmanager`

Coba:

```bash
which sdkmanager

```

Harus menunjukkan:

```
/home/…/Android/cmdline-tools/latest/bin/sdkmanager

```

Kemudian:

```bash
sdkmanager --sdk_root="$HOME/Android" --list

```

atau cukup:

```bash
sdkmanager --list

```

Kalau daftar paket muncul tanpa error, berarti setup `cmdline-tools` sudah benar.

---

### 7. Pasang komponen SDK yang dibutuhkan

Misalnya kamu ingin bekerja dengan Android API level 34 dan menjalankan emulator, kamu bisa memasang:

```bash
sdkmanager "platform-tools"
sdkmanager "platforms;android-34"
sdkmanager "build-tools;34.0.0"
sdkmanager "cmdline-tools;latest"
sdkmanager "system-images;android-34;default;x86_64"

```

Catatan:

- `platform-tools`: berisi `adb`, `fastboot`, dll.
- `platforms;android-34`: pustaka & API Android versi 34 agar proyek bisa dikompilasi.
- `build-tools;34.0.0`: alat build seperti `aapt2`, `zipalign`, dll.
- `cmdline-tools;latest`: jika kamu ingin memastikan versi terbaru command line tools.
- `system-images;android-34;default;x86_64`: image emulator agar bisa menjalankan Android 34 di emulator.

Jangan lupa:

```bash
sdkmanager --licenses

```

Terima semua lisensi yang diminta.

---

### 8. (Opsional) Membuat emulator & menjalankannya

Jika kamu ingin:

```bash
avdmanager create avd -n namaAVD -k "system-images;android-34;default;x86_64"
emulator -list-avds
emulator @namaAVD

```

---

## ⚠️ Catatan & tips

- Pastikan tanda kutip yang digunakan adalah versi ASCII (`"`) bukan kutip miring (“ atau ”).
- Jika kamu mendapat error **“Error: Could not determine SDK root”**, itu karena `sdkmanager` tidak bisa menemukan struktur yang diharapkan. Kamu bisa menggunakan flag eksplisit `-sdk_root=/path/ke/SDK` atau memperbaiki struktur folder.
- Versi `cmdline-tools` bisa lebih dari satu; menggunakan folder `latest` memudahkan referensi.
- Jika versi `cmdline-tools` yang ada sudah cukup dan tidak error, kamu tidak perlu menjalankan `sdkmanager "cmdline-tools;latest"` lagi.
- Pastikan `JAVA_HOME` atau versi Java kamu kompatibel, karena `sdkmanager` membutuhkan Java.
- Pada distribusi Linux lain, langkah-langkahnya relatif sama: unzip, struktur direktori, variabel lingkungan, install paket SDK lewat `sdkmanager`.
