# Skenario 8: Migrasi RDF ke UDF dan Generate Aplikasi Frontend

> Tahap kedelapan onboarding RESTForge. Migrasi payload backend (RDF) menjadi payload frontend (UDF), aktivasi license RESTForge Designer pada workspace frontend, lalu generate aplikasi frontend (HTML/JS/CSS) dari UDF tersebut.

---

## Tujuan (Objective)

1. File UDF `sandbox\frontend\payload\visitors.json` ter-generate sebagai hasil konversi dari RDF `sandbox\backend\payload\visitors.json`
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

Pindah working directory ke `sandbox\backend`, lalu jalankan command migrate:

```bat
cd playbook\sandbox\backend
npx restforge payload migrate --name=visitors.json --project=myapp --output=../frontend/payload --config=db-connection.env
```

Output yang diharapkan menampilkan ringkasan migrasi:

```
============================================================
PAYLOAD MIGRATE - RDF (backend) -> UDF (frontend)
============================================================

  Input        : ...\sandbox\backend\payload\visitors.json
  Output       : ...\sandbox\frontend\payload\visitors.json
  Project      : myapp
  apiBaseUrl   : http://127.0.0.1:3000/api/myapp
  Port         : 3000
  Pages        : 1

  [OK] visitors: 3 field(s), 3 table column(s)

  Migration completed successfully.
```

Verifikasi file UDF ter-create:

```bat
dir ..\frontend\payload\visitors.json
```

Catatan: bila file output sudah ada dari eksekusi sebelumnya, tambahkan flag `--overwrite` untuk menimpa.

---

### Langkah 2: Aktivasi License RESTForge Designer

Pindah working directory ke workspace frontend:

```bat
cd ..\frontend
```

Jalankan command activate dengan license key yang tersedia (ganti `XXXX-XXXX-XXXX-XXXX` dengan license key aktual):

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

Dari working directory `sandbox\frontend`, jalankan command generate:

```bat
restforge-designer generate --payload=payload/visitors.json --output=./apps/visitors-app --overwrite
```

Output menampilkan ringkasan jumlah file yang ditulis dan daftar warning (jika ada).

Verifikasi hasil generate:

```bat
dir apps\visitors-app
```

Output harus menampilkan file shared (mis. `index.html`, `app-start.bat`, folder `js\`, `css\`) dan file per-page (mis. `visitors.html`, `js\visitors.js`) beserta asset plugin.

---

## Kriteria Selesai (Completion Criteria)

| Item | Kondisi |
|------|---------|
| File UDF | `sandbox\frontend\payload\visitors.json` ter-create dari hasil migrate |
| License Designer | `restforge-designer license status` menampilkan `Validation: valid` |
| Folder aplikasi | `sandbox\frontend\apps\visitors-app\` berisi `index.html`, `app-start.bat`, folder `js\`, `css\`, dan asset plugin |
| File per-page | `visitors.html` dan `js\visitors.js` ter-generate di folder aplikasi |

---

## Catatan untuk Tahap Berikutnya (Notes for Next Stage)

Skenario 9 akan membahas cara menjalankan aplikasi frontend hasil generate (`app-start.bat`), test integrasi dengan backend server (CRUD `visitors` end-to-end dari browser ke database), dan editing manual UDF untuk kustomisasi tampilan (mis. `fieldRows`, label, icon).

---

## Pelaporan Issue (Issue Reporting)

Apabila ditemukan kondisi tidak sesuai ekspektasi, hentikan eksekusi dan dokumentasikan permasalahan beserta output lengkap perintah yang gagal, nomor langkah saat error terjadi, output `dir` pada folder yang relevan (`sandbox\backend\payload`, `sandbox\frontend\payload`, atau `sandbox\frontend\apps`), serta output `restforge-designer license status` apabila terkait verifikasi license.
