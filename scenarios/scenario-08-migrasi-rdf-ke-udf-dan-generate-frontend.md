# Skenario 8: Migrasi RDF ke UDF dan Generate Aplikasi Frontend

> Tahap kedelapan onboarding RESTForge. Migrasi payload backend (RDF) menjadi payload frontend (UDF), aktivasi license RESTForge Designer pada workspace frontend, lalu generate aplikasi frontend (HTML/JS/CSS) dari UDF tersebut. Command `payload migrate` menghasilkan UDF dalam format split multi-file secara default.

---

## Tujuan (Objective)

1. Struktur UDF split ter-generate di folder `sandbox\frontend\payload\` sebagai hasil konversi dari RDF `sandbox\backend\payload\visitors.json`, terdiri atas `app-config.json`, `pages\visitors.json`, dan aggregator `visitors-app.json`
2. License RESTForge Designer berstatus aktif (valid) pada mesin
3. Aplikasi frontend ter-generate ke folder `sandbox\frontend\apps\visitors-app\` berisi file HTML, JS, CSS, dan asset plugin

---

## Sumber Rujukan (Reference)

| Command | Handbook |
|---------|----------|
| `npx restforge payload migrate` | `restforge-handbook/commands/restforge-backend/payload/migrate.md` |
| `restforge-designer activate` | `restforge-handbook/commands/restforge-frontend/activate.md` |
| `restforge-designer license status` | `restforge-handbook/commands/restforge-frontend/license-status.md` |
| `restforge-designer generate` | `restforge-handbook/commands/restforge-frontend/generate.md` |

---

## Prasyarat (Prerequisite)

| Item | Cara Verifikasi |
|------|-----------------|
| Skenario 7 selesai | Endpoint `visitors` sudah tervalidasi via curl dan Postman; folder `sandbox\backend\payload\visitors.json` tersedia sebagai RDF source untuk migrasi; folder `sandbox\frontend\` sudah ter-create kosong sejak Skenario 1 |
| RESTForge Designer ter-install system-level | Command `restforge-designer --version` dapat dipanggil dari working directory mana saja |
| License key tersedia | Format `XXXX-XXXX-XXXX-XXXX` (4 segmen, masing-masing 4 karakter uppercase A-Z atau 0-9) |

Catatan instalasi RESTForge Designer: installer resmi tersedia pada [https://restforge.dev/download.html](https://restforge.dev/download.html). Pada halaman tersebut, pilih card **RESTForge Designer** lalu klik tombol **Download for Windows (x64)** untuk mengunduh Desktop Installer (.exe). Jalankan installer hingga selesai, kemudian verifikasi melalui command `restforge-designer --version` sebelum melanjutkan ke langkah eksekusi.

---

## Langkah Eksekusi (Execution Steps)

### Langkah 1: Migrasi RDF ke UDF dari Workspace Backend

Pastikan working directory berada di `sandbox\backend`, lalu jalankan command migrate:

```bat
npx restforge payload migrate --project=visitors-app --name=visitors.json --output=../frontend/payload --config=db-connection.env
```

Output yang diharapkan menampilkan ringkasan migrasi beserta daftar file yang ditulis:

```
============================================================
PAYLOAD MIGRATE - RDF (backend) -> UDF (frontend, split)
============================================================

  Input        : ...\sandbox\backend\payload\visitors.json
  Output dir   : ...\sandbox\frontend\payload
  Project      : visitors-app
  apiBaseUrl   : http://127.0.0.1:3000/api/visitors-app
  Backend port : 3000
  Frontend port: 8000
  Homepage     : visitors
  Pages        : 1

  [OK] visitors: 3 field(s), 3 table column(s)

  Files written:
    - app-config.json
    - pages\visitors.json
    - visitors-app.json

  Migration completed successfully.
