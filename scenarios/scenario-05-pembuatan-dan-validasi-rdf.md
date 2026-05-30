# Skenario 5: Pembuatan dan Validasi RDF

> Tahap kelima onboarding RESTForge. Tujuan akhir skenario ini adalah satu file RDF (Resource Definition File) untuk endpoint REST `visitors` dibuat dari introspeksi schema database dan lolos validasi terhadap struktur tabel actual.

---

## Tujuan (Objective)

Setelah menyelesaikan skenario ini, kondisi berikut harus tercapai:

1. File `payload\visitors.json` sudah ter-generate hasil introspeksi dari tabel `visitors` pada database PostgreSQL
2. Isi file RDF berisi struktur dasar (`tableName`, `fieldName`, `action`) yang konsisten dengan struktur tabel actual
3. Perintah `npx restforge payload validate --table=visitors` mengembalikan status `[OK]` tanpa drift

Catatan: skenario ini hanya mencakup pembuatan dan validasi RDF di sisi filesystem. Generate endpoint REST dari RDF (`endpoint create`) akan dibahas pada Skenario 6.

---

## Sumber Rujukan (Reference)

Seluruh perintah dan struktur RDF pada skenario ini bersumber dari handbook berikut:

| Topik | File Handbook |
|-------|---------------|
| Resource `payload` dan daftar sub-command | [commands/payload/README.md](https://github.com/restforge/handbook/blob/main/commands/payload/README.md) |
| Generate RDF dari schema database | [commands/payload/generate.md](https://github.com/restforge/handbook/blob/main/commands/payload/generate.md) |
| Validasi RDF terhadap struktur tabel | [commands/payload/validate.md](https://github.com/restforge/handbook/blob/main/commands/payload/validate.md) |
| Drift detection per-column antara RDF dan database | [commands/payload/diff.md](https://github.com/restforge/handbook/blob/main/commands/payload/diff.md) |
| Sinkronisasi RDF dengan database | [commands/payload/sync.md](https://github.com/restforge/handbook/blob/main/commands/payload/sync.md) |
| Anatomi RDF dan struktur file | [catalogs/rdf/README.md](https://github.com/restforge/handbook/blob/main/catalogs/rdf/README.md) |
| Field wajib RDF | [catalogs/rdf/mandatory-fields.md](https://github.com/restforge/handbook/blob/main/catalogs/rdf/mandatory-fields.md) |
| Field validasi per kolom | [catalogs/rdf/field-validation.md](https://github.com/restforge/handbook/blob/main/catalogs/rdf/field-validation.md) |
| Audit columns | [catalogs/rdf/audit-columns.md](https://github.com/restforge/handbook/blob/main/catalogs/rdf/audit-columns.md) |
| Konvensi penamaan RDF | [catalogs/rdf/naming-convention.md](https://github.com/restforge/handbook/blob/main/catalogs/rdf/naming-convention.md) |
| Pemakaian default config tanpa flag eksplisit | [commands/conventions.md](https://github.com/restforge/handbook/blob/main/commands/conventions.md) |

---

## Prasyarat (Prerequisite)

| Item | Cara Verifikasi |
|------|-----------------|
| Skenario 4 sudah selesai | Tabel `visitors` sudah ter-create pada database PostgreSQL |
| Working directory aktif di `sandbox\backend` | Jalankan `cd` pada cmd, path harus berakhiran `\playbook\sandbox\backend` |
| Folder `payload\` sudah ada | Folder ter-create oleh `restforge init` di Skenario 2. Verifikasi via `dir payload` |
| Default config sudah di-set | Perintah `npx restforge config get-default` menampilkan `config\db-connection.env` |

---

## Langkah Eksekusi (Execution Steps)

### Langkah 1: Verifikasi Working Directory dan Default Config

Pastikan posisi cmd berada di folder `sandbox\backend`:

```bat
cd
```

Output harus menunjukkan path yang berakhiran `\playbook\sandbox\backend`. Jika tidak, pindah dari folder root project:

```bat
cd playbook\sandbox\backend
```

Verifikasi default config aktif:

```bat
npx restforge config get-default
```

Output harus menampilkan path `config\db-connection.env`. Command pada resource `payload` akan menggunakan default config tersebut apabila flag `--config` tidak disediakan eksplisit. Hal ini memungkinkan seluruh perintah `payload *` pada Skenario 5 dijalankan tanpa `--config`.

---

### Langkah 2: Generate RDF dari Schema Database

Perintah `payload generate` melakukan introspeksi tabel database dan menghasilkan file RDF JSON. Flag `--table` wajib untuk menentukan tabel sumber.

Jalankan generate untuk tabel `visitors`:

```bat
npx restforge payload generate --table=visitors
```

Output akan men-generate file `payload\visitors.json` pada folder default `payload/`.

Verifikasi file ter-generate:

```bat
dir payload\visitors.json
```

Output `dir` harus menampilkan entri file tanpa error.

#### Output Directory Custom (Opsional)

Flag `--output` dapat digunakan untuk mengarahkan output ke folder lain (default: `payload/`):

```bat
npx restforge payload generate --table=visitors --output=./my-payloads
```

Pada skenario onboarding ini gunakan folder default `payload/` agar konsisten dengan konvensi handbook.

---

### Langkah 3: Inspeksi File RDF yang Ter-generate

Detail anatomi lengkap seluruh field RDF (termasuk `fieldValidation`, `defaultScope`, dan field opsional lainnya) tersedia pada handbook (lihat tabel Sumber Rujukan, khususnya `catalogs/rdf/README.md` dan `catalogs/rdf/mandatory-fields.md`).

Buka file hasil generate untuk inspeksi visual:

```bat
notepad payload\visitors.json
```

Isi file harus mencakup struktur dasar berikut:

| Section | Keterangan |
|---------|-----------|
| `tableName` | Nilai `"visitors"` |
| `primaryKey` | Nilai `"visitor_id"` (dari PK tabel `visitors`) |
| `fieldName` | Array berisi 4 kolom non-audit: `visitor_id`, `name`, `email`, `phone` |
| `action` | Object dengan flag default untuk action yang umum (datatables, create, update, delete, read, lookup, first) |

Catatan dari handbook [catalogs/rdf/audit-columns.md](https://github.com/restforge/handbook/blob/main/catalogs/rdf/audit-columns.md): kolom audit standar (`created_at`, `created_by`, `updated_at`, `updated_by`) ditangani otomatis melalui field `auditColumns` (default `true`) sehingga tidak perlu dicantumkan pada array `fieldName`. Apabila default behavior aktif, runtime akan menambahkan 4 kolom audit tersebut secara otomatis pada operasi CRUD.

Pada tahap onboarding ini isi file dapat dibiarkan apa adanya. Customisasi `fieldValidation`, `defaultScope`, `fieldNameLookup`, dan fitur lain dapat dilakukan pada skenario pengembangan setelah onboarding selesai.

---

### Langkah 4: Validasi RDF

Perintah `payload validate` melakukan verdict mode (OK/DRIFT/ERROR) per file RDF dengan membandingkan terhadap struktur tabel actual di database.

Jalankan validasi untuk tabel `visitors`:

```bat
npx restforge payload validate --table=visitors
```

Output yang diharapkan:

```
============================================================
SCHEMA VALIDATION - Payload vs Database
============================================================

  Payload directory : payload
  Files to validate : 1

------------------------------------------------------------

  [OK]    visitors.json (visitors)

------------------------------------------------------------

  Summary:
    Total  : 1 payload(s)
    OK     : 1
    Drift  : 0
    Error  : 0
```

Exit code:

| Code | Arti |
|------|------|
| `0` | Semua payload OK |
| `1` | Ada drift atau error |

Setelah RDF baru saja di-generate dari schema yang sama (Langkah 2), kondisi yang diharapkan adalah `[OK]` dengan exit code `0`.

#### Kategori Drift yang Mungkin Terdeteksi

Apabila status `[DRIFT]`, kategori yang dideteksi:

| Kategori | Penyebab |
|----------|----------|
| `Payload field missing from database` | Field di RDF tidak ada di tabel DB |
| `Type change` | Tipe data di `fieldValidation` RDF berbeda dengan kolom DB |
| `New database column` | Kolom DB tidak ada di RDF |
| `Audit column required but missing` | `auditColumns` aktif tapi DB tidak punya kolom audit standar |

---

### Langkah 5 (Opsional): Drift Detection Detail via `payload diff`

Langkah ini bersifat **opsional**. Validasi pada Langkah 4 sudah memberikan verdict (OK/DRIFT/ERROR) per file. Apabila perlu inspect detail per-column, gunakan `payload diff` yang ditujukan sebagai **inspector mode**.

Notasi detail yang ditampilkan:

| Notasi | Arti |
|--------|------|
| `[-] <col>` | Kolom ada di payload tapi tidak di database |
| `[~] <col>` | Tipe kolom berbeda antara payload dan database |
| `[+] <col>` | Kolom ada di database tapi tidak di payload |

Jalankan diff untuk `visitors`:

```bat
npx restforge payload diff --table=visitors
```

Setelah RDF baru saja di-generate (Langkah 2), output diharapkan menampilkan `[OK] visitors.json (visitors) Schema is in sync` tanpa notasi drift.

---

### Langkah 6 (Opsional): Sinkronisasi RDF dengan Database

Langkah ini bersifat **opsional** dan hanya diperlukan apabila pada Langkah 4 atau 5 terdeteksi drift. Perintah `payload sync` menerapkan schema drift dari database ke file payload (update kolom, tipe, dan resolusi `auditColumns`).

Jalankan sync untuk `visitors`:

```bat
npx restforge payload sync --table=visitors
```

Setelah sync, jalankan ulang `payload validate --table=visitors` untuk konfirmasi status `[OK]`.

Catatan dari handbook: matriks resolusi `auditColumns` pada `payload sync` mempertimbangkan kondisi payload dan database. Misal, apabila payload tidak men-set `auditColumns` (default `true`) namun database tidak memiliki 4 kolom audit standar, sync akan otomatis set `"auditColumns": false` dan menampilkan info log:

```
[restforge] Set "auditColumns": false in <file> because audit columns missing in database
```

Pada skenario onboarding ini, tabel `visitors` memiliki 4 kolom audit lengkap sehingga `auditColumns` akan tetap pada default `true` dan proses sinkronisasi tidak akan mengubah isi file payload.

---

## Langkah Berikutnya (Next Step)

Lanjut ke Skenario 6: generate endpoint dan runtime server.

