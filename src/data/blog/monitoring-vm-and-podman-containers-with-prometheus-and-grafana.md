---
title: Monitoring VM and Podman Containers with Prometheus and Grafana
description: Monitoring VM and Podman Containers with Prometheus and Grafana
pubDatetime: 2025-03-07T16:00:00Z
tags:
  - monitoring
  - grafana
draft: false
---
Untuk memonitoring server yang menjalankan aplikasi di VM dan container podman, Prometheus dan Grafana memang merupakan pilihan yang baik. Berikut langkah-langkah lengkap untuk mengimplementasikan sistem monitoring tersebut:

## 1. Instalasi Prometheus

```bash
# Update repository
sudo apt update  # Untuk Debian/Ubuntu
# atau
sudo dnf update  # Untuk RHEL/Fedora/CentOS

# Buat user Prometheus
sudo useradd --no-create-home --shell /bin/false prometheus

# Buat direktori untuk konfigurasi dan data
sudo mkdir -p /etc/prometheus /var/lib/prometheus

# Download Prometheus (ganti versi dengan yang terbaru)
wget <https://github.com/prometheus/prometheus/releases/download/v2.45.0/prometheus-2.45.0.linux-amd64.tar.gz>

# Ekstrak file
tar -xvf prometheus-2.45.0.linux-amd64.tar.gz

# Pindahkan binary ke lokasi yang sesuai
sudo cp prometheus-2.45.0.linux-amd64/prometheus /usr/local/bin/
sudo cp prometheus-2.45.0.linux-amd64/promtool /usr/local/bin/

# Pindahkan file konfigurasi
sudo cp -r prometheus-2.45.0.linux-amd64/consoles /etc/prometheus
sudo cp -r prometheus-2.45.0.linux-amd64/console_libraries /etc/prometheus

# Sesuaikan kepemilikan direktori
sudo chown -R prometheus:prometheus /etc/prometheus /var/lib/prometheus

```

## 2. Konfigurasi Prometheus

Buat file konfigurasi dasar:

```bash
sudo nano /etc/prometheus/prometheus.yml

```

Isi dengan konfigurasi dasar:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']

```

## 3. Buat service untuk Prometheus

```bash
sudo nano /etc/systemd/system/prometheus.service

```

Isi dengan:

```
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \\
    --config.file=/etc/prometheus/prometheus.yml \\
    --storage.tsdb.path=/var/lib/prometheus/ \\
    --web.console.templates=/etc/prometheus/consoles \\
    --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target

```

Mulai dan aktifkan service:

```bash
sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl enable prometheus

```

## 4. Instalasi Node Exporter (untuk monitoring VM)

```bash
# Download Node Exporter
wget <https://github.com/prometheus/node_exporter/releases/download/v1.6.0/node_exporter-1.6.0.linux-amd64.tar.gz>

# Ekstrak file
tar -xvf node_exporter-1.6.0.linux-amd64.tar.gz

# Pindahkan binary
sudo cp node_exporter-1.6.0.linux-amd64/node_exporter /usr/local/bin/

# Buat user untuk Node Exporter
sudo useradd --no-create-home --shell /bin/false node_exporter

# Buat service
sudo nano /etc/systemd/system/node_exporter.service

```

Isi file service dengan:

```
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target

```

Mulai dan aktifkan service:

```bash
sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl enable node_exporter

```

## 5. Konfigurasi Monitoring untuk Container Podman

### a. Instalasi cAdvisor untuk monitoring container

```bash
sudo podman run \\
  --name cadvisor \\
  --restart always \\
  --privileged \\
  --device=/dev/kmsg \\
  --net=host \\
  -v /:/rootfs:ro \\
  -v /var/run:/var/run:ro \\
  -v /sys:/sys:ro \\
  -v /var/lib/podman:/var/lib/podman:ro \\
  -v /var/lib/containers:/var/lib/containers:ro \\
  -v /dev/disk/:/dev/disk:ro \\
  -d \\
  gcr.io/cadvisor/cadvisor:v0.47.0

```

### b. Update Prometheus configuration untuk scrape data dari cAdvisor

```bash
sudo nano /etc/prometheus/prometheus.yml

```

Tambahkan konfigurasi berikut:

```yaml
  - job_name: 'cadvisor'
    static_configs:
      - targets: ['localhost:8080']

```

Restart Prometheus:

```bash
sudo systemctl restart prometheus

```

## 6. Instalasi Grafana

```bash
# Tambahkan repository (untuk Debian/Ubuntu)
sudo apt-get install -y apt-transport-https software-properties-common
wget -q -O - <https://packages.grafana.com/gpg.key> | sudo apt-key add -
sudo add-apt-repository "deb <https://packages.grafana.com/oss/deb> stable main"

# Atau untuk RHEL/CentOS
# sudo nano /etc/yum.repos.d/grafana.repo
# Tambahkan:
# [grafana]
# name=grafana
# baseurl=https://packages.grafana.com/oss/rpm
# repo_gpgcheck=1
# enabled=1
# gpgcheck=1
# gpgkey=https://packages.grafana.com/gpg.key
# sslverify=1
# sslcacert=/etc/pki/tls/certs/ca-bundle.crt

# Instalasi
sudo apt-get update
sudo apt-get install -y grafana

# Atau untuk RHEL/CentOS
# sudo dnf install grafana

# Mulai dan aktifkan service
sudo systemctl daemon-reload
sudo systemctl start grafana-server
sudo systemctl enable grafana-server

