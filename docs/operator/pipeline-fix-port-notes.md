# Catatan Port Perbaikan Pipeline — Repo POS

**Tanggal:** 2026-08-13
**Sumber:** control repo `m2s-vsh-platform` commit `e5e0d77` (branch `main`)
**Status:** Catatan untuk dikerjakan manual dari directory repo POS. Belum dieksekusi.

Perbaikan ini sudah diterapkan di control repo (commit `e5e0d77`) dan pub
(`pub-m2s-vsh-platform` commit `d8f9041`). Repo POS punya sejarah branch yang
berbeda, jadi catatan ini menandai apa dan di mana perubahannya.

## Konteks (apa masalahnya)

Dokumen `docs/operator/pipeline-design-recommendations.md` (di control repo)
merangkum tiga akar masalah: pipeline memperlakukan output agent LLM sebagai
deterministik. Tiga perbaikan:

1. **Structured output** — `spawn_agent` pakai `--json-schema` + `--output-format text`
   agar agent dipaksa patuh schema di lapisan API (bukan instruksi prompt).
2. **Hapus `normalize_handoff`** — python heredoc `sev_map`/`cat_map` redundant;
   structured output + Go validator sudah dua lapis enforce.
3. **Error boundary** — tiap `launch-*`/`collect-*` dibungkus `if ! cmd` dengan log jelas.

## Peta branch POS

Branch POS **tidak seragam**. Periksa dulu branch mana yang mau di-fix:

| Branch | Baris pipeline.sh | `normalize_handoff` | `gh pr list` | Catatan |
|---|---|---|---|---|
| `agent/pos-start` | 512 | 5 | 3 | **Modern** — mirip kontrol pre-fix. Target utama. |
| `develop` | 341 | 0 | 0 | Older — tanpa normalize_handoff, tanpa gh pr list |
| `main` | 320 | 0 | 0 | Older |
| `staging` | 320 | 0 | 0 | Older |

`agent/pos-start` adalah branch yang sudah memuat `normalize_handoff` + `gh pr list`
(persis kondisi kontrol sebelum commit fix). Branch `develop`/`main`/`staging`
adalah versi lebih lama dengan `spawn_agent` tanpa retry, baca `pr_url` dari
handoff (bukan `gh pr list`), tanpa resume guards.

**Rekomendasi:** fix di `agent/pos-start` dulu (port bersih, struktur sudah cocok),
lalu merge ke `develop`. Branch older perlu keputusan terpisah (upgrade penuh vs
biarkan) — jangan campur.

## Perubahan file (3 file, sama persis dengan kontrol)

### 1. `scripts/pipeline.sh`

Perubahan relatif terhadap versi `agent/pos-start` (yang identik dgn kontrol pre-fix):

- **`spawn_agent`** (fungsi, ~line 210): tambah `--json-schema "$schema"` +
  `--output-format text` ke perintah `claude --print`; hapus ekstraksi manual
  blok ```json/```yaml (awk/sed/python). Bundle schema via
  `scripts/bundle-handoff-schema.sh`, baca isi file (`cat`) karena `--json-schema`
  menerima JSON inline bukan path.
- **Hapus fungsi `normalize_handoff`** (seluruh blok python heredoc ~line 96-197,
  100+ baris). Hapus 5 pemanggilannya: `collect-result` (phase 3), `collect-review`
  (phase 4), `collect-qa` (phase 5), dan re-spawn implementer (2×).
- **Error boundary**: bungkus tiap `collect-*`, `launch-*`, `reserve-paths` dalam
  `if ! "$M2S_BIN" ...; then step_log "ERROR: ..."; exit 1 / continue; fi`.
  `set -euo pipefail` global dipertahankan.
- **Prompt** (`PROMPT_IMPL`/`PROMPT_REVIEW`/`PROMPT_QA`): ubah instruksi
  "tulis blok ```json" jadi "keluarkan structured output (JSON) di STDOUT".

Cara termudah untuk `agent/pos-start`: copy `scripts/pipeline.sh` dari control
repo commit `e5e0d77` (isinya identik).

### 2. `scripts/bundle-handoff-schema.sh` (BARU)

File baru. Bundle `schemas/handoff.schema.json` + `schemas/common.schema.json`
jadi satu file tanpa `$ref` lintas-file dan tanpa `$schema` meta-ref (karena
`claude --json-schema` menolak keduanya). Hasil ditulis ke
`schemas/.bundle/handoff.bundled.json`. Wajib `chmod +x`.

### 3. `.gitignore`

Tambah satu blok (setelah blok `bin/`):

```
# ── Schema bundled (generated) — hasil scripts/bundle-handoff-schema.sh ──
# Dibangkitkan saat runtime untuk `claude --json-schema` (yang menolak $ref
# lintas-file). Kanonik tetap schemas/*.schema.json.
schemas/.bundle/
```

## Verifikasi (setelah port)

```sh
bash -n scripts/pipeline.sh scripts/bundle-handoff-schema.sh   # syntax
bash scripts/bundle-handoff-schema.sh                          # bundle jalan
./scripts/pipeline.sh --task <id-nyata> --dry-run              # alur + schema populate
make check                                                     # fmt + vet + test
```

## Catatan penting

- **Schema POS sudah identik** dgn kontrol (`handoff.schema.json` + `common.schema.json`
  tidak berubah). Tidak perlu sentuh `schemas/`.
- **Model agent** POS identik dgn kontrol (`cmb-agent-coding`/`cmb-agent-review`/dst).
  Structured output sudah teruji bekerja dengan model-model ini.
- Peringatan "model not recognized" muncul di stderr (sudah ke `audit.log`), bukan
  blocker.
- `schemas/.bundle/` dihasilkan runtime, gitignored. Kanonik tetap `schemas/*.schema.json`.
