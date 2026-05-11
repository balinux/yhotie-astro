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
**project Smart Monitoring Ruangan (ESP32 + Mosquitto + NodeJS + ThingsBoard)** dengan **penerapan RAE (Receive – Analyze – Execute)**

Fokusnya: **Edge benar-benar menjadi otak sistem**, bukan hanya forward data.

---

# 📘 Update Arsitektur Project dengan RAE

## Arsitektur Lama

```
ESP32 → Mosquitto → NodeJS → ThingsBoard
```

NodeJS hanya kirim data ke cloud.

Belum ada intelligence.

## Register thingsboard

[demo.thingsboard.io](http://demo.thingsboard.io)

[thingngsboard.cloud](http://thingngsboard.cloud) iyhotie : 00000000

## Install Mosquitto

[https://mosquitto.org/download/](https://mosquitto.org/download/)

---

# 🚀 Arsitektur Baru (RAE)

```
ESP32 → Mosquitto → NodeJS Edge → ThingsBoard
                     ↓
                   Control
```

---

# 📊 Diagram Sistem RAE

```
+-------------------+
|   Sensor DHT11/22 |
+-------------------+
        |
        v
+-------------------+
|      ESP32        |
| publish suhu      |
| subscribe kipas   |
+-------------------+
        |
        v
+-------------------+
|    Mosquitto      |
|   MQTT Broker     |
+-------------------+
        |
        v
+-----------------------------+
|         NodeJS Edge         |
|                             |
| Receive  → data suhu        |
| Analyze  → suhu > 30        |
| Execute  → kipas ON         |
| Send     → ThingsBoard      |
+-----------------------------+
        |
        v
+-------------------+
|   ThingsBoard     |
|   Dashboard
		device 
		cloud       |
+-------------------+
```

---

# 🎯 RAE yang Diterapkan

## R — Receive

Edge menerima data dari ESP32

```
kampus/suhu
```

contoh:

```
{ temperature: 32 }
```

---

## A — Analyze

Edge mengecek kondisi

```
if suhu > 30
```

---

## E — Execute

Edge kirim perintah

```
kampus/kipas ON
```

---

## Cloud

ThingsBoard hanya:

```
monitor
simpan data
dashboard
```

---

# 🧠 Flow Sistem

```
ESP32 kirim suhu

28
29
30
31
32

NodeJS menerima

cek suhu

> 30 ?

YES

kipas ON

kirim ke ThingsBoard
```

---

# 📦 Struktur Project Baru

```
iot-rae
│
├── esp32
│   └── esp32.ino
│
├── edge
│   ├── edge.js
│   ├── rules.js
│   └── config.js
│
└── README.md
```

---

# 📄 config.ts

```jsx
const mqttBroker = "mqtt://localhost:1883";
const thingsboard = {
  token: "token anda",
  url: "<http://demo.thingsboard.io/api/v1/>",
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

---

# 📄 rules.ts (Analyze Logic)

Ini inti RAE.

```jsx

function analyze(data: { temperature: number }) {
  let action: { kipas?: string; lampu?: string } = {};

  if (data.temperature > 30) {
    action.kipas = "ON";
  } else {
    action.kipas = "OFF";
  }

  if (data.temperature < 25) {
    action.lampu = "ON";
  } else {
    action.lampu = "OFF";
  }

  return action;
}

export default analyze;

```

---

# 📄 index.ts (RAE Engine)

```jsx
import mqtt from "mqtt";
import axios from "axios";

import config from "./edge/config";
import rules from "./edge/rules";

const client = mqtt.connect(config.mqttBroker);

client.on("connect", () => {

    console.log("Edge Connected");

    client.subscribe(config.topics.suhu);

});

client.on("message", async (topic, message) => {

    const data = JSON.parse(message.toString());

    console.log("Receive:", data);

    // ======================
    // ANALYZE
    // ======================

    const action = rules(data);

    console.log("Analyze:", action);

    // ======================
    // EXECUTE
    // ======================

    if (action.kipas) {

        client.publish(config.topics.kipas, action.kipas);
        console.log("Execute kipas:", action.kipas);

    }

    if (action.lampu) {

        client.publish(config.topics.lampu, action.lampu);
        console.log("Execute lampu:", action.lampu);

    }

    // ======================
    // SEND TO CLOUD / Thingsboard
    // ======================

const url = `http://thingsboard.cloud/api/v1/${config.thingsboard.token}/telemetry`;

    const payload = {
        temperature: data.temperature,
        kipas: action.kipas || "OFF",
        lampu: action.lampu || "OFF"
    };

    try {
        const res = await axios.post(url, payload, {
            headers: {
                "Content-Type": "application/json",
            },
        });

        console.log("Status:", res.status);
        console.log("Data:", res.data);
    } catch (err) {
        console.error("Error:", err.response?.data || err.message);
    }

});
```

## package.json

```json

{
  "name": "edge-computing-6c",
  "module": "index.ts",
  "type": "module",
  "private": true,
  "devDependencies": {
    "@types/bun": "latest"
  },
  "peerDependencies": {
    "typescript": "^5"
  }
}

```

---

# 📡 ESP32 Update

## Tambahkan LED

```
GPIO 2 = kipas
GPIO 4 = lampu
```

---

# ESP32 Code

```cpp
#include <DHT.h>
#include <PubSubClient.h>
#include <WiFi.h>

#define DHTPIN 5
#define DHTTYPE DHT22
#define FAN 4
#define LAMP 2

// Tambah buffer size yang lebih besar
#define MQTT_MAX_PACKET_SIZE 512

DHT dht(DHTPIN, DHTTYPE);

const char *ssid = "TP";
const char *password = "password";
const char *mqtt_server = "10.67.8.92";

WiFiClient espClient;
PubSubClient client(espClient);

// ✅ Gunakan millis() bukan delay()
unsigned long lastPublish = 0;
const unsigned long publishInterval = 5000;

void callback(char *topic, byte *message, unsigned int length) {
  Serial.println("\\n📩 MQTT Message Masuk");

  char payload[128];
  if (length >= sizeof(payload))
    length = sizeof(payload) - 1;
  for (unsigned int i = 0; i < length; i++)
    payload[i] = (char)message[i];
  payload[length] = '\\0';

  Serial.print("📌 Topic: ");
  Serial.println(topic);
  Serial.print("📌 Message: ");
  Serial.println(payload);

  if (strcmp(topic, "kampus/kipas") == 0) {
    if (strcmp(payload, "ON") == 0) {
      digitalWrite(FAN, HIGH);
      Serial.println("🌀 FAN: ON");
    } else {
      digitalWrite(FAN, LOW);
      Serial.println("🌀 FAN: OFF");
    }
  }

  if (strcmp(topic, "kampus/lampu") == 0) {
    if (strcmp(payload, "ON") == 0) {
      digitalWrite(LAMP, HIGH);
      Serial.println("💡 LAMP: ON");
    } else {
      digitalWrite(LAMP, LOW);
      Serial.println("💡 LAMP: OFF");
    }
  }
}

void reconnect() {
  while (!client.connected()) {
    Serial.print("🔄 Menghubungkan ke MQTT...");
    // ✅ Gunakan unique client ID
    String clientId = "ESP32-" + String(random(0xffff), HEX);
    if (client.connect(clientId.c_str())) {
      Serial.println("✅ Connected!");
      // ✅ Subscribe ulang setiap reconnect
      bool subResult = client.subscribe("kampus/#");
      Serial.print("📡 Subscribe kampus/# : ");
      Serial.println(subResult ? "OK" : "GAGAL");
    } else {
      Serial.print("❌ Gagal, rc=");
      Serial.print(client.state());
      Serial.println(" coba lagi 2 detik...");
      delay(2000);
    }
  }
}

void setup() {
  Serial.begin(115200);
  pinMode(FAN, OUTPUT);
  pinMode(LAMP, OUTPUT);
  dht.begin();
  delay(2000);

  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\\n✅ WiFi Connected!");
  Serial.println(WiFi.localIP());

  // ✅ Set buffer size lebih besar
  client.setBufferSize(512);
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);
}

void loop() {
  if (!client.connected())
    reconnect();

  // ✅ client.loop() selalu jalan tanpa diblokir delay
  client.loop();

  unsigned long now = millis();
  if (now - lastPublish >= publishInterval) {
    lastPublish = now;

    float suhu = dht.readTemperature();
    float hum = dht.readHumidity();

    if (isnan(suhu) || isnan(hum)) {
      suhu = random(200, 350) / 10.0;
      hum = random(600, 900) / 10.0;
    }

    String payload = "{\\"temperature\\":";
    payload += suhu;
    payload += "}";

    Serial.print("📤 Publish suhu: ");
    Serial.println(payload);
    client.publish("kampus/suhu", payload.c_str());
  }
}
```

|Fungsi|GPIO|Pin Board|
|---|---|---|
|FAN|GPIO4|D2|
|LAMP|GPIO2|D4|
|DHT|GPIO5|D1|

---

# 🎯 Flow Lengkap Sistem

## Step 1

ESP32 baca suhu

```
31
```

---

## Step 2

MQTT kirim

```
kampus/suhu
```

---

## Step 3

NodeJS Receive

```
{ temperature: 31 }
```

---

## Step 4

Analyze

```
> 30
```

---

## Step 5

Execute

```
kipas ON
```

---

## Step 6

ESP32 nyalakan LED

```
fan ON
```

---

## Step 7

ThingsBoard

```
grafik suhu
```

---

# 🧠 Peran Edge Sekarang

|Komponen|Peran|
|---|---|
|ESP32|Sensor dan actuator|
|Mosquitto|Broker|
|NodeJS|**Edge RAE Engine**|
|ThingsBoard|Cloud monitoring|

---

# 🎓 Learning Outcome Mahasiswa

Mahasiswa memahami:

### IoT Device

ESP32

---

### MQTT

Mosquitto

# Tes MQTT dari CMD

Gunakan 2 CMD (WAJIB)

---

## 🟢 Terminal 1 (Subscriber)

```
mosquitto_sub -h localhost -t test
```

👉 Ini buat “dengar” pesan

---

## 🔵 Terminal 2 (Publisher)

```
mosquitto_pub -h localhost -t test -m "halo dari CMD"
```

---

## ✅ Hasil

Di terminal subscriber akan muncul:

```
halo dari CMD
```

👉 Artinya MQTT sukses 🔥

---

### Edge

NodeJS

---

### RAE

Receive Analyze Execute

---

### Cloud

ThingsBoard

---

# 🚀 Upgrade Selanjutnya

RAE bisa ditambah:

### Motion Sensor

```
ruangan kosong
lampu mati
```

---

### Jadwal

```
jam 18:00
lampu mati
```

---

### AI

```
prediksi suhu
```

---

# 🎯 Hasil Akhir

Project sekarang bukan hanya monitoring.

Tetapi:

**Smart IoT System dengan Edge Intelligence (RAE)**

yang bisa:

- monitoring
- kontrol
- automation
- real-time
- cloud dashboard

---

## Tes thingsboard

### curl

```json
curl -v -X POST <http://demo.thingsboard.io/api/v1/token> anda/telemetry --header Content-Type:application/json --data "{temperature:25}"
```

## cURL windows http

```json
curl -v -X POST <http://thingsboard.cloud/api/v1/token/telemetry> --header Content-Type:application/json --data "{temperature:25}"
```

## mqtt windows

```json
mosquitto_pub -d -q 1 -h demo.thingsboard.io -p 1883 -t v1/devices/me/telemetry -u "token anda" -m "{temperature:25}"
```

```json
mosquitto_pub -d -q 1 -h thingsboard.cloud -p 1883 -t v1/devices/me/telemetry -u "token pribadi" -m "{temperature:26}"
```

## MQTT Linux

```json
mosquitto_pub -d -q 1 -h demo.thingsboard.io -p 1883 -t v1/devices/me/telemetry -u "token anda" -m "{temperature:25}"
```

## HTTP linux

```json
curl -v -X POST <http://demo.thingsboard.io/api/v1/token/telemetry> --header Content-Type:application/json --data "{temperature:25, humidity:60}"
```

### mosquito

tes kirim ke edge computing

```json
mosquitto_pub -t kampus/suhu -m '{"temperature":30}'
```

```json
mosquitto_pub -t kampus/suhu -m "{\\"temperature\\":30}"
```

```json
mosquitto_pub -h localhost -t kampus/suhu -m '{"temperature":32}'
```

with host IP

```json
mosquitto_pub -h 192.168.1.7 -t kampus/suhu -m '{"temperature":32}'
```

### run mosquito with config

```json
mosquitto -c /etc/mosquitto/mosquitto.conf -v 
```

### Allow firewall if using linux

```json
sudo ufw allow 1883
```

### jika mosquitto/mqtt broker bermasalah

```json
1. Edit config Mosquitto
Buka file:
/etc/mosquitto/mosquitto.conf
atau kalau di Windows:
mosquitto.conf

2. Tambahkan ini:
listener 1883
allow_anonymous true
Kalau mau eksplisit:
listener 1883 0.0.0.0
👉 Artinya:

Terima koneksi dari semua IP (LAN, ESP32, dll)

3. HAPUS ini kalau ada ❌
bind_address 127.0.0.1

4. Restart Mosquitto
Linux:
sudo systemctl restart mosquitto
atau:
mosquitto -c /etc/mosquitto/mosquitto.conf
Windows:

Restart service / buka ulang mosquitto
```