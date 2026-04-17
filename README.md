# Text Mining X — Food Tray MBG

📊 **Dataset & hasil coding (Google Spreadsheet):** <https://docs.google.com/spreadsheets/d/1gfV87Z-3_2k7teawvWo-L7na2VEt_ZHIQfODutev8Co/edit?usp=sharing>

Hasil run text mining **X/Twitter** untuk penelitian skripsi
**"Redesign Food Tray MBG pada Sistem Distribusi SPPG"** (Wisnu, 2026).

---

## Ringkasan Run

| | |
|---|---|
| **Run ID** | `883a71c5-d986-4477-a222-7c588d2d16eb` |
| **Timestamp (UTC)** | 2026-04-17T16:25:56Z |
| **Window** | 2025-08-01 → 2025-10-31 (fixed, bukan rolling) |
| **Query packs** | 6 cluster (`umum_tray_mbg`, `isu_suhu`, `isu_tumpah_tutup`, `isu_material`, `isu_distribusi`, `isu_higiene`) |
| **Bahasa** | `id` (Indonesia), retweet excluded |
| **Total raw** | 94 post |
| **Total cleaned** | 93 post |
| **Total relevant** | 82 post |
| **Total coded** | 79 post (≥1 tema) |
| **LLM** | claude-opus-4.7 |
| **Sumber data** | MCP `user-xpoz` → `getTwitterPostsByKeywords` |
| **config_hash** | `144c95931fe3e84a` |

### Distribusi tema (post coded)

| Tema | Count | % |
|---|---:|---:|
| material_keamanan | 47 | 59.49% |
| higienitas | 32 | 40.51% |
| retensi_suhu | 10 | 12.66% |
| handling_stackability | 9 | 11.39% |
| ergonomi_penggunaan | 6 | 7.59% |
| kebocoran_tumpah | 6 | 7.59% |

### Distribusi sentimen (post coded)

| Label | Count | % |
|---|---:|---:|
| sangat_negatif | 23 | 29.1% |
| negatif | 33 | 41.8% |
| netral | 13 | 16.5% |
| positif | 10 | 12.7% |
| sangat_positif | 0 | 0.0% |

Post bersifat **multi-label** (satu post bisa membawa >1 tema), sehingga
total persentase tema > 100%.

---

## Struktur File

```
text-mining-x-food-tray-mbg/
├── _raw_fetched.json         ← payload mentah dari xpoz (sebelum normalisasi)
├── _run_meta.json            ← metadata run (params, counts, config_hash)
│
├── 01_raw.csv                ← post raw setelah normalisasi schema
├── 02_cleaned.csv            ← setelah dedupe + language filter + clean text
├── 03_relevant.csv           ← setelah filter relevansi (LLM + keyword fallback)
├── 04_coded.csv              ← + sentiment + theme labels (multi-label JSON)
├── 05_theme_summary.csv      ← agregat count/pct per tema
├── 06_top_words.csv          ← top 50 token (stopword Indonesia sudah dibuang)
├── 07_representative.csv     ← 5 post paling representatif per tema
├── 08_need_statements.csv    ← need statement + design attributes per tema
├── 09_manual_review.csv      ← subset untuk validasi manual peneliti
│
├── report.md                 ← laporan naratif (Ringkasan Eksekutif + Need Statements)
└── laporan_food_tray_mbg.pdf ← versi PDF dari report.md
```

### Skema kunci

**`04_coded.csv`** — satu baris per post, berisi:
- Metadata: `tweet_id`, `created_at`, `author_username`, `tweet_url`,
  engagement (`like_count`, `reply_count`, `repost_count`, dst).
- `text_raw`, `text_clean`.
- `relevance_score` (0–1), `relevance_reason`.
- `sentiment_label` ∈ {`sangat_negatif`, `negatif`, `netral`,
  `positif`, `sangat_positif`}, `sentiment_reason`.
- `themes` — JSON array, contoh `["material_keamanan","higienitas"]`.
- `theme_evidence` — JSON object tema → kutipan pendukung.

**`08_need_statements.csv`** — satu baris per tema:
`theme`, `priority`, `frequency`, `need_statement`,
`design_attributes` (JSON array), `evidence_quotes` (JSON array).

Skema lengkap: lihat `schemas/coded_post.schema.json` di repo induk.

---

## Need Statements (TL;DR)

Prioritas diturunkan dari frekuensi + severity, bukan sekadar jumlah
post. Detail + atribut desain turunan + kutipan bukti ada di
`report.md` §5.

1. **material_keamanan** _(prioritas: tinggi, 47 post)_ — Food tray
   MBG harus menggunakan material terverifikasi food-grade,
   bersertifikat halal, dan bebas risiko kontaminasi sepanjang rantai
   produksi (turunan: SS 304 bukan SS 201, sertifikasi BPJPH + SNI,
   auditable supply chain, QR-traceability).
2. **higienitas** _(tinggi, 32)_ — Harus mendukung sanitasi efektif,
   kompatibel mesin cuci industrial 85°C, tanpa celah/sudut tajam,
   gasket dapat dilepas (turunan: geometri non-porous, indikator
   visual tray kotor).
3. **retensi_suhu** _(tinggi, 10)_ — Harus mempertahankan suhu aman
   pangan (≥60°C) tanpa memerangkap uap penyebab basi (turunan:
   double-wall liner, ventilasi satu arah, gasket silikon food-grade).
4. **handling_stackability** _(sedang, 9)_ — Harus stackable stabil
   tanpa workaround rafia (turunan: self-locking interlocking, handle
   ergonomis, frame distribusi).
5. **ergonomi_penggunaan** _(sedang, 6)_ — Harus nyaman untuk siswa
   SD & petugas (turunan: <500g, bukaan satu-tangan, permukaan
   anti-panas).
6. **kebocoran_tumpah** _(sedang, 6)_ — Rapat terhadap cairan,
   breathable terhadap uap (turunan: sekat kompartemen, sealing
   differential, tes 45°).

---

## Keterbatasan

- **Volume kecil** (82 post relevan) → kesimpulan tematik indikatif,
  bukan representatif populasi.
- **Bias platform X** → diskusi cenderung politis; suara langsung
  petugas SPPG / siswa under-represented.
- **Periode Ags–Okt 2025** didominasi isu viral material non-halal
  (minyak babi) → mempengaruhi distribusi tema.
- **LLM bisa halusinasi** → `09_manual_review.csv` disediakan untuk
  audit manual oleh peneliti (wajib sebelum sitasi akademik).
- 2 query pack (`isu_tumpah_tutup`, `isu_distribusi`) sempat 0-hit di
  `countTweets` tapi search actual return hasil → beda indeks antara
  `countTweets` vs `getTwitterPostsByKeywords` di xpoz.
