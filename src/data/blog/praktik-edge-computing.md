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

# 📘 Praktikum Edge Computing
## Smart Monitoring Ruangan dengan RAE Pattern

**Project:** Smart Monitoring Ruangan (ESP32 + Mosquitto + NodeJS + ThingsBoard)

**Konsep Utama:** Penerapan pola **RAE (Receive – Analyze – Execute)** di mana Edge berfungsi sebagai "otak" sistem, bukan sekadar meneruskan data ke cloud.

---

## 📋 Daftar Isi

1. [Tujuan Pembelajaran](#tujuan-pembelajaran)
2. [Prasyarat](#prasyarat)
3. [Alat dan Bahan](#alat-dan-bahan)
4. [Arsitektur Sistem](#arsitektur-sistem)
5. [Persiapan Lingkungan](#persiapan-lingkungan)
6. [Implementasi](#implementasi)
7. [Pengujian](#pengujian)
8. [Latihan dan Tugas](#latihan-dan-tugas)
9. [Troubleshooting](#troubleshooting)
10. [Evaluasi](#evaluasi)

---

## 🎯 Tujuan Pembelajaran

Setelah mengikuti praktikum ini, mahasiswa diharapkan mampu:

| No | Tujuan Pembelajaran |
|----|---------------------|
| 1 | Memahami konsep dasar Edge Computing dan perbedaannya dengan Cloud Computing |
| 2 | Mengimplementasikan pola RAE (Receive-Analyze-Execute) pada sistem IoT |
| 3 | Mengkonfigurasi dan menggunakan MQTT Broker (Mosquitto) untuk komunikasi IoT |
| 4 | Mengembangkan aplikasi Edge menggunakan NodeJS/TypeScript |
| 5 | Mengintegrasikan sistem IoT dengan platform Cloud (ThingsBoard) |
| 6 | Menganalisis dan mengevaluasi performa sistem Edge Computing |

---

## 📚 Prasyarat

### Pengetahuan Dasar
- Pemrograman dasar (JavaScript/TypeScript)
- Konsep dasar IoT dan MQTT
- Penggunaan terminal/command line

### Software yang Diperlukan
- Node.js (v18 atau lebih baru)
- TypeScript
- MQTT Client (mosquitto_pub/mosquitto_sub)
- Arduino IDE (untuk ESP32)
- Git (opsional)

---

## 🛠️ Alat dan Bahan

### Hardware
| Komponen | Jumlah | Keterangan |
|----------|--------|------------|
| ESP32 | 1 | Microcontroller dengan WiFi |
| DHT22/DHT11 | 1 | Sensor suhu dan kelembaban |
| LED | 2 | Merah (kipas), Kuning (lampu) |
| Resistor 220Ω | 2 | Untuk LED |
| Breadboard | 1 | |
| Kabel Jumper | - | Sesuai kebutuhan |

### Pinout ESP32
| Fungsi | GPIO | Pin Board |
|--------|------|-----------|
| FAN (LED Merah) | GPIO 4 | D2 |
| LAMP (LED Kuning) | GPIO 2 | D4 |
| DHT Sensor | GPIO 5 | D1 |

---

## 🏗️ Arsitektur Sistem

### Arsitektur Lama (Tanpa Edge Intelligence)
```
ESP32 → Mosquitto → NodeJS → ThingsBoard
```
❌ NodeJS hanya meneruskan data ke cloud, tidak ada intelligence lokal.

### Arsitektur Baru (RAE Pattern)
```
ESP32 → Mosquitto → NodeJS Edge → ThingsBoard
                     ↓
                   Control Loop
```
✅ NodeJS Edge berfungsi sebagai otak yang menganalisis dan mengambil keputusan.

### Diagram Sistem Lengkap
```
+-------------------+       +-------------------+
|   Sensor DHT22    |       |   ThingsBoard     |
+-------------------+       +-------------------+
        |                           ^
        v                           |
+-------------------+       +-------------------+
|      ESP32        |       |   Cloud Storage   |
| - Publish suhu    |       +-------------------+
| - Subscribe kipas |
| - Subscribe lampu |
+-------------------+
        |                           ^
        v                           |
+-------------------+       +-------------------+
|    Mosquitto      |       |   Dashboard       |
|   MQTT Broker     |       +-------------------+
+-------------------+
        |
        v
+-----------------------------+
|         NodeJS Edge         |
|                             |
|  ┌─────────────────────┐   |
|  │ RECEIVE → data suhu │   |
|  ├─────────────────────┤   |
|  │ ANALYZE → suhu > 30 │   |
|  ├─────────────────────┤   |
|  │ EXECUTE → kipas ON  │   |
|  ├─────────────────────┤   |
|  │ SEND → ThingsBoard  │   |
|  └─────────────────────┘   |
+-----------------------------+
```

---

## 🔧 Persiapan Lingkungan

### Langkah 1: Register ThingsBoard

1. Buka [demo.thingsboard.io](http://demo.thingsboard.io) atau [thingsboard.cloud](http://thingsboard.cloud)
2. Login atau daftar akun baru
3. Buat Device baru dan copy **Access Token**

### Langkah 2: Install Mosquitto MQTT Broker

#### Windows
```bash
# Download dari: https://mosquitto.org/download/
# Install dan jalankan sebagai service
```

#### Linux (Ubuntu/Debian)
```bash
sudo apt update
sudo apt install mosquitto mosquitto-clients
sudo systemctl start mosquitto
sudo systemctl enable mosquitto
```

#### macOS
```bash
brew install mosquitto
brew services start mosquitto
```

### Langkah 3: Konfigurasi Mosquitto

Edit file konfigurasi `/etc/mosquitto/mosquitto.conf` (Linux) atau `mosquitto.conf` (Windows):

```conf
# Izinkan koneksi dari semua IP
listener 1883 0.0.0.0
allow_anonymous true

# Hapus atau comment baris ini jika ada:
# bind_address 127.0.0.1
```

Restart Mosquitto:
```bash
# Linux
sudo systemctl restart mosquitto

# Windows
# Restart service melalui Services Manager
```

### Langkah 4: Setup Firewall (Linux)
```bash
sudo ufw allow 1883/tcp
```

### Langkah 5: Inisialisasi Project NodeJS

```bash
# Buat folder project
mkdir iot-rae
cd iot-rae

# Inisialisasi project
npm init -y

# Install dependencies
npm install mqtt axios

# Install TypeScript (opsional)
npm install -D typescript @types/node @types/mqtt ts-node
```

---

## 💻 Implementasi

### Struktur Project
```
iot-rae/
│
├── esp32/
│   └── esp32.ino          # Code untuk ESP32
│
├── edge/
│   ├── config.ts          # Konfigurasi sistem
│   ├── rules.ts           # Logika analisis (Analyze)
│   └── index.ts           # RAE Engine utama
│
├── package.json
└── README.md
```

### File 1: `edge/config.ts`

```typescript
const mqttBroker = "mqtt://localhost:1883";
const thingsboard = {
  token: "GANTI_DENGAN_TOKEN_ANDA",
  url: "http://thingsboard.cloud/api/v1/",
};
const topics = {
  suhu: "kampus/suhu",
  kipas: "kampus/kipas",
  lampu: "kampus/lampu",
};

export default {
  mqttBroker,
  thingsboard,
  topics,
};
```

### File 2: `edge/rules.ts`

```typescript
/**
 * Logika ANALYZE dari pola RAE
 * Menganalisis data sensor dan menentukan aksi yang diperlukan
 */

interface SensorData {
  temperature: number;
  humidity?: number;
}

interface Action {
  kipas?: string;
  lampu?: string;
}

function analyze(data: SensorData): Action {
  const action: Action = {};

  // Rule 1: Jika suhu > 30°C, nyalakan kipas
  if (data.temperature > 30) {
    action.kipas = "ON";
  } else {
    action.kipas = "OFF";
  }

  // Rule 2: Jika suhu < 25°C, nyalakan lampu (indikator dingin)
  if (data.temperature < 25) {
    action.lampu = "ON";
  } else {
    action.lampu = "OFF";
  }

  return action;
}

export default analyze;
```

### File 3: `edge/index.ts`

```typescript
/**
 * RAE Engine - Receive, Analyze, Execute
 * 
 * Flow:
 * 1. RECEIVE - Menerima data dari ESP32 via MQTT
 * 2. ANALYZE - Menganalisis data menggunakan rules
 * 3. EXECUTE - Mengirim perintah kontrol ke ESP32
 * 4. SEND - Mengirim data ke ThingsBoard untuk monitoring
 */

import mqtt from "mqtt";
import axios from "axios";
import config from "./config.js";
import rules from "./rules.js";

const client = mqtt.connect(config.mqttBroker);

// ======================
// RECEIVE
// ======================
client.on("connect", () => {
  console.log("✅ Edge Connected to MQTT Broker");
  
  // Subscribe ke topic sensor
  client.subscribe(config.topics.suhu, (err) => {
    if (err) {
      console.error("❌ Subscribe error:", err);
    } else {
      console.log(`📡 Subscribed to: ${config.topics.suhu}`);
    }
  });
});

client.on("message", async (topic, message) => {
  try {
    // Parse data sensor
    const data = JSON.parse(message.toString());
    console.log(`\n📩 [RECEIVE] Topic: ${topic}`);
    console.log(`   Data:`, data);

    // ======================
    // ANALYZE
    // ======================
    const action = rules(data);
    console.log(`🧠 [ANALYZE] Action:`, action);

    // ======================
    // EXECUTE
    // ======================
    if (action.kipas) {
      client.publish(config.topics.kipas, action.kipas);
      console.log(`⚡ [EXECUTE] Kipas: ${action.kipas}`);
    }

    if (action.lampu) {
      client.publish(config.topics.lampu, action.lampu);
      console.log(`⚡ [EXECUTE] Lampu: ${action.lampu}`);
    }

    // ======================
    // SEND TO CLOUD
    // ======================
    await sendToThingsBoard(data, action);

  } catch (error) {
    console.error("❌ Error processing message:", error);
  }
});

/**
 * Mengirim data ke ThingsBoard Cloud
 */
async function sendToThingsBoard(sensorData: any, action: any) {
  const url = `${config.thingsboard.url}${config.thingsboard.token}/telemetry`;
  
  const payload = {
    temperature: sensorData.temperature,
    humidity: sensorData.humidity || 0,
    kipas: action.kipas || "OFF",
    lampu: action.lampu || "OFF",
    timestamp: Date.now()
  };

  try {
    const res = await axios.post(url, payload, {
      headers: { "Content-Type": "application/json" },
    });
    console.log(`☁️  [CLOUD] Data sent to ThingsBoard - Status: ${res.status}`);
  } catch (err: any) {
    console.error("❌ [CLOUD] Error:", err.response?.data || err.message);
  }
}

// Error handling
client.on("error", (err) => {
  console.error("MQTT Client Error:", err);
});

console.log("🚀 RAE Engine started...");
```

### File 4: `package.json`

```json
{
  "name": "edge-computing-rae",
  "version": "1.0.0",
  "description": "Edge Computing dengan pola RAE",
  "type": "module",
  "main": "edge/index.ts",
  "scripts": {
    "start": "node edge/index.js",
    "dev": "ts-node edge/index.ts",
    "build": "tsc"
  },
  "keywords": ["edge", "iot", "rae", "mqtt"],
  "author": "",
  "license": "MIT",
  "dependencies": {
    "axios": "^1.6.0",
    "mqtt": "^5.0.0"
  },
  "devDependencies": {
    "@types/mqtt": "^2.5.0",
    "@types/node": "^20.0.0",
    "ts-node": "^10.9.0",
    "typescript": "^5.0.0"
  }
}
```

### File 5: `esp32/esp32.ino`

```cpp
/**
 * ESP32 Smart Room Monitoring
 * 
 * Hardware:
 * - DHT22 di GPIO 5
 * - LED Kipas (Merah) di GPIO 4
 * - LED Lampu (Kuning) di GPIO 2
 */

#include <DHT.h>
#include <PubSubClient.h>
#include <WiFi.h>

// ======================
// Konfigurasi Hardware
// ======================
#define DHTPIN 5
#define DHTTYPE DHT22
#define FAN_PIN 4
#define LAMP_PIN 2

#define MQTT_MAX_PACKET_SIZE 512

// ======================
// Konfigurasi WiFi
// ======================
const char* ssid = "NAMA_WIFI_ANDA";
const char* password = "PASSWORD_WIFI_ANDA";

// ======================
// Konfigurasi MQTT
// ======================
const char* mqtt_server = "10.67.8.92";  // Ganti dengan IP broker
const int mqtt_port = 1883;

// ======================
// Inisialisasi Objek
// ======================
DHT dht(DHTPIN, DHTTYPE);
WiFiClient espClient;
PubSubClient client(espClient);

// ======================
// Variabel Timer
// ======================
unsigned long lastPublish = 0;
const unsigned long publishInterval = 5000;  // 5 detik

// ======================
// Callback MQTT
// ======================
void callback(char* topic, byte* message, unsigned int length) {
  Serial.println("\n📩 MQTT Message Masuk");

  // Parse message
  char payload[128];
  if (length >= sizeof(payload)) length = sizeof(payload) - 1;
  for (unsigned int i = 0; i < length; i++) {
    payload[i] = (char)message[i];
  }
  payload[length] = '\0';

  Serial.print("📌 Topic: ");
  Serial.println(topic);
  Serial.print("📌 Message: ");
  Serial.println(payload);

  // ======================
  // Execute Commands
  // ======================
  
  // Kontrol Kipas
  if (strcmp(topic, "kampus/kipas") == 0) {
    if (strcmp(payload, "ON") == 0) {
      digitalWrite(FAN_PIN, HIGH);
      Serial.println("🌀 FAN: ON");
    } else {
      digitalWrite(FAN_PIN, LOW);
      Serial.println("🌀 FAN: OFF");
    }
  }

  // Kontrol Lampu
  if (strcmp(topic, "kampus/lampu") == 0) {
    if (strcmp(payload, "ON") == 0) {
      digitalWrite(LAMP_PIN, HIGH);
      Serial.println("💡 LAMP: ON");
    } else {
      digitalWrite(LAMP_PIN, LOW);
      Serial.println("💡 LAMP: OFF");
    }
  }
}

// ======================
// Reconnect MQTT
// ======================
void reconnect() {
  while (!client.connected()) {
    Serial.print("🔄 Menghubungkan ke MQTT...");
    
    // Generate unique client ID
    String clientId = "ESP32-" + String(random(0xffff), HEX);
    
    if (client.connect(clientId.c_str())) {
      Serial.println("✅ Connected!");
      
      // Subscribe ke topik kontrol
      client.subscribe("kampus/kipas");
      client.subscribe("kampus/lampu");
      Serial.println("📡 Subscribed to control topics");
    } else {
      Serial.print("❌ Gagal, rc=");
      Serial.print(client.state());
      Serial.println(" coba lagi 2 detik...");
      delay(2000);
    }
  }
}

// ======================
// Setup
// ======================
void setup() {
  Serial.begin(115200);
  delay(2000);

  // Setup pins
  pinMode(FAN_PIN, OUTPUT);
  pinMode(LAMP_PIN, OUTPUT);
  digitalWrite(FAN_PIN, LOW);
  digitalWrite(LAMP_PIN, LOW);

  // Setup sensor
  dht.begin();

  // Connect WiFi
  Serial.println("\n📡 Menghubungkan ke WiFi...");
  WiFi.begin(ssid, password);
  
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  
  Serial.println("\n✅ WiFi Connected!");
  Serial.print("📡 IP Address: ");
  Serial.println(WiFi.localIP());

  // Setup MQTT
  client.setBufferSize(MQTT_MAX_PACKET_SIZE);
  client.setServer(mqtt_server, mqtt_port);
  client.setCallback(callback);
}

// ======================
// Loop
// ======================
void loop() {
  // Reconnect jika terputus
  if (!client.connected()) {
    reconnect();
  }
  
  // Process MQTT messages
  client.loop();

  // Publish sensor data secara periodik
  unsigned long now = millis();
  if (now - lastPublish >= publishInterval) {
    lastPublish = now;

    // Baca sensor
    float suhu = dht.readTemperature();
    float hum = dht.readHumidity();

    // Handle error pembacaan sensor
    if (isnan(suhu) || isnan(hum)) {
      Serial.println("⚠️ Sensor error, menggunakan nilai dummy");
      suhu = random(200, 350) / 10.0;
      hum = random(600, 900) / 10.0;
    }

    // Buat payload JSON
    String payload = "{\"temperature\":";
    payload += suhu;
    payload += ",\"humidity\":";
    payload += hum;
    payload += "}";

    // Publish ke MQTT
    Serial.print("📤 Publish suhu: ");
    Serial.println(payload);
    
    bool success = client.publish("kampus/suhu", payload.c_str());
    if (success) {
      Serial.println("✅ Data terkirim");
    } else {
      Serial.println("❌ Gagal mengirim data");
    }
  }
}
```

---

## 🧪 Pengujian

### Tahap 1: Tes MQTT Broker

Buka 2 terminal/command prompt:

#### Terminal 1 (Subscriber)
```bash
mosquitto_sub -h localhost -t "test/#" -v
```

#### Terminal 2 (Publisher)
```bash
mosquitto_pub -h localhost -t "test/hello" -m "Halo dari terminal!"
```

**Hasil yang diharapkan:** Pesan muncul di terminal subscriber.

### Tahap 2: Tes Kirim Data ke Edge

```bash
# Tes kirim data suhu ke edge
mosquitto_pub -h localhost -t "kampus/suhu" -m '{"temperature":32}'

# Tes dengan suhu rendah
mosquitto_pub -h localhost -t "kampus/suhu" -m '{"temperature":23}'

# Tes dengan suhu tinggi
mosquitto_pub -h localhost -t "kampus/suhu" -m '{"temperature":35}'
```

**Hasil yang diharapkan:**
- Suhu > 30: Kipas ON
- Suhu < 25: Lampu ON

### Tahap 3: Tes ThingsBoard

#### Via HTTP (cURL)
```bash
# Windows
curl -v -X POST http://thingsboard.cloud/api/v1/TOKEN_ANDA/telemetry --header "Content-Type:application/json" --data "{\"temperature\":25,\"humidity\":60}"

# Linux/macOS
curl -v -X POST "http://thingsboard.cloud/api/v1/TOKEN_ANDA/telemetry" -H "Content-Type: application/json" -d '{"temperature":25,"humidity":60}'
```

#### Via MQTT
```bash
mosquitto_pub -d -q 1 -h thingsboard.cloud -p 1883 -t "v1/devices/me/telemetry" -u "TOKEN_ANDA" -m '{"temperature":26,"humidity":65}'
```

### Tahap 4: Tes Sistem Lengkap

1. Jalankan NodeJS Edge:
```bash
npm start
```

2. Upload code ke ESP32 dan buka Serial Monitor

3. Amati flow sistem:
```
ESP32 → MQTT → Edge (RAE) → ESP32 (Control) → ThingsBoard
```

---

## 📝 Latihan dan Tugas

### Tugas 1: Modifikasi Rules (Bobot: 20%)

Tambahkan rule baru di `rules.ts`:
- Jika kelembaban > 70%, aktifkan peringatan
- Jika suhu > 35°C, aktifkan mode "darurat" (kipas + lampu ON)

### Tugas 2: Tambah Sensor Baru (Bobot: 20%)

Tambahkan sensor gerak (PIR) ke sistem:
- ESP32 membaca status gerak
- Edge menganalisis: jika tidak ada gerak selama 5 menit, matikan lampu

### Tugas 3: Implementasi Jadwal (Bobot: 20%)

Tambahkan fitur jadwal otomatis:
- Lampu otomatis mati jam 18:00 - 06:00
- Kipas otomatis ON jam 08:00 - 17:00 jika suhu > 28°C

### Tugas 4: Logging dan Monitoring (Bobot: 20%)

Tambahkan fitur:
- Simpan log semua keputusan Edge ke file
- Tampilkan statistik: berapa kali kipas ON/OFF hari ini
- Hitung rata-rata suhu per jam

### Tugas 5: Laporan Analisis (Bobot:20%)

Buat laporan yang berisi:
1. Diagram arsitektur sistem yang sudah dibangun
2. Penjelasan flow RAE dalam sistem
3. Perbandingan performa: dengan vs tanpa Edge Intelligence
4. Analisis kelebihan dan kekurangan implementasi
5. Rekomendasi pengembangan selanjutnya

---

## 🔧 Troubleshooting

### Masalah 1: ESP32 Tidak Connect ke MQTT

**Gejala:** Serial Monitor menampilkan "Gagal, rc=-2"

**Solusi:**
```bash
# 1. Cek koneksi WiFi
Serial.print("IP: ");
Serial.println(WiFi.localIP());

# 2. Cek IP broker (ping dari komputer)
ping 10.67.8.92

# 3. Pastikan Mosquitto allow anonymous
# Edit mosquitto.conf:
allow_anonymous true

# 4. Restart Mosquitto
sudo systemctl restart mosquitto
```

### Masalah 2: Data Tidak Muncul di ThingsBoard

**Gejala:** Tidak ada data di dashboard

**Solusi:**
```bash
# 1. Cek token device
# Pastikan token di config.ts benar

# 2. Tes manual dengan curl
curl -X POST "http://thingsboard.cloud/api/v1/TOKEN/telemetry" \
  -H "Content-Type: application/json" \
  -d '{"temperature":25}'

# 3. Cek log NodeJS untuk error
# Pastikan tidak ada error saat kirim data
```

### Masalah 3: Mosquitto Connection Refused

**Gejala:** "Connection refused" saat connect

**Solusi:**
```bash
# 1. Cek status Mosquitto
sudo systemctl status mosquitto

# 2. Cek port yang digunakan
sudo netstat -tlnp | grep 1883

# 3. Pastikan tidak ada service lain di port 1883

# 4. Cek firewall
sudo ufw status
sudo ufw allow 1883/tcp
```

### Masalah 4: ESP32 Reset Terus Menerus

**Gejala:** ESP32 restart berulang

**Solusi:**
```cpp
// 1. Tambahkan delay di setup
void setup() {
  delay(2000);  // Tunggu serial monitor siap
  // ...
}

// 2. Cek power supply
// Gunakan power supply yang stabil

// 3. Kurangi buffer size jika memory penuh
#define MQTT_MAX_PACKET_SIZE 256  // Kurangi dari 512
```

### Masalah 5: Sensor DHT Mengembalikan NaN

**Gejala:** Pembacaan sensor selalu NaN

**Solusi:**
```cpp
// 1. Cek wiring sensor
// Pastikan VCC, GND, Data terhubung benar

// 2. Tambahkan pull-up resistor 10kΩ jika perlu

// 3. Tambahkan delay antar pembacaan
void loop() {
  // ...
  delay(2000);  // DHT butuh delay minimal 2 detik
}
```

---

## 📊 Evaluasi

### Kriteria Penilaian

| Aspek | Bobot | Kriteria |
|-------|-------|-----------|
| Fungsionalitas | 30% | Sistem berjalan sesuai spesifikasi RAE |
| Kode Quality | 20% | Kode rapi, terstruktur, terdokumentasi |
| Inovasi | 20% | Penambahan fitur di luar requirement dasar |
| Laporan | 20% | Laporan lengkap dan analisis mendalam |
| Presentasi | 10% | Penjelasan sistem dan demo |

### Rubrik Detail

#### Fungsionalitas (30%)
| Skor | Deskripsi |
|------|-----------|
| 25-30 | Semua fitur berjalan sempurna, ESP32-Edge-Cloud terintegrasi |
| 20-24 | Mayoritas fitur berjalan, ada minor bug |
| 15-19 | Fitur dasar berjalan, beberapa fitur tidak |
| 0-14 | Sistem tidak berfungsi |

#### Kode Quality (20%)
| Skor | Deskripsi |
|------|-----------|
| 17-20 | Kode sangat rapi, well-documented, best practices |
| 13-16 | Kode rapi, dokumentasi cukup |
| 9-12 | Kode berfungsi tapi kurang rapi |
| 0-8 | Kode berantakan, sulit dibaca |

#### Inovasi (20%)
| Skor | Deskripsi |
|------|-----------|
| 17-20 | Fitur tambahan yang kreatif dan berguna |
| 13-16 | Beberapa fitur tambahan |
| 9-12 | Minimal fitur tambahan |
| 0-8 | Tidak ada inovasi |

---

## 🎓 Ringkasan Materi

### Konsep RAE

| Tahap | Deskripsi | Implementasi |
|-------|-----------|--------------|
| **R** - Receive | Menerima data dari sensor | MQTT Subscribe ke `kampus/suhu` |
| **A** - Analyze | Menganalisis data dan buat keputusan | Fungsi `rules()` mengecek kondisi |
| **E** - Execute | Melakukan aksi berdasarkan keputusan | MQTT Publish ke `kampus/kipas`, `kampus/lampu` |

### Peran Komponen

| Komponen | Peran dalam Sistem |
|----------|-------------------|
| **ESP32** | IoT Device - Sensor & Actuator |
| **Mosquitto** | MQTT Broker - Transport layer |
| **NodeJS Edge** | Edge Intelligence - RAE Engine |
| **ThingsBoard** | Cloud Platform - Monitoring & Storage |

### Keunggulan Edge Computing

✅ **Latensi Rendah** - Keputusan diambil lokal, tidak perlu tunggu cloud  
✅ **Bandwidth Efisien** - Hanya data penting dikirim ke cloud  
✅ **Offline Capability** - Sistem tetap jalan meski cloud down  
✅ **Privacy** - Data sensitif diproses lokal  
✅ **Real-time Response** - Kontrol instan tanpa delay

---

## 🚀 Pengembangan Lanjutan

### Ide Pengembangan

1. **Machine Learning di Edge**
   - Prediksi suhu menggunakan model ML sederhana
   - Anomaly detection untuk pola tidak normal

2. **Multi-Device Management**
   - Banyak ESP32 dalam satu sistem
   - Load balancing antar device

3. **Dashboard Lokal**
   - Web dashboard lokal menggunakan Express.js
   - Real-time update dengan WebSocket

4. **Alert System**
   - Notifikasi email/Telegram saat suhu > threshold
   - SMS alert untuk kondisi darurat

5. **Data Analytics**
   - Analisis tren suhu harian/mingguan
   - Export data ke CSV untuk analisis lebih lanjut

---

## 📚 Referensi

- [MQTT Essentials](https://www.hivemq.com/mqtt-essentials/)
- [ThingsBoard Documentation](https://thingsboard.io/docs/)
- [Edge Computing Best Practices](https://aws.amazon.com/edge-computing/)
- [ESP32 MQTT Guide](https://randomnerdtutorials.com/esp32-mqtt-publish-subscribe-arduino-ide/)

---

## 📞 Kontak & Bantuan

Jika mengalami kesulitan:
1. Cek bagian **Troubleshooting** di atas
2. Diskusikan dengan kelompok/asisten
3. Buat issue di repository dengan detail error

---

**Selamat Mengerjakan! 🎉**

*Ingat: Edge Computing bukan tentang menggantikan cloud, tapi tentang complement - bekerja sama untuk sistem yang lebih cerdas dan efisien.*