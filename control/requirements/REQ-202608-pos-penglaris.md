# Requirements — POS Penglaris

- **Tanggal**: 2026-08-12
- **Status**: Draft (analisis Project Manager)
- **Project ID**: pos-penglaris
- **Fase**: Fase 1 (fitur backup) → Fase 2 (aplikasi offline penuh)
- **Penulis**: Project Manager

## 1. Ringkasan Eksekutif

Aplikasi POS mobile sederhana, berjalan hybrid (mostly offline, online hanya
untuk fitur backup data). Target platform: Android (Flutter) dan iOS (Swift).
Backend: Go dengan PostgreSQL. Seluruh alur fitur tergambar pada Figma (46
screen, frame 360×800, bahasa Indonesia). Fitur backup belum ada di Figma dan
menjadi prioritas Fase 1.

Fase 1 (MVP): fitur backup data transaksi ke server dengan auth anonymous
device ID. Fase 2: keseluruhan aplikasi POS offline.

## 2. Stakeholder

| Stakeholder | Peran | Kebutuhan |
|---|---|---|
| Pemilik usaha mikro/UMKM (end user) | Pengguna akhir | POS sederhana, bisa dipakai offline, data aman via backup |
| User (klien) | Pemberi brief | Produk sesuai Figma + fitur backup |
| Project Manager | Otoritas scope & priority | Requirements, backlog, task breakdown |
| TL/SA | Analisis teknis & kontrak | Requirement yang jelas untuk diturunkan jadi task contract |
| UI/UX Designer | Desain | Layar backup memakai screen existing (no new Figma screen) |
| Backend Engineer | Implementasi API | Table users, transactions, backups + endpoint |
| Flutter/Android & iOS Engineer | Implementasi FE | Halaman backup + konsumsi API |
| QA Engineer | Verifikasi acceptance criteria | Kriteria terukur per requirement |

## 3. Masalah Utama

- POS harus tetap berfungsi tanpa internet (offline-first) — jualan tidak boleh
  berhenti karena koneksi hilang.
- Data transaksi tersimpan di perangkat; tanpa backup, data hilang bila
  perangkat rusak, hilang, atau diganti.
- Aplikasi hybrid (offline dominan) menyulitkan sinkronisasi data ke server.
- Tidak ada mekanisme pemulihan data transaksi setelah perangkat hilang/rusak.
- User tidak dapat melihat data transaksi lama dari perangkat/perangkat lain.

## 4. Tujuan Bisnis

1. Menyediakan POS mobile sederhana yang dapat dipakai sepenuhnya offline.
2. Menjamin data transaksi aman melalui mekanisme backup berkala ke server.
3. Menyediakan cara pemulihan (restore) data transaksi dari server.
4. Menjaga biaya dan kompleksitas backend tetap rendah (auth tanpa akun/password).
5. Menghasilkan produk siap rilis dengan dua codebase mobile native
   (Flutter/Android, Swift/iOS) dan satu backend bersama.

## 5. Scope

### 5.1 Scope keseluruhan produk (Fase 1 + Fase 2)

Aplikasi POS mobile hybrid dengan fitur:

1. **Auth & Onboarding** — Registrasi Akun (email → OTP → sinkronasi data),
   Update Page, Cantumkan nama usaha.
2. **Manajemen Produk** — List produk (dengan empty state), Tambah/Ubah/Hapus
   produk, Detail produk, Atur ketersediaan stok, Urutkan/Ubah/Hapus kategori.
3. **Transaksi POS** — List belum bayar, Pilih produk (list + kategori), Tambah
   manual (kalkulator custom), Simpan dulu, Bayar (Tunai / Non Tunai / data
   tambahan), Detail transaksi, Cetak struk, Sukses Bayar & Simpan.
4. **Laporan** — Overview penjualan (Hari ini / Bulan ini / Tahun ini + %
   perubahan), Riwayat penjualan, Kelola stok (filter/search).
5. **Akun/Profil** — Akun Saya, Ubah Profil Toko, Ubah data usaha.
6. **Error & Empty States** — No connection, Error state.
7. **Fitur Backup (Fase 1, prioritas)** — Halaman informasi penggunaan storage,
   button "backup data" khusus data transaksi, form rentang tanggal (start/end),
   komunikasi via API backend, penghapusan data ter-backup dari perangkat,
   restore data via get dengan param id + start/end date.

