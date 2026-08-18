# Requirements — POS Katalog (FC-304)

**Tanggal**: 2026-08-17
**Status**: Draft (analisis Project Manager)
**Project ID**: pos-katalog
**Fase**: Fase 1 (fitur katalog produk)
**Penulis**: Project Manager

## 1. Ringkasan Eksekutif

FC-304 "POS Katalog" adalah fitur Penggunaan katalog untuk sistem POS Penglaris. Fitur ini memungkinkan pengguna POS untuk melihat, mengelola, dan mencari data produk secara terstruktur. Fitur ini berbeda dari backup transaksi (CONTRACT-301) yang fokus pada penyimpanan data transaksi ke server.

## 2. Stakeholder

| Stakeholder | Peran | Kebutuhan |
|---|---|---|
| Pemilik usaha mikro/UMKM (end user) | Pengguna POS | Akses mudah ke data produk, pencarian cepat, manajemen stok |
| User (klien) | Pemberi brief | Produk sesuai kebutuhan katalog produk yang efisien |
| Project Manager | Otoritas scope & priority | Requirements, backlog, task breakdown |
| TL/SA | Analisis teknis & kontrak | Requirement yang jelas untuk diturunkan jadi task contract |
| UI/UX Designer | Desain | Tampilan katalog yang intuitif dan responsif |
| Backend Engineer | Implementasi API | Endpoint CRUD produk + pencarian |
| Flutter/Android Engineer | Implementasi FE | Tampilan katalog di Flutter Android |
| iOS Engineer | Implementasi FE | Tampilan katalog di SwiftUI iOS |
| QA Engineer | Verifikasi acceptance criteria | Kriteria terukur per requirement |

## 3. Masalah Utama

- Pengguna POS memerlukan akses cepat ke data produk yang tersedia
- Pencarian produk perlu efisien, terutama dengan banyak data
- Data produk perlu dapat dikelola (tambah, ubah, hapus, urutan)
- Stok produk perlu dapat dilihat dan dikelola dari katalog
- Kategori produk perlu terstruktur untuk navigasi yang Mudah

## 4. Tujuan Bisnis

1. Menyediakan katalog produk yang mudah dilihat dan dicari
2. Memungkinkan manajemen data produk (CRUD) dari aplikasi POS
3. Memungkinkan filter/pencarian produk berdasarkan kategori
4. Menyajikan data stok yang up-to-date
5. Konsisten di kedua platform mobile (Flutter Android & SwiftUI iOS)

## 5. Scope dan Out-of-Scope

### 5.1 Scope (Yang Termasuk)

- Tampilan daftar produk dengan foto, nama, harga, dan stok tersisa
- Fitur pencarian produk berdasarkan nama
- Fitur filter produk per kategori
- Halaman detail produk (nama, harga, stok, kategori, deskripsi)
- Form tambah/edit produk dengan validasi
- Daftar kategori produk yang terdaftar
- Fungsi urutkan produk (per nama, per harga, per stok)
- API backend untuk CRUD produk
- Sinkronisasi data produk lokal ↔ server
- Dukungan offline untuk melihat data produk yang sudah ada

### 5.2 Out-of-Scope (Yang Tidak Termasuk)

- Pembayaran atau transaksi - fokus hanya pada manajemen produk
- Import/export data produk (batch operation)
- Gambar produk yang sudah ada di folder asset - hanya URL/filename
- Barcode/QR code scanning
- Integrasi dengan supplier/API eksternal untuk update stok otomatis
- Firmware printer atau hardware POS lain
- Multi-grosir / varian produk (satu SKU = satu produk)

## 6. Requirements Detail

Format: `REQ-POS-KAT-NNN — <judul>`. Status default `open`.

### 6.1 Manajemen Produk

- REQ-POS-KAT-001 — Terdapat halaman katalog yang menampilkan daftar produk dengan foto, nama, harga, dan stok tersisa.
- REQ-POS-KAT-002 — Pengguna dapat menambahkan produk baru melalui form dengan field: nama produk, harga beli, harga jual, stok, kategori, dan foto.
- REQ-POS-KAT-003 — Pengguna dapat mengedit data produk yang ada (nama, harga, stok, kategori, foto).
- REQ-POS-KAT-004 — Pengguna dapat menghapus produk dari katalog yang ada.
- REQ-POS-KAT-005 — Setiap produk memiliki kategori yang dapat dipilih dari daftar atau ditambahkan baru.

