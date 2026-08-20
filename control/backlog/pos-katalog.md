# Backlog — POS Katalog (FC-304)

**Tanggal**: 2026-08-17
**Status**: Draft
**Dikembangkan oleh**: Project Manager

## Ringkasan

Daftar item kerja untuk fitur katalog produk POS Katalog. Fitur ini memungkinkan pengguna POS untuk melihat, mengelola, dan mencari data produk secara terstruktur di platform Android (Flutter) dan iOS (SwiftUI).

## Item Kerja yang Diperlukan

### Backend (Go) — 5 Task

| Task ID | Tipe Task | Peran | Repo Target | Dependency |
|---|---|---|---|---|
| BE-KAT-001 | database-schema | technical-lead-system-analyst | m2s-vsh-project-backend | - |
| BE-KAT-002 | api-endpoint-crud-produk | backend-engineer | m2s-vsh-project-backend | BE-KAT-001 |
| BE-KAT-003 | api-endpoint-pencarian | backend-engineer | m2s-vsh-project-backend | BE-KAT-001 |
| BE-KAT-004 | api-endpoint-kategori | backend-engineer | m2s-vsh-project-backend | - |
| BE-KAT-005 | database-migration | backend-engineer | m2s-vsh-project-backend | BE-KAT-001 |

#### Detail Backend Tasks

1. **BE-KAT-001 — Database Schema Setup**
   - Buat tabel `products` (id, nama, harga_beli, harga_jual, stok, kategori_id, foto_url, created_at, updated_at)
   - Buat tabel `categories` (id, nama, created_at, updated_at)
   - Tambahkan foreign key dari products ke categories
   - Buat index untuk pencarian produk

2. **BE-KAT-002 — Endpoint CRUD Produk**
   - POST /api/v1/products — buat produk baru
   - GET /api/v1/products — daftar semua produk
   - GET /api/v1/products/:id — detail produk
   - PUT /api/v1/products/:id — update produk
   - DELETE /api/v1/products/:id — hapus produk

3. **BE-KAT-003 — Endpoint Pencarian & Filter**
   - GET /api/v1/products/search?q=<keyword> — pencarian by nama
   - GET /api/v1/products?category=<id> — filter by kategori

4. **BE-KAT-004 — Endpoint Manajemen Kategori**
   - POST /api/v1/categories — buat kategori baru
   - GET /api/v1/categories — daftar semua kategori
   - PUT /api/v1/categories/:id — update kategori
   - DELETE /api/v1/categories/:id — hapus kategori (hanya jika tidak ada produk terkait)

5. **BE-KAT-005 — Database Migration**
   - Buat migration file untuk setup schema
   - Seed initial categories (opsional)

### Flutter Android (Dart) — 4 Task

| Task ID | Tipe Task | Peran | Repo Target | Dependency |
|---|---|---|---|---|
| FE-KAT-001 | ui-katalog-list | flutter-engineer | m2s-vsh-project-frontend | - |
| FE-KAT-002 | ui-detail-produk | flutter-engineer | m2s-vsh-project-frontend | FE-KAT-001 |
| FE-KAT-003 | form-crud-produk | flutter-engineer | m2s-vsh-project-frontend | FE-KAT-001 |
| FE-KAT-004 | pencarian-filter | flutter-engineer | m2s-vsh-project-frontend | BE-KAT-003 |

#### Detail Flutter Tasks

1. **FE-KAT-001 — Tampilan Daftar Katalog**
   - Halaman katalog dengan ListView produk
   - Setiap item menampilkan: foto, nama, harga jual, stok
   - Empty state saat tidak ada produk
   - Refresh indicator untuk pull-to-refresh

2. **FE-KAT-002 — Detail Produk**
   - Halaman detail produk
   - Tampilkan semua informasi produk
   - Tombol edit/hapus
   - Navigasi kembali ke daftar

3. **FE-KAT-003 — Form CRUD Produk**
   - Form tambah produk dengan validasi
   - Form edit produk (isii otomatis)
   - Pilihan kategori dari dropdown
   - Upload foto (opsional)
   - Submit ke API backend