### 5.2 Scope Fase 1 (MVP Backup)

- Halaman backup menampilkan ukuran storage device yang dipakai aplikasi POS.
- Button "backup data" khusus data transaksi.
- Setelah button diklik: popup/form rentang tanggal (dari s/d) data yang
  di-backup.
- Komunikasi app ↔ server via API REST.
- Data transaksi dikirim ke server lengkap dengan start date, end date, dan id
  user.
- Data yang sudah di-backup dihapus dari storage device (hard-delete bertahap
  + verifikasi: kirim per-batch, hapus batch hanya setelah ack server).
- Data dapat dilihat kembali (restore) via request get dengan parameter wajib
  id dan tanggal start & end.
- Auth backup: anonymous device ID (UUID) — app generate UUID pertama jalan,
  dikirim ke backend, dipakai sebagai user id.
- Backend: table `users` (device UUID), `transactions`, `backups` + endpoint.
- Layar backup memakai screen existing (No connection / Error state untuk
  offline/failure; bottomsheet pattern yang ada untuk form tanggal). Tidak ada
  screen Figma baru.

### 5.3 Teknologi

| Lapisan | Teknologi |
|---|---|
| Mobile Android | Flutter |
| Mobile iOS | Swift (native) |
| Backend | Go |
| Database | PostgreSQL |

## 6. Out of Scope

- Payment gateway / integrasi pembayaran online.
- Multi-perangkat sinkronisasi realtime antar perangkat pemilik yang sama.
- Fitur backup untuk data selain transaksi (produk, kategori, laporan, profil).
- Login/password tradisional pada fitur backup (menggunakan anonymous device ID).
- Backup otomatis terjadwal (hanya backup manual via tombol).
- Enkripsi data transaksi pada device (di luar scope fase 1).
- Versi web/desktop POS.
- Notifikasi push.

## 7. Requirements Detail

> Format: `REQ-POS-NNN — <judul>`. Status default `open`.

### Fase 1 — Backup (prioritas)

- REQ-POS-001 — Aplikasi POS berjalan hybrid: seluruh fitur utama dapat dipakai
  offline penuh; koneksi internet hanya dibutuhkan untuk fitur backup.
- REQ-POS-002 — Pada first run, aplikasi generate anonymous device ID berupa
  UUID, menyimpannya secara lokal, dan memakainya sebagai user id untuk semua
  request backup.
- REQ-POS-003 — Terdapat halaman backup yang menampilkan besaran storage device
  yang digunakan aplikasi POS ini.
- REQ-POS-004 — Terdapat button "backup data" yang khusus mem-backup data
  transaksi.
- REQ-POS-005 — Setelah button diklik, muncul popup/form pemilihan rentang
  tanggal (tanggal mulai sampai tanggal selesai) data yang akan di-backup ke
  server.
- REQ-POS-006 — Komunikasi app-server memakai API backend REST.
- REQ-POS-007 — Payload backup berisi data transaksi lengkap dengan start date,
  end date, dan id user (device UUID).
- REQ-POS-008 — Setelah backup sukses, data transaksi yang ter-backup dihapus
  dari storage device dan hanya tersimpan di server.
- REQ-POS-009 — Penghapusan data device dilakukan hard-delete bertahap:
  transaksi dikirim per-batch; batch dihapus hanya setelah mendapat ack server.
- REQ-POS-010 — User dapat melihat kembali data transaksi via request get data
  dengan parameter wajib id (device UUID) dan rentang tanggal start & end.
- REQ-POS-011 — Backend menyimpan data terstruktur: table users (device UUID),
  table transactions, dan table backups (rekaman riwayat backup per rentang).
- REQ-POS-012 — Layar backup tidak membuat screen Figma baru; memakai screen
  error existing (No connection / Error state) untuk kondisi offline/failure dan
  bottomsheet pattern existing untuk form tanggal.
- REQ-POS-013 — Kondisi tanpa koneksi saat backup ditangani dengan tampilan
  error existing dan tidak terjadi kehilangan data (data tetap utuh di device
  sampai backup berhasil).

