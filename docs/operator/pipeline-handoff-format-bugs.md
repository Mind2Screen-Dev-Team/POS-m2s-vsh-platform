# Temuan: Bug Format Handoff + Resiliensi Pipeline (5 bug interlock)

**Tanggal:** 2026-08-12
**Versi:** M2S-VSH Lite v0.1.0
**Status:** Open (menunggu fix manusia — path human-only)
**Sumber:** eksekusi pipeline fase-1 POS Penglaris (CONTRACT-301, BE-301/302/303, FE-301, IOS-301)

## Ringkasan

Pipeline berjalan untuk task app, tapi menemukan 5 bug yang saling terkait. Bug
1 (CI `ref: develop`) sudah diperbaiki + diverifikasi. Bug 2 (reviewer handoff
capture) sudah diperbaiki. Bug 3–5 masih terbuka dan membutuhkan perbaikan
permanen di runner/agent/schema (human-only), bukan patch manual per-task.

## Bug 1 — CI `ref: develop` (SELESAI)

**Gejala:** PR app repo `validate-changed-paths` gagal — "contract tidak
ditemukan di control repo".

**Akar:** `Checkout control repo` di `.github/workflows/path-enforcement.yml`
tak menyetel `ref`, sehingga `actions/checkout` mengambil **default branch =
`main`**. Kontrak task ada di `develop`.

**Fix:** tambah `ref: develop` di checkout control. Sudah diterapkan ke 6 repo
(original R&D, pub, kontrol, backend, android, iOS) + template canonical.

**Verifikasi:** backend PR #5 `validate-changed-paths` PASS, BE-301 merged.

## Bug 2 — Reviewer handoff dibuang (SELESAI)

**Gejala:** pipeline WARN "code-reviewer tidak menulis .task/handoff.json".

**Akar:** `spawn_agent` menjalankan `claude --print ... > /dev/null`, membuang
structured output reviewer, lalu menunggu file handoff yang tak pernah ditulis
(agent read-only, plan mode, tanpa Edit/Write).

**Fix:** runner tangkap stdout, ekstrak blok ```json, tulis ke
`.task/handoff.json` (Q9). Sudah diterapkan di `scripts/pipeline.sh` +
`scripts/review.sh` (original, pub, kontrol, develop).

## Bug 3 — Format handoff tidak konsisten (TERBUKA)

**Gejala:** `collect-result` / `collect-review` / `collect-qa` menolak handoff
dengan pelanggaran bervariasi:
- `changed_files`: got string, want object (FE-301)
- `tests`: got array, want object (BE-301, BE-302, QA)
- `findings`: missing `reason`, `recommended_action`; `category` invalid (BE-301 review)
- `decision`: "approved" bukan enum valid (BE-301 review)

**Akar:** agent implementer/QA/reviewer menghasilkan handoff dengan bentuk yang
berbeda dari `schemas/handoff.schema.json`:
- `changed_files` seharusnya array object `{path, purpose, change_kind}` —
  implementer menulis array string.
- `tests` seharusnya object `{executed: [{command, result, output_excerpt?}]}` —
  implementer/QA menulis array langsung.
- `findings` seharusnya `{severity, category, location, reason,
  recommended_action}` — reviewer menulis bentuk longgar (`file`/`line`/`message`,
  `category: "cleanup"`, `result: "pass"`).

**Mengapa:** prompt instruksi di `spawn_agent` (PROMPT_IMPL/PROMPT_REVIEW/
PROMPT_QA) tidak memuat bentuk schema secara eksplisit. Agent mengarang bentuk
sendiri, dan tidak ada validasi intermediate sebelum collect-*.

**Fix permanen yang diusulkan:**
1. **Pertegas prompt** di `scripts/pipeline.sh` — sebut bentuk field yang wajib:
   `changed_files` = array object, `tests` = `{executed: []}`, `findings` field
   lengkap. Rujuk `schemas/examples/handoff-BE-101.valid.yaml` sebagai contoh
   canonical.
2. **Validasi early** — jalankan `m2s validate-handoff` (atau `collect-*` dalam
   mode dry) segera setelah agent selesai, dan bila invalid, beri error ke agent
   dengan pelanggaran spesifik (auto-fix loop) alih-alih exit diam.

## Bug 4 — Agent spawn intermitten (TERBUKA)

**Gejala:** implementer BE-303 spawn lalu mati tanpa kerja: tak commit, tak
handoff, tak audit.log, tak PR. Pipeline WARN "handoff kosong" lalu exit.
Kontras dengan BE-301/CONTRACT-301 yang sukses dengan agent yang sama.

**Akar (hipotesis, belum terverifikasi):** model agent `cmb-agent-coding` /
`cmb-agent-review` kadang tak tersedia/resolvable (mirip kegagalan subagent
Explore di awal sesi dengan model `claude-opus-5`). Tanpa output/audit, tidak
ada bukti agen benar-benar berjalan.

**Fix permanen yang diusulkan:**
1. `spawn_agent` harus memeriksa exit code `claude --print` dan menulis stderr
   ke log (bukan `> /dev/null` + `|| true` yang menelan kegagalan).
2. Tambah retry bila exit code non-zero (mis. maks 2×).
3. Catat ke `audit.log` bahwa spawn terjadi + hasilnya (saat ini hanya WARN).

## Bug 5 — Pipeline resume salah (TERBUKA)

**Gejala:** setelah `collect-review` approve (status `reviewing`), pipeline yang
di-run ulang masuk **phase 3** (spawn implementer) alih-alih phase 5 (QA). Ini
terjadi karena guard phase 3 hanya memeriksa `implementation-complete`, tidak
mengenali `reviewing`.

**Akar:** `scripts/pipeline.sh` phase 3 loop:
```bash
CURRENT_STATUS=$(read_status)
[[ "$CURRENT_STATUS" == "implementation-complete" ]] && break
```
Bila status `reviewing` (approve) atau `merge-ready`, pipeline tidak break,
lalu `spawn_agent implementer` dijalankan kembali — menimpa handoff.

**Fix permanen yang diusulkan:**
- Phase 3 guard: tambah kondisi break untuk status `reviewing`,
  `qa-testing`, `merge-ready`, `ci-passed`, `merged` — resume dari titik yang
  benar, bukan selalu spawn implementer.
- Phase 4 guard: kenali `reviewing` (sudah approve) agar tidak re-run reviewer.
- Phase 5 guard: kenali `qa-testing` / `merge-ready` agar tidak re-run QA.

## Kepemilikan & Path

Semua fix di atas menyentuh **path human-only**: `scripts/**`, `cmd/m2s/**`,
`.claude/agents/**`, `schemas/**`, `.github/**`. Tidak boleh dikerjakan agent.
Pemilik fix: manusia operator M2S-VSH.

## Prioritas

| Bug | Dampak | Urutan |
|---|---|---|
| 3 (format handoff) | setiap task stuck di collect-* | 1 |
| 5 (resume) | task approve tak bisa lanjut QA | 2 |
| 4 (spawn intermitten) | task gagal total tanpa bukti | 3 |
| 2 (reviewer capture) | SELESAI | — |
| 1 (CI ref) | SELESAI | — |

## Rekomendasi eksekusi

1. Fix bug 3 + 5 + 4 di `scripts/pipeline.sh` + `scripts/review.sh` (satu PR,
   karena satu file).
2. `make verify` di control repo.
3. Re-run pipeline task yang stuck (BE-303, FE-301, IOS-301, BE-302) dengan
   script yang sudah diperbaiki.