4. **FE-KAT-004 — Fitur Pencarian & Filter**
   - Search bar untuk pencarian produk
   - Filter dropdown kategori
   - Sorting options (nama, harga, stok)
   - Combine search + filter

### iOS (SwiftUI) — 4 Task

| Task ID | Tipe Task | Peran | Repo Target | Dependency |
|---|---|---|---|---|
| IOS-KAT-001 | ui-katalog-list | ios-developer | m2s-vsh-project-ios | - |
| IOS-KAT-002 | ui-detail-produk | ios-developer | m2s-vsh-project-ios | IOS-KAT-001 |
| IOS-KAT-003 | form-crud-produk | ios-developer | m2s-vsh-project-ios | IOS-KAT-001 |
| IOS-KAT-004 | pencarian-filter | ios-developer | m2s-vsh-project-ios | BE-KAT-003 |

#### Detail iOS Tasks

1. **IOS-KAT-001 — Tampilan Daftar Katalog**
   - Halaman katalog dengan List produk
   - Setiap item menampilkan: foto, nama, harga jual, stok
   - Empty state saat tidak ada produk
   - Pull-to-refresh

2. **IOS-KAT-002 — Detail Produk**
   - Halaman detail produk
   - Tampilkan semua informasi produk
   - Tombol edit/hapus
   - Navigasi kembali ke daftar

3. **IOS-KAT-003 — Form CRUD Produk**
   - Form tambah produk dengan validasi
   - Form edit produk (isii otomatis)
   - Pilihan kategori dari Picker
   - Upload foto (opsional)
   - Submit ke API backend

4. **IOS-KAT-004 — Fitur Pencarian & Filter**
   - Search bar untuk pencarian produk
   - Filter Picker kategori
   - Sortir options (nama, harga, stok)
   - Combine search + filter

### API Contract (Control Repo) — 1 Task

| Task ID | Tipe Task | Peran | Repo Target | Dependency |
|---|---|---|---|---|
| CONTRACT-KAT-001 | api-contract | technical-lead-system-analyst | POS-m2s-vsh-platform | - |

#### Detail Contract Task

1. **CONTRACT-KAT-001 — API Contract Katalog**
   - Definisikan endpoint API untuk produk dan kategori
   - Spesifikasi request/response JSON
   - Error handling konsisten
   - Validasi input di backend

### Testing & QA — 3 Task

| Task ID | Tipe Task | Peran | Repo Target | Dependency |
|---|---|---|---|---|
| QA-KAT-001 | unit-test-backend | qa-engineer | m2s-vsh-project-backend | BE-KAT-005 |
| QA-KAT-002 | integration-test-api | qa-engineer | m2s-vsh-project-backend | BE-KAT-002, BE-KAT-003, BE-KAT-004 |
| QA-KAT-003 | e2e-test-mobile | qa-engineer | m2s-vsh-project-frontend, m2s-vsh-project-ios | FE-KAT-004, IOS-KAT-004 |

## Dependency Graph

```
CONTRACT-KAT-001 (API Contract)
         |
    BE-KAT-001 (DB Schema)
         |-----------------
         |                 \
    BE-KAT-002, BE-KAT-003, BE-KAT-004 (Endpoints)
         |                 |
    BE-KAT-005 (Migration)  |
                            |
               FE-KAT-001, FE-KAT-002, FE-KAT-003, FE-KAT-004 (Flutter)
                            |
                     QA-KAT-001, QA-KAT-002 (Backend Testing)
                            |
                     QA-KAT-003 (E2E Mobile Testing)
```

## Timeline Estimasi

- Backend: 3-5 hari kerja
- Flutter: 4-6 hari kerja  
- iOS: 4-6 hari kerja
- Contract & Review: 1-2 hari
- Testing: 2-3 hari

**Total estimasi**: 10-15 hari kerja (dikerjakan secara paralel)

## Prioritas

Medium — Fitur ini diperlukan untuk lengkapnya ekosistem POS Penglaris, namun tidak segera mendesak. Bisa dikerjakan setelah fitur backup (CONTRACT-301) selesai.