```

## 7. Konfigurasi Grafana

1. Akses Grafana di browser melalui [http://alamat-ip-server:3000](http://alamat-ip-server:3000)
2. Login dengan username dan password default (admin/admin)
3. Ganti password saat diminta

### a. Tambahkan Prometheus sebagai Data Source

1. Klik ikon gear (Settings) di sidebar kiri
2. Pilih "Data Sources"
3. Klik "Add data source"
4. Pilih "Prometheus"
5. Isikan URL: [http://localhost:9090](http://localhost:9090)
6. Klik "Save & Test"

### b. Import Dashboard untuk Node Exporter

1. Klik ikon "+" di sidebar kiri
2. Pilih "Import"
3. Masukkan ID: 1860 (Node Exporter Full dashboard)
4. Klik "Load"
5. Pilih Prometheus data source
6. Klik "Import"

### c. Import Dashboard untuk cAdvisor (Container Monitoring)

1. Klik ikon "+" di sidebar kiri
2. Pilih "Import"
3. Masukkan ID: 14282 (cAdvisor dashboard)
4. Klik "Load"
5. Pilih Prometheus data source
6. Klik "Import"

## 8. Konfigurasi Monitoring untuk Aplikasi Khusus di VM dan Containers

### a. Untuk aplikasi di VM

Jika aplikasi mendukung Prometheus, tambahkan endpoint metrics ke konfigurasi prometheus.yml:

```yaml
  - job_name: 'aplikasi-vm'
    static_configs:
      - targets: ['localhost:aplikasi-port']

```

### b. Untuk aplikasi di container Podman

Pastikan container mengekspos port metrics Prometheus dan tambahkan ke konfigurasi:

```yaml
  - job_name: 'aplikasi-container'
    static_configs:
      - targets: ['localhost:port-metrics-container']

```

## 9. Konfigurasi Alerting (Opsional)

### a. Konfigurasi Alertmanager

```bash
# Download Alertmanager
wget <https://github.com/prometheus/alertmanager/releases/download/v0.26.0/alertmanager-0.26.0.linux-amd64.tar.gz>

# Ekstrak
tar -xvf alertmanager-0.26.0.linux-amd64.tar.gz

# Pindahkan binary
sudo cp alertmanager-0.26.0.linux-amd64/alertmanager /usr/local/bin/

# Buat user
sudo useradd --no-create-home --shell /bin/false alertmanager

# Buat direktori
sudo mkdir -p /etc/alertmanager /var/lib/alertmanager

# Sesuaikan kepemilikan
sudo chown alertmanager:alertmanager /etc/alertmanager /var/lib/alertmanager

# Buat konfigurasi
sudo nano /etc/alertmanager/alertmanager.yml

```

Isi dengan:

```yaml
global:
  resolve_timeout: 5m

route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h
  receiver: 'email'

receivers:
- name: 'email'
  email_configs:
  - to: 'your-email@example.com'
    from: 'alertmanager@example.com'
    smarthost: smtp.example.com:587
    auth_username: 'your-email@example.com'
    auth_password: 'your-password'

```

Buat service untuk Alertmanager:

```bash
sudo nano /etc/systemd/system/alertmanager.service

```

Isi dengan:

```
[Unit]
Description=Alertmanager
Wants=network-online.target
After=network-online.target

[Service]
User=alertmanager
Group=alertmanager
Type=simple
ExecStart=/usr/local/bin/alertmanager \\
  --config.file=/etc/alertmanager/alertmanager.yml \\
  --storage.path=/var/lib/alertmanager

[Install]
WantedBy=multi-user.target

```

Mulai dan aktifkan service:

```bash
sudo systemctl daemon-reload
sudo systemctl start alertmanager
sudo systemctl enable alertmanager

```

### b. Konfigurasi Rules di Prometheus

```bash
sudo mkdir -p /etc/prometheus/rules
sudo nano /etc/prometheus/rules/alert.rules.yml

```

Isi dengan:

```yaml
groups:
- name: example
  rules:
  - alert: HighCPULoad
    expr: 100 - (avg by(instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "High CPU load (instance {{ $labels.instance }})"
      description: "CPU load is > 80%\\n  VALUE = {{ $value }}\\n  LABELS: {{ $labels }}"

```

Update konfigurasi Prometheus untuk menggunakan rules dan alertmanager:

```bash
sudo nano /etc/prometheus/prometheus.yml

```

Tambahkan:

```yaml
rule_files:
  - "rules/alert.rules.yml"

alerting:
  alertmanagers:
  - static_configs:
    - targets:
      - localhost:9093

```

Restart Prometheus:

```bash
sudo systemctl restart prometheus

```

## 10. Pertimbangan Keamanan

- Aktifkan autentikasi pada Prometheus dan Grafana
- Gunakan HTTPS untuk akses web UI
- Batasi akses ke port monitoring dengan firewall
- Pertimbangkan untuk menggunakan reverse proxy seperti Nginx

Dengan mengikuti langkah-langkah di atas, Anda akan memiliki sistem monitoring yang komprehensif untuk server VM dan container Podman menggunakan Prometheus dan Grafana.

[https://www.youtube.com/watch?v=M6ds-70GIpY&t=1753s&pp=ygUkc2VydmVyIG1vbml0b3JpbmcgcHJvbWV0aGV1cyBncmFmYW5h0gcJCX4JAYcqIYzv](https://www.youtube.com/watch?v=M6ds-70GIpY&t=1753s&pp=ygUkc2VydmVyIG1vbml0b3JpbmcgcHJvbWV0aGV1cyBncmFmYW5h0gcJCX4JAYcqIYzv)