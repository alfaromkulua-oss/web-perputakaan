# BookBase — Sistem Informasi Perpustakaan Digital (Node.js + MySQL)

Aplikasi **Website Perpustakaan Digital (Sistem Informasi Perpustakaan)** yang modern, profesional, responsif, dan terhubung langsung secara nyata dengan database MySQL **`db_perpustakaan`** (menggunakan **Node.js, Express, MySQL2, dan Tailwind CSS** dengan palet warna **Biru Navy & Putih** bergaya *BookBase*).

---

## 1. Akun Pengujian Awal (Admin & Anggota)

Setelah menjalankan `npm run db:setup` atau mengimpor file SQL, Anda dapat langsung login dengan akun berikut:

| Peran (Role) | Email | Password | Hak Akses Utama |
| :--- | :--- | :--- | :--- |
| **Admin** | `admin@perpustakaan.id` | `admin123` | Dashboard & grafik, CRUD buku + upload cover, CRUD anggota, setujui/tolak pinjaman, proses pengembalian & denda otomatis, cetak/ekspor 6 laporan (PDF & Excel). |
| **Anggota** | `anggota@perpustakaan.id` | `anggota123` | Melihat beranda & katalog buku, mencari/memfilter koleksi, mengajukan peminjaman buku, dan memantau riwayat peminjaman & status denda. |

> **Tips:** Di halaman **Login**, tersedia tombol **"Akun Pengujian Cepat"** yang dapat diklik untuk mengisi email dan password Admin atau Anggota secara otomatis.

---

## 2. Struktur Folder Proyek

```text
Web perpustakaan/
├── database/
│   ├── schema.sql                # Script DDL pembuatan 9 tabel & relasi PK/FK (db_perpustakaan)
│   └── db_perpustakaan.sql       # File dump SQL lengkap (struktur 9 tabel + data awal siap impor)
├── public/                       # Frontend Aplikasi (HTML5, Tailwind CSS, Vanilla JS)
│   ├── index.html                # Halaman Utama (Beranda, Katalog, Login/Register/Reset, Dashboard Admin)
│   ├── css/
│   │   └── style.css             # Styling kustom Navy Blue & White + Print Layout Laporan
│   ├── js/
│   │   └── app.js                # Logika frontend, Fetch API + CSRF token, Chart.js, Ekspor PDF & Excel
│   └── uploads/
│       └── covers/               # Penyimpanan file cover buku yang diunggah Admin
├── server/                       # Backend Node.js + Express
│   ├── server.js                 # Entrypoint server HTTP & middleware keamanan
│   ├── db.js                     # Konfigurasi koneksi pool MySQL (`db_perpustakaan`)
│   ├── lib/
│   │   ├── password.js           # Hashing password aman berbasis crypto.scrypt + timingSafeEqual
│   │   ├── security.js           # Session HMAC, proteksi CSRF token, validasi input & RBAC
│   │   └── activity.js           # Pencatatan otomatis ke tabel `aktivitas`
│   ├── routes/
│   │   ├── auth.js               # Login, Registrasi Anggota, Logout, Lupa Password
│   │   ├── books.js              # Katalog, Filter, Detail, CRUD Buku, Upload Cover, Kelola Stok
│   │   ├── members.js            # CRUD Manajemen Anggota Perpustakaan
│   │   ├── loans.js              # Pengajuan Pinjam, Setujui/Tolak, Pengembalian & Denda Otomatis
│   │   └── admin.js              # Statistik Dashboard, Grafik, Log Aktivitas, & 6 Endpoint Laporan
│   └── scripts/
│       └── setup-db.js           # Script otomatis pembuatan database, 9 tabel, cover SVG, dan seed data
├── .env                          # Konfigurasi aktif koneksi MySQL & tarif denda
├── .env.example                  # Contoh konfigurasi environment
└── package.json                  # Daftar dependensi & npm scripts
```

---

## 3. Skema Database MySQL (`db_perpustakaan`)

Database **`db_perpustakaan`** terdiri dari **9 tabel** yang saling berelasi menggunakan **Primary Key** dan **Foreign Key**:

