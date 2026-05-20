---
title: Install dan Konfigurasi MariaDB di Arch Linux
description: Belajar cara install MariaDB di Arch Linux secara bersih dan aman. Panduan lengkap mulai dari instalasi paket, inisialisasi direktori data, hingga setup security.
pubDatetime: 2026-05-19T16:00:00Z
tags:
  - blog
  - linux
  - mariadb
draft: false
---
## Panduan Lengkap Instalasi dan Konfigurasi MariaDB di Arch Linux

MariaDB adalah salah satu sistem manajemen database relasional (_Relational Database Management System_ / RDBMS) paling populer di dunia. Di ekosistem Arch Linux, MariaDB diadopsi sebagai pengganti resmi dan bawaan (_drop-in replacement_) untuk MySQL karena performanya yang dinilai lebih cepat dan sepenuhnya bersifat open-source.

Artikel ini akan membahas langkah demi langkah memasang, mengonfigurasi, hingga mengamankan MariaDB di sistem Arch Linux Anda secara bersih dan tepat.

---

## Prasyarat Sebelum Instalasi

Sebelum memulai, pastikan sistem Arch Linux Anda berada dalam kondisi terperbarui untuk menghindari konflik dependensi pustaka (_library cache_).

Buka terminal Anda dan jalankan perintah sinkronisasi sistem berikut:

```bash
sudo pacman -Syu
```

---

## Langkah Demi Langkah Instalasi MariaDB

Berbeda dengan beberapa distribusi Linux lain (seperti Ubuntu atau Debian) yang otomatis melakukan konfigurasi awal saat instalasi, Arch Linux menerapkan prinsip _KISS (Keep It Simple, Stupid)_. Artinya, Anda harus melakukan inisialisasi database secara manual.

## Langkah 1: Memasang Paket MariaDB

Gunakan manajer paket `pacman` untuk mengunduh dan memasang paket utama MariaDB langsung dari repositori resmi Arch:

```bash
sudo pacman -S mariadb
```

## Langkah 2: Inisialisasi Direktori Data (Wajib)

Sebelum menyalakan layanan, Anda wajib membuat struktur direktori database dan tabel sistem bawaan terlebih dahulu. Jika langkah ini dilewati, layanan MariaDB dipastikan akan gagal berjalan (_crash/error_).

Eksekusi perintah inisialisasi berikut:

```bash
sudo mariadb-install-db --user=mysql --basedir=/usr --datadir=/var/lib/mysql
```

- `--user=mysql`: Menetapkan kepemilikan berkas database kepada pengguna sistem `mysql`.
- `--datadir=/var/lib/mysql`: Lokasi fisik tempat seluruh data tabel Anda akan disimpan di dalam penyimpanan.

## Langkah 3: Mengaktifkan dan Menjalankan Layanan MariaDB

Sekarang, mari kita jalankan sistem database menggunakan `systemd`. Perintah di bawah ini akan langsung menyalakan database sekaligus mengaturnya agar otomatis aktif setiap kali komputer/server dinyalakan (_booting_):

```bash
sudo systemctl enable --now mariadb
```

Untuk memastikan layanan telah berjalan dengan status hijau (_running_), Anda dapat memeriksanya melalui perintah:

```bash
sudo systemctl status mariadb
```

## Langkah 4: Mengamankan Instalasi Awal

Secara bawaan, instalasi baru MariaDB belum memiliki kata sandi untuk akun administrator (`root`) dan memiliki beberapa konfigurasi yang kurang aman untuk produksi.

Jalankan skrip keamanan interaktif bawaan berikut:

```bash
sudo mariadb-secure-installation
```

Anda akan dipandu melalui beberapa pertanyaan keamanan di layar terminal:

1. Enter current password for root (enter for none): Tekan _Enter_ saja karena awalnya belum ada password.
2. Switch to unix_socket authentication [Y/n]: Ketik `n` (Disarankan agar Anda tetap bisa login menggunakan password biasa di berbagai aplikasi client).
3. Change the root password? [Y/n]: Ketik `Y` lalu masukkan kata sandi baru yang kuat untuk root database Anda.
4. Remove anonymous users? [Y/n]: Ketik `Y` untuk menghapus akses pengguna tanpa nama demi keamanan.
5. Disallow root login remotely? [Y/n]: Ketik `Y` jika Anda hanya ingin mengakses database dari komputer ini saja (_localhost_).
6. Remove test database and access to it? [Y/n]: Ketik `Y` untuk menghapus database uji bawaan.
7. Reload privilege tables now? [Y/n]: Ketik `Y` agar semua perubahan di atas langsung diterapkan.

---

## Mengakses Console MariaDB

Setelah seluruh proses konfigurasi selesai, Anda sekarang dapat masuk ke lingkungan command-line MariaDB dengan mengetik:

```bash
mariadb -u root -p
```

Masukkan password root yang telah Anda buat pada Langkah 4.

> 💡 Tips Blog: Perintah lama seperti `mysql -u root -p` masih didukung sebagai alias, namun komunitas Arch Linux sangat menyarankan pembaca mulai beralih menggunakan perintah baru `mariadb` karena alias lama tersebut direncanakan akan dihapus di masa mendatang.

---

## Manajemen Perintah Dasar Systemd

Sebagai pemilik server atau pengembang, berikut adalah beberapa perintah penting berbasis `systemctl` yang sering digunakan untuk mengelola layanan MariaDB Anda:

- Menghentikan database: `sudo systemctl stop mariadb`
- Menyalakan kembali database: `sudo systemctl start mariadb`
- Memuat ulang konfigurasi setelah modifikasi file: `sudo systemctl restart mariadb`

---

## Catatan Tambahan Bagi Pengguna Lanjutan (Optional)

Jika pembaca blog Anda nantinya berniat menggunakan MariaDB di dalam LXC Container atau membutuhkan akses aplikasi grafis dari luar (seperti DBeaver atau DataGrip), mereka perlu mengubah konfigurasi _Bind Address_.

Secara bawaan, MariaDB mengunci jaringan pada IP lokal `127.0.0.1`. Agar database dapat diakses dari jaringan luar, buka file konfigurasi berikut:

```bash
sudo nano /etc/my.cnf.d/server.cnf
```

Cari blok `[mysqld]` lalu ubah baris penunjuk alamat jaringan menjadi:

```ini
[mysqld]
bind-address = 0.0.0.0
```

Simpan file tersebut dan muat ulang layanan dengan perintah `sudo systemctl restart mariadb`.

---

## Kesimpulan

Proses instalasi MariaDB di Arch Linux tergolong sangat bersih dan memberikan kendali penuh kepada penggunanya. Dengan mengikuti panduan ini, database Anda kini siap digunakan untuk mendukung berbagai kebutuhan aplikasi lokal, web server (LAMP/LEMP Stack), maupun keperluan pengembangan perangkat lunak lainnya.

Selamat mencoba, dan jangan ragu untuk meninggalkan pertanyaan di kolom komentar jika Anda menemui kendala!
