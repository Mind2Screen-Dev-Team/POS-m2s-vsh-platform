# Backlog — POS Penglaris

- **Tanggal**: 2026-08-12
- **Project ID**: pos-penglaris
- **Sumber**: `control/requirements/REQ-202608-pos-penglaris.md`
- **Catatan**: Item dikelompokkan per komponen/domain. Belum ada urutan prioritas.
  Fase 1 (backup) ditandai **[F1]**, Fase 2 (aplikasi offline) **[F2]**.
  ID backlog sementara memakai prefiks `BL-`.

## Backend (Go + PostgreSQL)

| ID | Item | Fase | Requirement |
|---|---|---|---|
| BL-BE-001 | API contract backup: endpoint POST /api/v1/backup, GET /api/v1/backup, skema request/response (start date, end date, id user, batch transaksi) | [F1] | REQ-POS-006, 007, 010 |
| BL-BE-002 | Setup database PostgreSQL: migration table `users` (device UUID), `transactions`, `backups` | [F1] | REQ-POS-011 |
| BL-BE-003 | Endpoint POST /api/v1/backup — terima batch transaksi + start/end date + id user; simpan terstruktur; ack per batch | [F1] | REQ-POS-007, 009, 011 |
| BL-BE-004 | Endpoint GET /api/v1/backup — restore data transaksi dengan param wajib id + start/end date | [F1] | REQ-POS-010 |
| BL-BE-005 | (Pertimbangan) Validasi UUID device id + validasi rentang tanggal di sisi server | [F1] | REQ-POS-007, 010 |
| BL-BE-006 | (Fase 2) Auth email + OTP: registrasi, verifikasi, sinkronasi data | [F2] | REQ-POS-020 |
| BL-BE-007 | (Fase 2) Endpoint CRUD produk & kategori untuk sinkronasi | [F2] | REQ-POS-022, 023 |
| BL-BE-008 | (Fase 2) Endpoint laporan penjualan (overview, riwayat, stok) | [F2] | REQ-POS-026, 027 |

## Android (Flutter)

| ID | Item | Fase | Requirement |
|---|---|---|---|
| BL-AN-001 | Generate & simpan anonymous device ID (UUID) saat first run; dipakai sebagai user id | [F1] | REQ-POS-002 |
| BL-AN-002 | Halaman backup: tampilkan storage device yang dipakai aplikasi POS | [F1] | REQ-POS-003 |
| BL-AN-003 | Button "backup data" + popup/form rentang tanggal (start/end) via bottomsheet pattern existing | [F1] | REQ-POS-004, 005, 012 |
| BL-AN-004 | Kirim data transaksi per-batch ke server; hapus batch hanya setelah ack (hard-delete bertahap + verifikasi) | [F1] | REQ-POS-008, 009 |
| BL-AN-005 | Tampilan offline/failure pakai screen error existing (No connection / Error state); data aman sampai backup sukses | [F1] | REQ-POS-012, 013 |
| BL-AN-006 | Restore: request get data transaksi dengan id + start/end date, tampilkan hasil | [F1] | REQ-POS-010 |
| BL-AN-007 | (Fase 2) Auth flow: registrasi email → OTP → sinkronasi data, Update Page, nama usaha | [F2] | REQ-POS-020, 021 |
| BL-AN-008 | (Fase 2) Manajemen produk & kategori (list, tambah/ubah/hapus, detail, stok) | [F2] | REQ-POS-022, 023 |
| BL-AN-009 | (Fase 2) Transaksi: list belum bayar, pilih produk, kalkulator custom, simpan dulu, bayar, detail, cetak struk | [F2] | REQ-POS-024, 025 |
| BL-AN-010 | (Fase 2) Laporan: overview penjualan, riwayat, kelola stok | [F2] | REQ-POS-026, 027 |
| BL-AN-011 | (Fase 2) Profil: Akun Saya, Ubah Profil Toko, Ubah data usaha | [F2] | REQ-POS-028 |

## iOS (Swift)

| ID | Item | Fase | Requirement |
|---|---|---|---|
| BL-IO-001 | Generate & simpan anonymous device ID (UUID) saat first run; dipakai sebagai user id | [F1] | REQ-POS-002 |
| BL-IO-002 | Halaman backup: tampilkan storage device yang dipakai aplikasi POS | [F1] | REQ-POS-003 |
| BL-IO-003 | Button "backup data" + popup/form rentang tanggal (start/end) via bottomsheet pattern existing | [F1] | REQ-POS-004, 005, 012 |
| BL-IO-004 | Kirim data transaksi per-batch ke server; hapus batch hanya setelah ack (hard-delete bertahap + verifikasi) | [F1] | REQ-POS-008, 009 |
| BL-IO-005 | Tampilan offline/failure pakai screen error existing (No connection / Error state); data aman sampai backup sukses | [F1] | REQ-POS-012, 013 |
| BL-IO-006 | Restore: request get data transaksi dengan id + start/end date, tampilkan hasil | [F1] | REQ-POS-010 |
| BL-IO-007 | (Fase 2) Auth flow: registrasi email → OTP → sinkronasi data, Update Page, nama usaha | [F2] | REQ-POS-020, 021 |
| BL-IO-008 | (Fase 2) Manajemen produk & kategori | [F2] | REQ-POS-022, 023 |
| BL-IO-009 | (Fase 2) Transaksi: list belum bayar, pilih produk, kalkulator custom, bayar, detail, cetak struk | [F2] | REQ-POS-024, 025 |
| BL-IO-010 | (Fase 2) Laporan penjualan & kelola stok | [F2] | REQ-POS-026, 027 |
| BL-IO-011 | (Fase 2) Profil: Akun Saya, Ubah Profil Toko, Ubah data usaha | [F2] | REQ-POS-028 |

## Control (repo control)

| ID | Item | Fase | Requirement |
|---|---|---|---|
| BL-CT-001 | Analisis teknis TL/SA: turunkan requirements jadi task contract (CONTRACT, BE, FE, QA) | [F1] | Semua REQ-POS |
| BL-CT-002 | (Fase 2) Analisis teknis lanjutan untuk seluruh fitur offline | [F2] | REQ-POS-020 s.d. 029 |
| BL-CT-003 | (Fase 2) Dokumentasi pre-dev & SRS untuk fase offline | [F2] | REQ-POS |

## QA

| ID | Item | Fase | Requirement |
|---|---|---|---|
| BL-QA-001 | Skenario QA end-to-end backup: kirim batch, ack, hapus device, restore | [F1] | REQ-POS-006 s.d. 010, 013 |
| BL-QA-002 | (Fase 2) Skenario QA end-to-end seluruh alur POS offline | [F2] | REQ-POS-020 s.d. 029 |

## Catatan

- Backlog belum diurutkan berdasarkan prioritas; dependency akan ditetapkan di
  task breakdown (`control/projects/pos-penglaris-task-breakdown.md`).
- Fase 1 = MVP backup, harus selesai sebelum fase 2 dikerjakan.