1. **`users`** — Menyimpan akun login Admin dan Anggota (`id_user`, `nama`, `email`, `password_hash`, `role`, `reset_token`, `status_aktif`).
2. **`anggota`** — Menyimpan profil lengkap anggota (`id_anggota`, `id_user` [FK], `kode_anggota`, `nama_lengkap`, `nis_nim`, `email`, `no_telepon`, `alamat`, `tanggal_pendaftaran`).
3. **`kategori`** — Menyimpan kategori buku (`id_kategori`, `nama_kategori`, `slug`, `deskripsi`).
4. **`buku`** — Menyimpan koleksi buku (`id_buku`, `kode_buku`, `isbn`, `judul_buku`, `penulis`, `penerbit`, `tahun_terbit`, `id_kategori` [FK], `deskripsi`, `lokasi_rak`, `jumlah_stok`, `total_koleksi`, `cover_buku`).
5. **`peminjaman`** — Menyimpan transaksi induk peminjaman (`id_peminjaman`, `kode_peminjaman`, `id_anggota` [FK], `tanggal_pengajuan`, `tanggal_pinjam`, `batas_pengembalian`, `status` [`Menunggu`, `Dipinjam`, `Dikembalikan`, `Ditolak`, `Terlambat`], `disetujui_oleh` [FK]).
6. **`detail_peminjaman`** — Menyimpan rincian buku yang dipinjam (`id_detail`, `id_peminjaman` [FK], `id_buku` [FK], `jumlah`, `kondisi_pinjam`).
7. **`pengembalian`** — Menyimpan realisasi pengembalian buku (`id_pengembalian`, `id_peminjaman` [FK], `tanggal_kembali`, `terlambat_hari`, `kondisi_kembali`, `diproses_oleh` [FK]).
8. **`denda`** — Menyimpan perhitungan denda keterlambatan otomatis (`id_denda`, `id_pengembalian` [FK], `id_peminjaman` [FK], `jumlah_hari`, `tarif_per_hari`, `total_denda`, `status_bayar`).
9. **`aktivitas`** — Menyimpan riwayat aktivitas seluruh pengguna (`id_aktivitas`, `id_user` [FK], `peran`, `aksi`, `keterangan`, `ip_address`, `created_at`).

---

## 4. Panduan Instalasi & Menjalankan di Localhost (XAMPP)

### Langkah 1: Aktifkan MySQL di XAMPP
1. Buka **XAMPP Control Panel**.
2. Klik tombol **Start** pada modul **MySQL** (dan **Apache** bila ingin membuka *phpMyAdmin*).
3. Pastikan port MySQL berjalan di `3306`.

### Langkah 2: Konfigurasi Koneksi Database (`.env`)
Pastikan file `.env` di folder proyek berisi pengaturan berikut (sesuai default XAMPP):
```env
PORT=3000
DB_HOST=127.0.0.1
DB_PORT=3306
DB_USER=root
DB_PASSWORD=
DB_NAME=db_perpustakaan
SESSION_SECRET=bookbase_navy_secret_key_2026_perpustakaan_digital
FINE_PER_DAY=2000
DEFAULT_LOAN_DAYS=7
```

### Langkah 3: Buat Database & Tabel (Otomatis atau via phpMyAdmin)
- **Cara Otomatis (Direkomendasikan):**
  Jalankan perintah berikut di terminal:
  ```bash
  npm install
  npm run db:setup
  ```
  Perintah ini otomatis membuat database `db_perpustakaan`, 9 tabel beserta relasinya, akun Admin & Anggota, serta 12 buku awal.

- **Cara Manual via phpMyAdmin:**
  1. Buka `http://localhost/phpmyadmin`.
  2. Klik menu **Import** lalu pilih file `database/db_perpustakaan.sql` (atau `database/schema.sql`).
  3. Klik **Go / Kirim**.

### Langkah 4: Jalankan Server Aplikasi
```bash
npm start
```
Buka browser dan akses: **`http://localhost:3000`**
