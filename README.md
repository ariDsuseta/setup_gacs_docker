# 🚀 GenieACS v1.2 Docker Setup (AVX Optimized)

Repositori ini berisi panduan dan file konfigurasi untuk menjalankan **GenieACS** menggunakan Docker Compose. Konfigurasi ini telah disesuaikan untuk berjalan di server dengan CPU yang tidak mendukung instruksi AVX (menggunakan MongoDB 4.4).

---

## 🛠️ Komponen Stack
* **GenieACS 1.2.16.0**: Main service (CWMP, NBI, FS, UI).
* **MongoDB 4.4**: Database (Versi stabil untuk CPU lama/VPS murah).
* **Redis 6.2**: Cache broker untuk meningkatkan performa.

---

## 📦 Cara Instalasi

### 1. Persiapan Direktori
Login ke Debian kamu dan jalankan:
```bash
mkdir genieacs-docker && cd genieacs-docker
