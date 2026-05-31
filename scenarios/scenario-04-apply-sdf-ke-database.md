# Skenario 4: Apply SDF ke Database dan Drift Detection

> Tahap keempat onboarding RESTForge. Tujuan akhir skenario ini adalah tabel `visitors` ter-create pada database PostgreSQL hasil migrate dari SDF, dan drift detection mengkonfirmasi konsistensi antara SDF dengan struktur tabel actual.

---

## Tujuan (Objective)

Setelah menyelesaikan skenario ini, kondisi berikut harus tercapai:

1. Tabel `visitors` sudah ter-create pada database PostgreSQL yang dikonfigurasi di `config\db-connection.env`
2. Perintah `npx restforge schema list` menampilkan tabel `visitors` pada daftar tabel database
3. Perintah `npx restforge schema describe --table=visitors` menampilkan struktur tabel sesuai SDF
4. Perintah `npx restforge schema diff` mengembalikan exit code `0` (tidak ada drift antara SDF dan database)

---

## Sumber Rujukan (Reference)

Seluruh perintah pada skenario ini bersumber dari handbook berikut:

| Topik | File Handbook |
|-------|---------------|
| Apply SDF ke database (load → validate → DDL → execute) | [commands/schema/migrate.md](https://github.com/restforge/handbook/blob/main/commands/schema/migrate.md) |
| Daftar tabel database via live introspection | [commands/schema/list.md](https://github.com/restforge/handbook/blob/main/commands/schema/list.md) |
| Detail struktur satu tabel | [commands/schema/describe.md](https://github.com/restforge/handbook/blob/main/commands/schema/describe.md) |
| Drift detection bidirectional | [commands/schema/diff.md](https://github.com/restforge/handbook/blob/main/commands/schema/diff.md) |
| Workflow sinkronisasi SDF dengan database | [catalogs/sdf/maintenance/sync-database.md](https://github.com/restforge/handbook/blob/main/catalogs/sdf/maintenance/sync-database.md) |
| Pemakaian default config tanpa flag eksplisit | [commands/conventions.md](https://github.com/restforge/handbook/blob/main/commands/conventions.md) |

---

## Prasyarat (Prerequisite)

| Item | Cara Verifikasi |
|------|-----------------|
| Skenario 3 sudah selesai | File `schema\visitors.js` sudah ter-generate dan lolos `schema validate` |
| Working directory aktif di `sandbox\backend` | Jalankan `cd` pada cmd, path harus berakhiran `\playbook\sandbox\backend` |
| Database PostgreSQL pada `DB_NAME` sudah ada dan dapat diakses | Sudah ter-create pada Skenario 2 (manual atau via prompt auto-create) |
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

Verifikasi default config aktif (sesuai Skenario 2 Langkah 5):

```bat
npx restforge config get-default
```

Output harus menampilkan path `config\db-connection.env`. Command pada resource `payload`, `schema`, dan `query` akan menggunakan default config tersebut apabila flag `--config` tidak disediakan eksplisit. Hal ini memungkinkan seluruh perintah pada Langkah 2 hingga Langkah 6 (migrate, list, describe, diff) dijalankan tanpa `--config`.

---

### Langkah 2 (Opsional): Dry-Run Migrate per File

Langkah ini bersifat **opsional** namun direkomendasikan untuk memahami DDL yang akan dieksekusi sebelum apply ke database. Sesuai Best Practice Pipeline, flag `--dry-run` selalu digunakan terlebih dahulu sebelum apply real ke production.

Flag `--path` menerima path file atau folder schema. Pada skenario onboarding ini, migrate dilakukan **per file** agar setiap tabel ter-apply secara terkontrol dan eksplisit.

Jalankan dry-run untuk file `visitors.js`:

```bat
npx restforge schema migrate --path=schema/visitors.js --dry-run
```

Output mencakup:

- Preview DDL `CREATE TABLE visitors (...)` lengkap dengan kolom, primary key, dan unique constraint
- Preview `CREATE INDEX` untuk field `name`
- Baris akhir: `Dry-run complete. No changes applied.`

Exit code dry-run adalah `2`. Hal ini normal dan tidak menandakan error.

---

### Langkah 3: Apply Migrate per File

Jalankan apply per file tanpa flag `--dry-run`:

```bat
npx restforge schema migrate --path=schema/visitors.js
```

Output sukses menampilkan:

- Ringkasan target koneksi (dialect, host, database, user)
- Daftar statement yang dieksekusi dengan status `✓` per statement
- Pesan sukses pada akhir eksekusi

Apabila database `DB_NAME` belum ada, migrate akan menampilkan prompt auto-create database sesuai mekanisme yang sudah dibahas pada Skenario 2 Langkah 4.

#### Pendekatan Per File vs Per Folder

| Pendekatan | Perintah | Use Case |
|------------|----------|----------|
| Per file (digunakan di skenario ini) | `--path=schema/visitors.js` | Apply satu tabel secara eksplisit. Cocok untuk onboarding bertahap dan rollout terkontrol |
| Per folder | `--path=schema` | Apply seluruh SDF di folder sekaligus. Cocok untuk bootstrap awal database atau rebuild penuh |

Pada onboarding ini pendekatan per file dipilih agar setiap tabel ter-migrate secara terkontrol. Untuk skenario produksi dengan banyak file SDF, pendekatan per folder dapat dipertimbangkan setelah seluruh SDF tervalidasi.

#### Mode Drop (Opsional, Destruktif)

Karakteristik mode drop:

- Mode yang digunakan secara default adalah `--drop=false` dan menghasilkan perintah `CREATE TABLE IF NOT EXISTS` (tanpa ada efek bila data tabel sudah ada isinya)
- Mode `--drop=true` bersifat **destruktif**: tabel dihapus terlebih dahulu sebelum dibuat ulang, **seluruh data pada tabel akan hilang permanen**

Mode drop dapat digunakan apabila perlu rebuild tabel dari nol (mis. setelah modifikasi SDF major yang tidak compatible dengan `ALTER TABLE`

```bat
npx restforge schema migrate --path=schema/visitors.js --drop=true
```

Peringatan: command akan menampilkan baris peringatan `⚠ --drop=true will DROP all tables defined in <path>. Existing data will be lost.` sebelum eksekusi. Jangan jalankan mode ini pada database production tanpa backup terverifikasi.

Untuk skenario onboarding ini, gunakan mode default tanpa `--drop` (tabel `visitors` belum berisi data sehingga mode drop tidak diperlukan).

---

### Langkah 4: Verifikasi Tabel Ter-create via `schema list`

Perintah `schema list` menampilkan daftar tabel pada database secara live introspection (membaca langsung dari database, bukan dari SDF).

Jalankan:

```bat
npx restforge schema list
```

Output JSON pretty-print harus mencantumkan tabel `visitors` sebagai salah satu entri.

Apabila database memiliki banyak tabel dari project lain, gunakan filter schema untuk mempersempit hasil (default schema PostgreSQL adalah `public`):

```bat
npx restforge schema list --schema=public
```

---

### Langkah 5: Verifikasi Struktur Tabel via `schema describe`

Perintah `schema describe` menampilkan detail struktur satu tabel: kolom, primary key, foreign keys, dan indexes.

Jalankan:

```bat
npx restforge schema describe --table=visitors
```

Output JSON pretty-print mencakup informasi berikut:

| Section | Isi yang Diharapkan |
|---------|---------------------|
| Kolom | 8 kolom: `visitor_id`, `name`, `email`, `phone`, `created_at`, `created_by`, `updated_at`, `updated_by` dengan tipe native PostgreSQL |
| Primary Key | `visitor_id` |
| Indexes | Index pada `name`, plus index implisit dari `email` UNIQUE constraint dan `visitor_id` PRIMARY KEY |
| Foreign Keys | Kosong (tabel `visitors` adalah master-data single-table tanpa FK) |

Untuk fokus inspeksi pada kolom saja, gunakan flag untuk menonaktifkan section tertentu:

```bat
npx restforge schema describe --table=visitors --include-foreign-keys=false --include-indexes=false
```

---

### Langkah 6: Drift Detection per File via `schema diff`

Perintah `schema diff` melakukan drift detection bidirectional antara SDF dan struktur tabel actual di database. Bersifat read-only dan aman dijalankan kapan saja.

Konsisten dengan pendekatan per-file pada Langkah 3, jalankan diff untuk file `visitors.js`:

```bat
npx restforge schema diff --path=schema/visitors.js
```

Setelah migrate baru saja sukses (Langkah 3), kondisi yang diharapkan adalah **tidak ada drift**. Exit code: `0`.

Exit Code:

| Code | Arti |
|------|------|
| `0` | Tidak ada drift |
| `1` | Drift terdeteksi |
| `2` | Error (config tidak valid, tabel tidak ditemukan, koneksi gagal, dll.) |

Output human-readable akan menampilkan ringkasan per tabel. Untuk output JSON yang dapat di-parse pada CI/CD pipeline:

```bat
npx restforge schema diff --path=schema/visitors.js --json
```

---

## Langkah Berikutnya (Next Step)

Lanjut ke Skenario 5: generate dan validasi RDF.