### Fase 2 — Aplikasi POS offline (keseluruhan)

- REQ-POS-020 — Auth flow: registrasi akun dengan email → verifikasi OTP →
  sinkronasi data.
- REQ-POS-021 — Halaman Update Page dan Cantumkan nama usaha setelah registrasi.
- REQ-POS-022 — Manajemen produk: list (dengan empty state), tambah, ubah,
  hapus, detail produk, atur ketersediaan stok.
- REQ-POS-023 — Manajemen kategori: urutkan, ubah, hapus kategori.
- REQ-POS-024 — Transaksi: list belum bayar, pilih produk (list + kategori),
  tambah manual via kalkulator custom, simpan dulu.
- REQ-POS-025 — Pembayaran: metode Tunai / Non Tunai, data tambahan, detail
  transaksi, cetak struk, halaman sukses bayar & simpan.
- REQ-POS-026 — Laporan: overview penjualan (Hari ini / Bulan ini / Tahun ini)
  dengan persentase perubahan.
- REQ-POS-027 — Riwayat penjualan dan kelola stok (filter/search).
- REQ-POS-028 — Profil: Akun Saya, Ubah Profil Toko, Ubah data usaha.
- REQ-POS-029 — Error states: No connection dan Error state konsisten di
  seluruh aplikasi.

## 8. Asumsi, Open Question, Risiko, Keputusan

### Asumsi

- A-01: Satuan waktu rentang backup adalah tanggal (hari), diukur di zona waktu
  lokal device.
- A-02: "Storage device yang digunakan aplikasi" dihitung dari ukuran data
  lokal milik aplikasi (DB/penyimpanan lokal), bukan total storage perangkat.
- A-03: Backup bersifat one-way (device → server). Restore ditarik via endpoint
  get tanpa menghapus data server.
- A-04: Penggantian perangkat di luar scope Fase 1 (belum dibahas mekanisme
  pindah device UUID).

### Open Question

- OQ-01: Bagaimana cara user mengetahui/mentransfer device UUID saat pindah
  perangkat? (belum dijawab — memengaruhi restore di perangkat baru)
- OQ-02: Apakah restore menghapus data server (one-time restore) atau tetap
  tersedia (bisa di-restore berulang)?
- OQ-03: Format data transaksi yang di-backup — hanya tabel transaksi mentah
  atau termasuk referensi produk/kategori/metode bayar? (perlu keputusan TL/SA)
- OQ-04: Batas ukuran payload / jumlah transaksi per satu request backup?
- OQ-05: Kapan data dinyatakan "sukses backup" di sisi device — seluruh batch
  ter-ack atau per batch?
- OQ-06: Nama brand/format UI halaman backup diikuti gaya Figma (DM Sans,
  bahasa Indonesia) tanpa screen baru — perlu konfirmasi pola bottomsheet mana
  yang dijadikan basis.

### Risiko

- R-01: Hard-delete tanpa ack server → kehilangan data permanen. Dimitigasi
  REQ-POS-009 (hapus per-batch setelah ack).
- R-02: Device UUID hilang → data tidak bisa direstore. Dimitigasi: simpan
  UUID lokal dengan ekspor/cadangan dipertimbangkan Fase 2.
- R-03: Dua codebase mobile (Flutter + Swift) → biaya pengembangan dan
  pemeliharaan ganda. Dimitigasi: backend tunggal, API contract sama.

### Keputusan (disepakati user)

- D-01: Auth backup memakai anonymous device ID (UUID), bukan akun/password.
- D-02: Delete data device setelah backup = hard-delete bertahap + verifikasi.
- D-03: Layar backup tidak membuat screen Figma baru — pakai screen error
  existing + bottomsheet pattern existing.
- D-04: Backend membuat table users, transactions, backups + endpoint; transaksi
  ter-backup disimpan terstruktur di server.

## 9. Definition of Done (Requirements)

- Seluruh requirement dapat dipahami TL/SA.
- Scope dan out-of-scope eksplisit.
- Owner dan dependency antar task jelas (lihat task breakdown).
- Open question bisnis tuntas atau tercatat.
- TL/SA menerima input yang cukup untuk menyusun task contract.
