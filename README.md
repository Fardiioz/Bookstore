#  Bookstore

Aplikasi web toko buku full-stack dengan sistem multi-role (Admin & User), mencakup katalog buku, keranjang belanja, pemesanan dengan pembayaran QR, live chat, ulasan produk, hingga laporan penjualan.

Proyek ini dibangun sebagai tugas mata kuliah (MKK), menggunakan arsitektur terpisah antara **backend (REST API)** dan **frontend (SPA)**.

---

## Fitur Utama

### Untuk Pengguna (User)
-  Registrasi, login, dan reset password via kode OTP email
-  Melihat katalog buku & detail buku per kategori
-  Keranjang belanja (tersimpan di database)
-  Checkout & pembayaran dengan QR Code
-  Riwayat pesanan & unduh invoice PDF
-  Memberi ulasan (review) pada buku
-  Live chat langsung dengan admin

### Untuk Admin
-  Manajemen buku (CRUD)
-  Manajemen kategori (CRUD)
-  Manajemen pengguna (CRUD)
-  Kasir — konfirmasi & scan pembayaran pesanan
-  Live chat dengan seluruh user
-  Laporan penjualan — export ke PDF & Excel

---

## Tech Stack

**Backend**
- [Laravel 12](https://laravel.com/) (PHP 8.2+)
- Laravel Sanctum — autentikasi SPA/token
- MySQL — database
- Maatwebsite/Excel & barryvdh/laravel-dompdf — export laporan (Excel & PDF)
- Intervention/Image — pengolahan gambar

**Frontend**
- [Nuxt 4](https://nuxt.com/) (Vue 3)
- Pinia — state management
- Tailwind CSS — styling
- Lucide Icons
- qrcode & jsQR — generate & scan QR pembayaran

---

##  Struktur Proyek

```
tugas-mkk-bookstore-main/
├── backend/
│   └── bookstore-api/      # Laravel REST API
└── frontend/
    └── bookstore-web/      # Nuxt SPA (client)
```

---

##  Instalasi & Menjalankan Proyek

### Prasyarat
- PHP >= 8.2 & Composer
- Node.js >= 18 & npm
- MySQL

### 1. Clone repository
```bash
git clone https://github.com/Fardiioz/Bookstore.git
cd Bookstore
```

### 2. Setup Backend (Laravel API)
```bash
cd backend/bookstore-api
composer install
cp .env.example .env
php artisan key:generate
```

Buka file `.env`, sesuaikan konfigurasi database dan email:
```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=bookstore
DB_USERNAME=root
DB_PASSWORD=

MAIL_MAILER=smtp
MAIL_HOST=smtp.gmail.com
MAIL_USERNAME=email_kamu@gmail.com
MAIL_PASSWORD=app_password_kamu
```

Jalankan migrasi database:
```bash
php artisan migrate
```

Jalankan server backend:
```bash
php artisan serve
```
Backend akan berjalan di `http://localhost:8000`

### 3. Setup Frontend (Nuxt)
```bash
cd ../../frontend/bookstore-web
npm install
cp .env.example .env
```

Pastikan isi `.env` frontend mengarah ke backend:
```env
NUXT_PUBLIC_API_BASE=http://localhost:8000
```

Jalankan server frontend:
```bash
npm run dev
```
Frontend akan berjalan di `http://localhost:3000`

---

##  Environment Variables

Kedua bagian proyek (`backend` dan `frontend`) memiliki file `.env.example` masing-masing sebagai template. **Jangan pernah** meng-commit file `.env` asli ke repository — file ini berisi kredensial sensitif (database, email, dsb) dan sudah dikecualikan lewat `.gitignore`.

| Lokasi | Fungsi |
|---|---|
| `backend/bookstore-api/.env.example` | Template konfigurasi Laravel (DB, mail, session, dll) |
| `frontend/bookstore-web/.env.example` | Template konfigurasi Nuxt (base URL API) |

---

##  Ringkasan API Endpoint

| Kategori | Endpoint | Keterangan |
|---|---|---|
| Auth | `POST /api/register`, `/login`, `/logout` | Autentikasi pengguna |
| Auth | `POST /api/forgot-password/*` | Reset password via OTP email |
| Katalog | `GET /api/books`, `/api/categories` | Data publik (read-only) |
| Keranjang | `GET/POST/PUT/DELETE /api/cart` | Kelola keranjang belanja |
| Pesanan | `GET/POST /api/orders`, `/api/orders/scan` | Pemesanan & scan pembayaran |
| Ulasan | `POST/PUT/DELETE /api/reviews` | Ulasan buku |
| Chat | `GET/POST /api/chats` | Live chat user ↔ admin |
| Admin | `/api/admin/books`, `/categories`, `/users` | CRUD data (khusus admin) |
| Admin | `/api/admin/reports`, `export-pdf`, `export-excel` | Laporan penjualan |

Autentikasi endpoint ber-role menggunakan **Laravel Sanctum** (Bearer Token).

---

##  Kontributor

Dikembangkan oleh **[Fardiioz](https://github.com/Fardiioz)** sebagai bagian dari tugas Mata Kuliah Keahlian (MKK).

---

## 📄 Lisensi

Proyek ini dibuat untuk keperluan tugas/edukasi.
