---
title: menjalankan aplikasi berbasis nodejs dengan container podman
description: tutrial dockerizing aplikasi Node.js dan menjalankannya dengan Podman
pubDatetime: 2026-01-28T16:00:00Z
tags:
  - nodejs
  - podman
draft: false
---
Berikut adalah langkah-langkah untuk **dockerizing aplikasi Node.js** dan menjalankannya dengan **Podman**:

---

### **1. Siapkan Aplikasi Node.js**

Buat direktori aplikasi Node.js jika belum ada. Misalnya:

```bash
mkdir nodejs-app
cd nodejs-app

```

Buat file `package.json`:

```json
{
  "name": "nodejs-app",
  "version": "1.0.0",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  }
}

```

Buat file `server.js` untuk aplikasi sederhana:

```jsx
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.send('Hello, Podman!');
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server is running on port ${PORT}`);
});

```

Instal dependensi:

```bash
npm install

```

---

### **2. Buat Dockerfile**

Buat file `Dockerfile` di direktori aplikasi:

```
# Gunakan nodejs-alpine base image
FROM node:22-alpine

# Tentukan working directory di container
WORKDIR /app

# Salin file package.json dan package-lock.json
COPY package*.json ./

# instalasi dependensi dan menghindari instalasi dependensi dev.
RUN npm install --only=production

# salin kode aplikasi
COPY . .

# expose port aplikasi
EXPOSE 3005

# perintah untuk menjalankan aplikasi
CMD [ "npm","start" ]
```

---

### **3. Tambahkan `.dockerignore`**

Untuk memastikan file yang tidak diperlukan tidak masuk ke dalam image, buat file `.dockerignore`:

```
node_modules
npm-debug,log
	.DS_Store 
```

---

### **4. Build Image**

Gunakan Podman untuk build image:

```bash
podman build -t nodejs-alpine .

```

---

### **5. Jalankan Container**

Jalankan container dengan Podman:

```bash
podman run -d -p 3005:3005 --name nodejs-app nodejs-alpine

```

- **`d`**: Jalankan container di background.
- **`p 3000:3000`**: Map port 3000 di host ke port 3000 di container.
- **`-name nodejs-container`**: Nama container.

---

### **6. Verifikasi**

Buka browser atau gunakan `curl` untuk mengecek apakah aplikasi berjalan:

```bash
curl <http://localhost:3005>

```

Output-nya akan berupa:

```
Hello, Podman!

```

---

### **7. Kelola Container**

Beberapa perintah dasar untuk mengelola container dengan Podman:

- **List container**:
    
    ```bash
    podman ps -a
    
    ```
    
- **Hentikan container**:
    
    ```bash
    podman stop nodejs-container
    
    ```
    
- **Hapus container**:
    
    ```bash
    podman rm nodejs-container
    
    ```
    
- **Hapus image**:
    
    ```bash
    podman rmi nodejs-app
    
    ```
    

---

Jika Anda menggunakan **rootless Podman**, container akan berjalan tanpa izin root, sehingga lebih aman. Selamat mencoba! 🚀