```

Command menulis struktur split multi-file ke `Output dir`:

| File | Isi |
|------|-----|
| `app-config.json` | Blok `appConfig` (appName, appCode, plugin, apiBaseUrl, port) yang dipakai bersama oleh seluruh page |
| `pages\visitors.json` | Definisi page `visitors` (fields, features) dalam pembungkus `{ "pages": [ ... ] }` |
| `visitors-app.json` | Aggregator bernama sesuai `--project`: menarik `appConfig` via `extends`, menyusun page via `include`, serta mendefinisikan `homepage` dan `navigation` |

Verifikasi struktur UDF ter-create:

```bat
dir ..\frontend\payload
dir ..\frontend\payload\pages
```

`dir ..\frontend\payload` harus menampilkan `app-config.json`, `visitors-app.json`, dan folder `pages`, sedangkan `dir ..\frontend\payload\pages` menampilkan `visitors.json`.

Catatan: bila file output sudah ada dari eksekusi sebelumnya, tambahkan flag `--overwrite` untuk menimpa seluruh file split tersebut.

---

### Langkah 2: Aktivasi License RESTForge Designer

Pindah working directory ke workspace frontend (`sandbox\frontend`, satu level di atas `backend`), lalu jalankan command activate dengan license key yang tersedia (ganti `XXXX-XXXX-XXXX-XXXX` dengan license key aktual):

```bat
restforge-designer activate --key=XXXX-XXXX-XXXX-XXXX
```

Output sukses:

```
╭─ Activation Complete ───────────────────────╮
│ Validation : successful                     │
│ Product    : RESTForge Designer             │
│ Expires    : YYYY-MM-DD                     │
│ Storage    : OS keyring                     │
╰─────────────────────────────────────────────╯
```

Catatan: license berlaku global per user account, bukan per workspace. Bila pada mesin yang sama sudah pernah dilakukan `activate` dengan license key yang sama, langkah ini dapat di-skip. Bila ditemukan pesan `License Already Active` dan license key berbeda, gunakan flag `--force` untuk overwrite.

---

### Langkah 3: Verifikasi License Aktif

Pastikan license berstatus valid sebelum generate:

```bat
restforge-designer license status
```

Output yang diharapkan menampilkan tabel status dan panel validasi:

```
┌─────────────────────┬───────────────────────────────┐
│ Field               │ Value                         │
├─────────────────────┼───────────────────────────────┤
│ Active key          │ XXXX-XXXX-XXXX-XXXX           │
│ Resolved from       │ encrypted storage             │
│ Encrypted storage   │ OS keyring                    │
│ OS keyring          │ yes                           │
└─────────────────────┴───────────────────────────────┘

╭─ Validation Result ─────────────────────────╮
│ Validation : valid                          │
│ Product    : RESTForge Designer             │
│ Expires    : YYYY-MM-DD                     │
╰─────────────────────────────────────────────╯
```

Field `Validation` harus bernilai `valid` (atau `valid (cached)`). Bila bernilai invalid atau expired, hentikan eksekusi dan koordinasikan dengan pemilik license.

---

### Langkah 4: Generate Aplikasi Frontend dari UDF

Dari working directory `sandbox\frontend`, jalankan command generate. Flag `--payload` menunjuk ke aggregator `visitors-app.json`, bukan ke fragmen `pages\visitors.json`. Aggregator inilah yang memuat `appConfig` (via `extends`) dan merangkai seluruh page (via `include`); fragmen di folder `pages\` tidak memuat `appConfig` sehingga tidak dapat dijadikan payload langsung:

```bat
restforge-designer generate --payload=payload/visitors-app.json --output=./apps/visitors-app --overwrite
```

Output menampilkan ringkasan jumlah file yang ditulis dan daftar warning (jika ada).

Verifikasi hasil generate:

```bat
dir apps\visitors-app
```

Output harus menampilkan file shared (mis. `index.html`, `app-start.bat`, folder `js\`, `css\`) dan file per-page (mis. `visitors.html`, `js\visitors.js`) beserta asset plugin.

---

## Langkah Berikutnya (Next Step)

Lanjut ke Skenario 9: run aplikasi frontend dan verifikasi GET visitors.
