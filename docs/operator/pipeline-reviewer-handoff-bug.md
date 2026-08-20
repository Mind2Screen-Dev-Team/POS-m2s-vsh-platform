# Temuan: Pipeline Code Reviewer tidak dapat menulis handoff (bug spawn_agent)

**Tanggal:** 2026-08-12
**Versi:** M2S-VSH Lite v0.1.0
**Status:** Open (menunggu fix manusia — path human-only)
**Lingkup:** `scripts/pipeline.sh`, `scripts/review.sh`, kontrak task `CONTRACT-301`

## Ringkasan

Pipeline **mengirim Code Reviewer sebagai agent read-only (plan mode, tanpa
Edit/Write)**, tetapi runner lalu mengharapkan `.task/handoff.json` ada di
worktree untuk diproses `collect-review`. Handoff itu **tidak pernah dibuat** —
ada kesalahan desain antara dua komponen: `spawn_agent` membuang output
reviewer, dan `collect-review` membaca handoff dari file yang tidak ada.
Akibatnya pipeline berhenti di phase 4 dengan status `implementation-complete`
(bukan `reviewing`/`changes-requested`), dan task tidak pernah lanjut ke QA.

Bug ini ada di **kedua repository** (asli dan control clone) karena `pipeline.sh`
dan `review.sh` diidentikkan (diff kosong).

## Reproduksi

1. Task kontrak mencapai `implementation-complete` (implementer selesai, PR dibuat).
2. Pipeline masuk phase 4: `launch-review` → spawn Code Reviewer.
3. Log:
   ```
   [pipeline:CONTRACT-301] phase 4: spawn code-reviewer model=cmb-agent-review
   [pipeline:CONTRACT-301] spawn role=code-reviewer model=cmb-agent-review cwd=/.../CONTRACT-301
   [pipeline:CONTRACT-301] WARN: code-reviewer tidak menulis .task/handoff.json — handoff kosong
   ```
4. `collect-review` dipanggil dengan handoff kosong → pipeline berhenti.
5. Status task tetap `implementation-complete` (tidak maju).

## Akar Masalah (dua lapis)

### 1. `spawn_agent` membuang output reviewer (`scripts/pipeline.sh:110`, `scripts/review.sh:54`)

```bash
(cd "$wt" && printf '%s' "$prompt" \
  | claude --print \
      --model "$model" \
      --allowedTools "$tools" \
  > /dev/null) || true   # ← output reviewer dibuang
```

- Reviewer menghasilkan **structured output** (handoff JSON) di stdout
  (`claude --print` mencetak output ke stdout).
- `> /dev/null` membuang output itu.
- Setelah spawn, pipeline memeriksa `[[ -f "$wt/.task/handoff.json" ]]` — file
  ini **tidak pernah ada** karena reviewer tidak menulis file apa pun.

### 2. `collect-review` membaca handoff dari file (`cmd/m2s/commands.go:693`)

```go
handoffPath := fs.String("handoff", "", "path handoff review (.yaml)")
...
doc, err := v.Load(*handoffPath, contract.KindHandoff)
```

- `collect-review` memerlukan path handoff **file** yang valid.
- Karena reviewer tidak menulis file, `collect-review` selalu gagal saat
  `-handoff` menunjuk file kosong/tidak ada.

### Kontradiksi desain (dokumentasi)

- Agent def `code-reviewer.md`: "read-only, plan mode tanpa Write/Edit (A-03,
  Q9)... **Agent mengembalikan structured output; runner yang menuliskannya** ke
  `reviews/code/**`."
- Handoff schema `handoff.schema.json:21`: "code-reviewer juga menghasilkan
  handoff meski read-only (Q9)" + `changed_files` wajib `maxItems: 0`.
- Review-report schema: "Dokumen registry/referensi — belum menjadi Kind
  validator runner."
- **Kesimpulan:** desain menyebut "runner yang menulis", tapi runner (`pipeline.sh`
  / `review.sh`) tidak menangkap output — bug di lapisan orchestrasi, bukan di
  agen.

## Bukti di repo