### 6.2 Pencarian & Filter

- REQ-POS-KAT-006 — Terdapat field pencarian untuk memfilter daftar produk berdasarkan nama.
- REQ-POS-KAT-007 — Daftar produk dapat difilter berdasarkan kategori yang tersedia.

### 6.3 Urutan & Pengaturan

- REQ-POS-KAT-008 — Daftar produk dapat diurutkan berdasarkan: nama (A-Z/Z-A), harga (terendah/termahal), stok (terbanyak/terkecil).
- REQ-POS-KAT-009 — Pengguna dapat menambahkan kategori baru jika tidak ada yang cocok.

### 6.4 Tampilan & Pengalaman Pengguna

- REQ-POS-KAT-010 — Halaman katalog menampilkan state kosong (empty state) saat tidak ada produk.
- REQ-POS-KAT-011 — Tampilan produk responsif di kedua platform (Flutter Android & SwiftUI iOS).
- REQ-POS-KAT-012 — Informasi stok produk selalu terlihat pada setiap item di daftar.

### 6.5 Backend & API

- REQ-POS-KAT-013 — Backend menyediakan endpoint CRUD produk lengkap (Create, Read, Update, Delete).
- REQ-POS-KAT-014 — Backend menyediakan endpoint pencarian/filter produk berdasarkan nama/kategori.
- REQ-POS-KAT-015 — Backend menyediakan endpoint manajemen kategori (list, create, update, delete).
- REQ-POS-KAT-016 — API menggunakan format JSON standar dengan response yang konsisten.

### 6.6 Offline Support

- REQ-POS-KAT-017 — Data produk yang sudah ada dapat dilihat offline.
- REQ-POS-KAT-018 — Perubahan produk yang dibuat offline disimpan secara lokal dan dikirimkan saat online.

## 7. Asumsi

- A-01: Produk hanya memiliki satu kategori utama (bukan multi-kategori).
- A-02: Harga beli dan harga jual adalah nilai tetap (bukan varian/harga ganda).
- A-03: Stok adalah jumlah numerik yang dapat diedit manual.
- A-04: Foto produk disimpan sebagai URL/filename yang sudah ada di asset.
- A-05: Data produk disimpan di tabel `products` dengan foreign key ke `categories`.

## 8. Open Question

- OQ-01: Apakah ada batasan maksimal jumlah produk per katalog?
- OQ-02: Format foto produk yang disuk? (base64, URL eksternal, atau local path)
- OQ-03: Apakah perlu fitur import massal produk (CSV)?
- OQ-04: Apakah ada requirement untuk menampilkan margin keuntungan (harga jual - harga beli)?

## 9. Open Questions yang Perlu Dijawab User

- OQ-BIZ-001: Apakah fitur ini bersifat wajib (mandatory) atau opsional untuk Fase 1?
- OQ-BIZ-002: Apakah ada batasan kuota produk yang boleh ditambahkan pengguna?
- OQ-BIZ-003: Apakah ada requirement khusus untuk tampilan di mode gelap/dark?

## 10. Definition of Done (Requirements)

- Seluruh requirement dapat dipahami TL/SA.
- Scope dan out-of-scope eksplisit.
- Owner dan dependency antar task jelas (lihat task breakdown).
- Open question bisnis tuntas atau tercatat.
- TL/SA menerima input yang cukup untuk menyusun task contract.

## 6. Requirement IDs (untuk task contract)

- REQ-POS-KAT-001: Database schema produk dan kategori
- REQ-POS-KAT-002: Unit test repository (tables, indexes)
- REQ-POS-KAT-003: API endpoint CRUD produk
- REQ-POS-KAT-004: API endpoint CRUD kategori + migration
- REQ-POS-KAT-005: Tampilan daftar katalog Flutter
- REQ-POS-KAT-006: Tampilan daftar katalog iOS
- REQ-POS-KAT-007: Detail produk Flutter
- REQ-POS-KAT-008: Form CRUD produk Flutter/iOS
- REQ-POS-KAT-009: Pencarian & filter produk Flutter/iOS

## 7. Business Rules

- BL-BE-KAT-001: Foreign key products.kategori_id -> categories.id ON DELETE SET NULL
- BL-FE-KAT-001: Consistent UI patterns antara Flutter dan iOS
- BL-IOS-KAT-001: Same as BL-FE-KAT-001
