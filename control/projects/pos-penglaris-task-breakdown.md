# Task Breakdown — POS Penglaris

- **Tanggal**: 2026-08-12
- **Project ID**: pos-penglaris
- **Sumber**: `control/backlog/pos-penglaris.md`
- **Status**: Analisis (belum ada task contract; `control/tasks/specifications/`
  adalah domain TL/SA)

## Aturan Umum

- Setiap task: satu jenis task, satu role engineer, satu repo target, satu branch
  `agent/<TASK-ID>-<slug>` dengan base `develop`.
- Merge ke `develop`; promosi `main` oleh manusia.
- Nama task sementara (TASK-ID) diusulkan PM. TL/SA yang menyetujui dan menulis
  contract final.

## Konvensi ID Task

| Prefiks | Jenis |
|---|---|
| `CONTRACT-` | Penulisan API contract |
| `BE-` | Backend (Go) |
| `FE-` | Frontend mobile (Flutter/Android) |
| `IOS-` | iOS (Swift) |
| `QA-` | Quality assurance |

## Fase 1 — MVP Backup (7 task)

Dependency chain: CONTRACT → BE (table → POST → GET) paralel dengan FE design
sempit → QA.

### 1. CONTRACT-301 — API Contract Backup

| Field | Nilai |
|---|---|
| Jenis task | contract-change |
| Role | technical-lead-system-analyst |
| Repo target | control (contracts/CONTRACT-*.yaml) |
| Dependency | — (pertama) |
| BL-ID | BL-BE-001 |
| Deskripsi | Skema endpoint POST /api/v1/backup (kirim batch transaksi + start/end date + id user) dan GET /api/v1/backup (restore, param wajib id + start/end date). Response/error schema, format tanggal, batas batch. |
| Output | contracts/CONTRACT-301.yaml |

### 2. BE-301 — Backend: Database Schema Backup

| Field | Nilai |
|---|---|
| Jenis task | backend-implementation |
| Role | backend-engineer |
| Repo target | POS-backend |
| Dependency | CONTRACT-301 |
| BL-ID | BL-BE-002 |
| Deskripsi | Migration PostgreSQL: table `users` (device UUID PK), `transactions`, `backups` (rekaman per rentang start/end). |
| Output | code, unit-tests |

### 3. BE-302 — Backend: POST Backup (kirim + simpan + ack)

| Field | Nilai |
|---|---|
| Jenis task | backend-implementation |
| Role | backend-engineer |
| Repo target | POS-backend |
| Dependency | BE-301 |
| BL-ID | BL-BE-003, BL-BE-005 |
| Deskripsi | Handler POST /api/v1/backup: validasi id (UUID), validasi rentang tanggal, simpan batch transaksi terstruktur, ack per batch. |
| Output | code, unit-tests |

### 4. BE-303 — Backend: GET Backup (restore data)

| Field | Nilai |
|---|---|
| Jenis task | backend-implementation |
| Role | backend-engineer |
| Repo target | POS-backend |
| Dependency | BE-301 |
| BL-ID | BL-BE-004, BL-BE-005 |
| Deskripsi | Handler GET /api/v1/backup: restore transaksi dengan param wajib id + start/end date. |
| Output | code, unit-tests |

### 5. FE-301 — Flutter: Fitur Backup

| Field | Nilai |
|---|---|
| Jenis task | frontend-implementation (Flutter/Android) |
| Role | android-developer |
| Repo target | POS-android |
| Dependency | CONTRACT-301 |
| BL-ID | BL-AN-001 s.d. BL-AN-006 |
| Deskripsi | Generate & simpan UUID first run; halaman backup + ukuran storage; button "backup data" + bottomsheet rentang tanggal; kirim per-batch + hapus batch setelah ack; tampilan error existing (No connection / Error state); restore via GET. |
| Output | code, unit-tests, PR |

### 6. IOS-301 — iOS Swift: Fitur Backup

| Field | Nilai |
|---|---|
| Jenis task | mobile-implementation (iOS/Swift) |
| Role | ios-developer |
| Repo target | POS-iOS |
| Dependency | CONTRACT-301 |
| BL-ID | BL-IO-001 s.d. BL-IO-006 |
| Deskripsi | Sama dengan FE-301 pada platform iOS native Swift: UUID, halaman backup, bottomsheet tanggal, kirim/hapus per-batch, error state, restore. |
| Output | code, unit-tests, PR |

### 7. QA-301 — QA End-to-End Backup

| Field | Nilai |
|---|---|
| Jenis task | qa |
| Role | qa-engineer |
| Repo target | POS-backend (integration) + POS-android/POS-iOS |
| Dependency | BE-302, BE-303, FE-301, IOS-301 |
| BL-ID | BL-QA-001 |
| Deskripsi | Skenario E2E: kirim batch → ack → hapus device → restore; offline/failure tidak kehilangan data. |
| Output | QA report per contract |

### Ringkasan Fase 1

| TASK-ID | Jenis | Role | Repo | Dependency |
|---|---|---|---|---|
| CONTRACT-301 | contract-change | TL/SA | control | — |
| BE-301 | backend-implementation | backend-engineer | POS-backend | CONTRACT-301 |
| BE-302 | backend-implementation | backend-engineer | POS-backend | BE-301 |
| BE-303 | backend-implementation | backend-engineer | POS-backend | BE-301 |
| FE-301 | frontend-implementation | android-developer | POS-android | CONTRACT-301 |
| IOS-301 | mobile-implementation | ios-developer | POS-iOS | CONTRACT-301 |
| QA-301 | qa | qa-engineer | POS-backend / mobile | BE-302, BE-303, FE-301, IOS-301 |

## Fase 2 — Aplikasi POS Offline (belum di-breakdown)

Cakupan: auth email+OTP, manajemen produk/kategori, transaksi POS lengkap,
laporan, profil, error states (REQ-POS-020 s.d. 029, BL-BE-006 s.d. BL-QA-002).

- Dependency utama Fase 2: aplikasi offline penuh berjalan di atas struktur data
  lokal yang sama dengan yang dipakai backup Fase 1 (skema transaksi device).
- Keputusan desain dari Fase 1 (format payload, skema transaksi) menjadi fondasi
  Fase 2; perubahan skema di kemudian hari mahal → perlu konfirmasi open question
  OQ-03 sebelum Fase 2 dimulai.

## Catatan untuk TL/SA

1. TASK-ID `CONTRACT-301/BE-30x/FE-301/IOS-301/QA-301` usulan; silakan finalisasi
   saat menulis contract.
2. Open question OQ-01 s.d. OQ-06 di dokumen requirements perlu dijawab sebelum
   contract Fase 1 difinalkan.
3. Dependency antar task Fase 1 sudah dipetakan; urutan eksekusi paralel:
   CONTRACT-301 → {BE-301 → BE-302, BE-303} ∥ {FE-301, IOS-301} → QA-301.