| Artifact | Temuan |
|---|---|
| `scripts/pipeline.sh:110` | `> /dev/null` membuang stdout reviewer |
| `scripts/review.sh:54` | Sama persis (jalur standalone) |
| `cmd/m2s/commands.go:693` | `collect-review` butuh `-handoff` file |
| `.claude/agents/code-reviewer.md` | read-only, "runner yang menuliskannya" (Q9) |
| `schemas/handoff.schema.json:21,231-238` | handoff reviewer wajib `decision`, `changed_files: []` |
| `schemas/review-report.schema.json:9` | "belum menjadi Kind validator runner" |
| Diff `orig/pipeline.sh` vs kontrol | identical (bug disalin) |
| Contoh `schemas/examples/handoff-review-BE-101.valid.yaml` | bentuk normatif handoff reviewer |

## Temuan Tambahan (CI)

PR #4 (`[task CONTRACT-301]`) — `contracts/CONTRACT-301.yaml` saja diubah (valid,
dalam allowed path), tapi CI `validate-changed-paths` **FAIL**. Log gagal tidak
menunjukkan step yang menjalankan validasi (UNKNOWN STEP). Terpisah dari bug
handoff, perlu investigasi tersendiri.

## Solusi yang Diusulkan (acuan fix — dipilih manusia)

### Opsi A (direkomendasikan): tangkap output reviewer → tulis handoff oleh runner

Ubah `spawn_agent` agar menangkap stdout reviewer, validasi sebagai handoff, dan
tulis ke `.task/handoff.json` (peran "runner yang menulis" sesuai Q9):

```bash
# scripts/pipeline.sh — ganti blok `> /dev/null`
out="$(cd "$wt" && printf '%s' "$prompt" \
  | claude --print --model "$model" --allowedTools "$tools" 2>/dev/null || true)"
# out berisi structured output reviewer (JSON/YAML handoff)
if [[ -n "$out" ]]; then
  printf '%s\n' "$out" > "$wt/.task/handoff.json"
fi
```

Catatan:
- Reviewer dilarang Edit/Write app code — menulis `.task/handoff.json` **oleh
  runner** (bukan agent) tetap memenuhi batasan plan mode.
- `collect-review` tetap bekerja tanpa perubahan (membaca file yang kini ada).
- Perlu penanganan: apakah output `claude --print` berupa YAML/JSON murni atau
  bercampur log. Disarankan agen diinstruksikan menandai output (mis. blok
  `---HANDOFF---` ... `---END HANDOFF---`) atau runner mengekstrak fragmen
  valid pertama.

### Opsi B: beri reviewer tools minimal untuk menulis handoff

Tambahkan `Edit`/`Write` pada `--allowedTools` reviewer hanya untuk path
`.task/handoff.json` (permission scoped), sehingga reviewer menulis handoff
langsung. Lebih sederhana tetapi melanggar prinsip "reviewer read-only"
(plan mode, A-03, Q9) — menurunkan jaminan tidak menulis app code.

### Opsi C: pisahkan "produksi review" dari "penulisan handoff"

Refactor pipeline: spawn reviewer sekali untuk menghasilkan review (structured
output), runner menulisnya; implementer tidak ikut. Sama esensi Opsi A tetapi
didesain lebih rapi untuk fix loop (maks 3×).

## Langkah Verifikasi Setelah Fix

1. Jalankan ulang `./scripts/pipeline.sh --task CONTRACT-301` (status sudah
   `implementation-complete`, pipeline lanjut ke phase 4).
2. Harapkan: reviewer menghasilkan handoff → `collect-review` sukses → status
   `reviewing` (approve) → phase 5 QA.
3. `make verify` (validate wrapper + schemas + agents + hooks) di control repo.
4. `go test ./cmd/m2s/...` — pastikan unit test collect-review masih lolos.
5. Uji jalur standalone: `./scripts/review.sh --task CONTRACT-301`.

## Kepemilikan & Path

Fix ini menyentuh **path human-only** (`scripts/**`, `cmd/m2s/**`) — tidak boleh
dikerjakan agent. Pemilik fix: manusia operator M2S-VSH. Setelah fix, update
dokumen ini ke status Resolved.
