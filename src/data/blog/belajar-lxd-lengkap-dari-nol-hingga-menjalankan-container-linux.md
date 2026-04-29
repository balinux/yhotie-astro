---
title: Belajar LXD dari Nol sampai Bisa Menjalankan Server di Dalam Laptop
description: LXD adalah platform modern untuk menjalankan container Linux yang terasa seperti mesin virtual, tetapi jauh lebih ringan dan cepat. Dibangun di atas LXC, LXD memungkinkan pengguna menjalankan berbagai sistem operasi seperti Debian, Ubuntu, atau Alpine dalam satu host dengan efisiensi tinggi.
pubDatetime: 2026-04-28T16:00:00Z
tags:
  - lxc
  - containers
  - lxd
draft: false
---
Kalau kamu pernah merasa:

- ingin coba aplikasi di distro lain tanpa install ulang OS
    
- ingin punya banyak “server” di satu laptop
    
- atau ingin environment dev yang bersih dan terisolasi
    

Maka kamu perlu kenal **LXD**.

Artikel ini akan membimbing kamu dari:  
👉 konsep dasar  
👉 install  
👉 sampai praktik nyata

Tanpa ribet.

---

# 🧠 Apa itu LXD?

**LXD** adalah platform untuk menjalankan dan mengelola container Linux yang terasa seperti mesin virtual (VM), tapi jauh lebih ringan.

Secara sederhana:

- Docker → container aplikasi
    
- LXD → container **sistem (full OS)**
    

LXD dibangun di atas LXC dan menyediakan API + CLI yang mempermudah pengelolaan container. ([TechTarget](https://www.techtarget.com/searchitoperations/definition/LXD-Linux-container-hypervisor?utm_source=chatgpt.com "What is LXD (Linux container hypervisor)? | Definition from TechTarget"))

👉 Artinya:  
Kamu bisa menjalankan **Debian, Ubuntu, Alpine, dll** di dalam satu OS.

---

# ⚡ Kenapa LXD Menarik?

Beberapa keunggulan utama:

- 🚀 Start dalam hitungan detik
    
- 💾 Hemat resource (lebih ringan dari VM)
    
- 🧠 Bisa jalan seperti OS penuh
    
- 📦 Support banyak distro
    
- 🔁 Snapshot & clone instan
    

LXD bahkan bisa menjalankan ribuan container dalam satu sistem. ([TechTarget](https://www.techtarget.com/searchitoperations/definition/LXD-Linux-container-hypervisor?utm_source=chatgpt.com "What is LXD (Linux container hypervisor)? | Definition from TechTarget"))

---

# 🧱 Konsep Penting di LXD

Sebelum lanjut, pahami ini:

### 1. Container = OS mini

Berbeda dengan Docker, container di LXD:

- punya filesystem sendiri
    
- bisa install banyak aplikasi
    
- bisa jalan seperti server
    

---

### 2. Image

Template OS:

- `debian/12`
    
- `alpine/3.18`
    
- `ubuntu/22.04`
    

---

### 3. Storage Pool

Tempat semua container disimpan:

```
/var/lib/lxd/storage-pools/
```

---

### 4. Network Bridge

Container dapat IP sendiri (seperti VM kecil)

---

# ⚙️ Install LXD (Arch / CachyOS)

```bash
sudo pacman -S lxd
sudo systemctl enable --now lxd
```

---

# 🚀 Setup Awal

```bash
lxd init
```

Jawaban aman:

- storage → `dir`
    
- network → `yes`
    
- IPv4 → `auto`
    
- IPv6 → `auto`
    

---

# 🧪 Membuat Container Pertama

## 🔹 Debian

```bash
lxc launch images:debian/12 debian-test
```

## 🔹 Alpine (super ringan)

```bash
lxc launch images:alpine/3.18 alpine-test
```

---

# 🔍 Melihat Container

```bash
lxc list
```

---

# 🔑 Masuk ke Container

```bash
lxc exec debian-test bash
```

---

# 🛑 Start & Stop

```bash
lxc start debian-test
lxc stop debian-test
```

---

# 📦 Install Aplikasi di Container

Contoh di Debian:

```bash
apt update
apt install nginx
```

---

# 🌐 Akses dari Host

Cek IP:

```bash
lxc list
```

Buka di browser:

```
http://IP_CONTAINER
```

👉 Kamu baru saja membuat server 🔥

---

# ⚡ Kenapa LXD Cepat?

Karena sistem **image caching**:

- pertama kali → download image
    
- berikutnya → pakai cache lokal
    

Hasilnya:  
👉 container kedua jauh lebih cepat dibuat

---

# 📦 Melihat Cache Image

```bash
lxc image list
```

---

# 🧹 Hapus Container

```bash
lxc delete debian-test --force
```

---

# 💾 Snapshot (Backup Instan)

```bash
lxc snapshot debian-test snap1
```

Restore:

```bash
lxc restore debian-test snap1
```

---

# 🔁 Clone Container

```bash
lxc copy debian-test debian-copy
```

---

# 🌉 Port Forwarding (biar seperti native)

```bash
lxc config device add debian-test web proxy \
listen=tcp:0.0.0.0:8080 connect=tcp:127.0.0.1:80
```

Akses:

```
http://localhost:8080
```

---

# 🧠 LXD vs Docker (Singkat)

|LXD|Docker|
|---|---|
|Full OS|1 aplikasi|
|Persistent|Ephemeral|
|Seperti VM|Microservice|

---

# 🔥 Use Case Nyata

LXD sering dipakai untuk:

- 🧪 Testing multi distro
    
- 🏗️ Development backend
    
- 🖥️ Simulasi server
    
- ☁️ Private cloud mini
    
- 🧠 Lab DevOps
    

---

# ⚠️ Kesalahan Umum

❌ Menganggap LXD seperti Docker  
❌ Expect app langsung muncul di host  
❌ Tidak set UID mapping

---

# 🚀 Bonus: Integrasi dengan Tool Lain

Kombinasi powerful:

```
LXD → OS container
Podman → app container
```

👉 ini dipakai di banyak setup advanced

---

# 🎯 Kesimpulan

LXD adalah:

👉 “VM ringan berbasis container”  
👉 powerful untuk dev dan server  
👉 cocok untuk eksplorasi Linux

Kalau kamu ingin:

- belajar DevOps
    
- testing sistem
    
- bikin lab sendiri
    

👉 LXD adalah tools yang wajib kamu kuasai.
