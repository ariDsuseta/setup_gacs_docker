# 🚀 GenieACS v1.2 Stack - Docker Deployment

[![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com/)
[![GenieACS](https://img.shields.io/badge/GenieACS-1.2.16-orange?style=for-the-badge)](https://genieacs.com/)
[![Debian](https://img.shields.io/badge/Debian-A81D33?style=for-the-badge&logo=debian&logoColor=white)](https://www.debian.org/)

Panduan lengkap instalasi GenieACS v1.2 menggunakan Docker Compose di lingkungan Debian. Konfigurasi ini telah dioptimalkan untuk performa stabil dan kompatibilitas CPU lama (Non-AVX).

---

## 📑 Daftar Isi
1. [Fitur Utama](#-fitur-utama)
2. [Prasyarat](#-prasyarat)
3. [Arsitektur Container](#-arsitektur-container)
4. [Langkah Instalasi](#-langkah-instalasi)
5. [Konfigurasi Router (CPE)](#-konfigurasi-router-cpe)
6. [Manajemen Virtual Parameters](#-manajemen-virtual-parameters)
7. [Troubleshooting](#-troubleshooting)

---

## ✨ Fitur Utama
* **Kompatibilitas Luas**: Menggunakan MongoDB 4.4 agar bisa berjalan di VPS/Server tanpa instruksi CPU AVX.
* **Auto-Healing**: Kebijakan `restart: always` memastikan service hidup kembali setelah server reboot atau mati listrik.
* **Storage Terpisah**: Data database dan logs dipisahkan ke dalam Docker Volumes agar aman saat update image.
* **Full Stack**: Termasuk CWMP, Northbound API, File Server, dan User Interface.

---

## 📋 Prasyarat
* OS: Debian 10/11/12 atau Ubuntu 20.04+.
* RAM: Minimal 2GB (Rekomendasi 4GB).
* Port Terbuka:
  * `3000` (Web Dashboard)
  * `7547` (CWMP - Komunikasi Router)
  * `7557` (NBI - API Interface)
  * `7567` (FS - File Server untuk Firmware Update)

---

## 🏗️ Arsitektur Container
Konfigurasi ini menjalankan 3 container utama:
1. **GenieACS Services**: Menjalankan core aplikasi.
2. **MongoDB 4.4**: Database untuk menyimpan data perangkat dan konfigurasi.
3. **Redis 6.2**: Cache broker untuk mempercepat respons komunikasi CWMP.

---

## 🚀 Langkah Instalasi

### 1. Update Sistem & Install Docker
```bash
sudo apt update && sudo apt install -y docker.io docker-compose
sudo systemctl enable --now docker
```
### 2: Membuat Direktori Project
Buat folder khusus agar data tidak berantakan:
```bash
mkdir ~/genieacs-docker && cd ~/genieacs-docker
```
### 3: Membuat Konfigurasi Docker Compose
Buat file `docker-compose.yml`:
```bash
nano docker-compose.yml
```
Lalu masukkan kode berikut (Sudah menggunakan MongoDB 4.4 untuk menghindari error AVX):
```bash
services:
  mongodb:
    image: mongo:4.4  # Ubah bagian ini
    container_name: genieacs-mongodb
    restart: always
    volumes:
      - mongo_data:/data/db

  redis:
    image: redis:6.2-alpine
    container_name: genieacs-redis
    restart: always

  genieacs:
    image: drumsergio/genieacs:1.2.16.0
    container_name: genieacs-services
    restart: always
    depends_on:
      - mongodb
      - redis
    environment:
      - GENIEACS_MONGODB_CONNECTION_URL=mongodb://mongodb:27017/genieacs
      - GENIEACS_REDIS_CONNECTION_URL=redis://redis:6379/0
      - GENIEACS_UI_JWT_SECRET=baztech_data_global # Ganti dengan secret yang kuat
      - GENIEACS_FS_IP=0.0.0.0
      - GENIEACS_CWMP_ACCESS_LOG_FILE=/var/log/genieacs/genieacs-cwmp-access.log
    ports:
      - "7547:7547"   # CWMP
      - "7557:7557"   # NBI
      - "7567:7567"   # FS (File Server)
      - "3000:3000"   # UI (Dashboard)
    volumes:
      - genieacs_data:/opt/genieacs/ext
      - genieacs_logs:/var/log/genieacs

  genieacs-panel-api:
    image: solusidigitalnet/genieacspanelapi:latest
    container_name: genieacs-panel-api
    ports:
      - "1996:1997"
    extra_hosts:
      - "host.docker.internal:host-gateway"
    environment:
      - JWT_SECRET=your-secret-key-here
      - JWT_EXPIRES_IN=1h
      - REFRESH_TOKEN_EXPIRES_IN=7d
      - add_wan=yes
      - NODE_ENV=production
    restart: always

volumes:
  mongo_data:
  genieacs_data:
  genieacs_logs:
```
### 4: Menjalankan GenieACS
Eksekusi perintah berikut untuk menarik image dan menjalankan container:
```bash
docker compose up -d
```
### 5: Verifikasi Status
Pastikan semua container berstatus Up:
```bash
docker compose ps
```
---
### 🔒 6. Keamanan
* Segera ganti GENIEACS_UI_JWT_SECRET di file docker-compose.yml.
* Jangan biarkan port 7557 terbuka untuk publik tanpa firewall (hanya untuk internal API).
* Gunakan Reverse Proxy (seperti Nginx) jika ingin mengaktifkan SSL/HTTPS.
---
### 🌍 7. Akses
* `gacs` : (IP_HOST:3000)
* `g-dashboard` : (IP-HOST:1996)
