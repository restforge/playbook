# Skenario 3: Pembuatan dan Validasi SDF

> Tahap ketiga onboarding RESTForge. Tujuan akhir skenario ini adalah satu file SDF (Schema Definition File) untuk tabel master `visitors` dibuat dari katalog template RESTForge, lolos validasi, dan menghasilkan DDL preview yang valid untuk dialect PostgreSQL.

---

## Tujuan (Objective)

Setelah menyelesaikan skenario ini, kondisi berikut harus tercapai:

1. Folder `schema/` sudah terbentuk di working directory `sandbox\backend`
2. File `schema/visitors.js` sudah ter-generate dari katalog template RESTForge (domain `generic`)
3. Perintah `npx restforge schema validate --path=schema` mengembalikan status sukses
4. Perintah `npx restforge schema models --path=schema` menampilkan ringkasan struktur model

Catatan: skenario ini hanya mencakup pembuatan dan validasi SDF di sisi filesystem. Apply ke database (`schema migrate`) akan dibahas pada skenario terpisah. Preview DDL via `schema generate-ddl` tersedia sebagai langkah opsional di akhir skenario untuk konteks edukatif.

---

## Sumber Rujukan (Reference)

Seluruh perintah dan struktur SDF pada skenario ini bersumber dari handbook berikut:

| Topik | File Handbook |
|-------|---------------|
| Resource `schema` dan daftar sub-command | [commands/schema/README.md](https://github.com/restforge/handbook/blob/main/commands/schema/README.md) |
| Katalog template (87 template, 30+ domain) | [commands/schema/template.md](https://github.com/restforge/handbook/blob/main/commands/schema/template.md) |
| Scaffold SDF dari template `dummy` (alternatif) | [commands/schema/init.md](https://github.com/restforge/handbook/blob/main/commands/schema/init.md) |
| Validasi SDF | [commands/schema/validate.md](https://github.com/restforge/handbook/blob/main/commands/schema/validate.md) |
| Daftar models dengan structural summary | [commands/schema/models.md](https://github.com/restforge/handbook/blob/main/commands/schema/models.md) |
| Generate DDL preview | [commands/schema/generate-ddl.md](https://github.com/restforge/handbook/blob/main/commands/schema/generate-ddl.md) |
| Anatomi `defineModel()` dan struktur file | [catalogs/sdf/README.md](https://github.com/restforge/handbook/blob/main/catalogs/sdf/README.md) |
| Konvensi penamaan tabel, field, constraint | [catalogs/sdf/naming-convention.md](https://github.com/restforge/handbook/blob/main/catalogs/sdf/naming-convention.md) |
| Sintaks shorthand field | [catalogs/sdf/field-shorthand.md](https://github.com/restforge/handbook/blob/main/catalogs/sdf/field-shorthand.md) |
| Sistem tipe (string, integer, decimal, dst.) | [catalogs/sdf/field-types.md](https://github.com/restforge/handbook/blob/main/catalogs/sdf/field-types.md) |
| Sistem constraint (pk, notnull, unique, dst.) | [catalogs/sdf/constraints.md](https://github.com/restforge/handbook/blob/main/catalogs/sdf/constraints.md) |

---

## Prasyarat (Prerequisite)

| Item | Cara Verifikasi |
|------|-----------------|
| Skenario 2 sudah selesai | File `config\db-connection.env` sudah terisi dan default config sudah di-set |
| Working directory aktif di `sandbox\backend` | Jalankan `cd` pada cmd, path harus berakhiran `\playbook\sandbox\backend` |

---

## Langkah Eksekusi (Execution Steps)

### Langkah 1: Verifikasi Working Directory

Pastikan posisi cmd berada di folder `sandbox\backend`:

```bat
cd
```

Output harus menunjukkan path yang berakhiran `\playbook\sandbox\backend`. Jika tidak, pindah dari folder root project:

```bat
cd playbook\sandbox\backend
```

---

### Langkah 2: Eksplorasi Katalog Domain

RESTForge menyediakan katalog template SDF siap pakai yang dikelompokkan per domain aplikasi (generic, erp, finance, hr, retail, hospitality, dan domain lainnya). Daftar domain dapat dilihat via flag `--list-domains`.

Jalankan:

```bat
npx restforge schema template --list-domains
```

Output menampilkan tabel domain berisi kolom `Code`, `Name`, dan `Group`. Kolom `Code` adalah nilai yang dipakai sebagai filter pada perintah berikutnya (mis. `generic`, `erp`, `finance`).

Output juga menampilkan total jumlah domain pada baris terakhir.

---

### Langkah 3: Preview Struktur SDF Target

Setelah memilih domain (pada skenario ini menggunakan `generic`) dan nama tabel target (`visitors`), preview struktur SDF tanpa men-generate file via flag `--show`:

```bat
npx restforge schema template --domain=generic --table=visitors --show
```

Output yang ditampilkan adalah blok kode JavaScript SDF lengkap. Struktur yang ditampilkan mengikuti pattern dari handbook `catalogs/sdf/README.md`:

```javascript
module.exports = ({ defineModel }) => defineModel('visitors', {
  fields: {
    visitor_id: 'string:36 pk',
    name:       'string:100 notnull',
    email:      'string:100 unique notnull',
    phone:      'string:20 notnull',
    created_at: 'timestamp default:now()',
    created_by: 'string:100',
    updated_at: 'timestamp',
    updated_by: 'string:100'
  },
  indexes: [
    'name'
  ]
});
```

Pemetaan komponen pada struktur tersebut (bersumber dari `catalogs/sdf/field-shorthand.md`, `field-types.md`, dan `constraints.md`):

| Field | Shorthand | Penjelasan |
|-------|-----------|-----------|
| `visitor_id` | `string:36 pk` | String length 36 (UUID), primary key. Constraint `pk` otomatis memberi `notnull` |
| `name` | `string:100 notnull` | Nama pengunjung, wajib diisi |
| `email` | `string:100 unique notnull` | Email pengunjung, unique dan wajib diisi |
| `phone` | `string:20 notnull` | Nomor telepon, wajib diisi |
| `created_at` | `timestamp default:now()` | Audit field dengan default fungsi `now()` (diterjemahkan per dialect: PostgreSQL menjadi `CURRENT_TIMESTAMP`) |
| `created_by` | `string:100` | Identitas pembuat record |
| `updated_at` | `timestamp` | Audit field timestamp tanpa default. Nilai di-set oleh layer aplikasi pada saat update record |
| `updated_by` | `string:100` | Identitas pemutakhir record |

---

### Langkah 4: Pahami Contoh Data dan Use Case

Tambahkan flag `--example` pada perintah preview untuk menampilkan metadata template, deskripsi, contoh data, dan catatan penting terkait constraint dan use case bisnis:

```bat
npx restforge schema template --domain=generic --table=visitors --show --example
```

Output mencakup beberapa section:

| Section | Isi |
|---------|-----|
| Metadata | File source template, category (`master-data`), pattern (`single-table`), section (`reception-guest`), domains terkait |
| Deskripsi | Penjelasan fungsi tabel (mis. master pengunjung dengan email unik dan kontak dasar) |
| Penjelasan kolom utama | Anotasi per field beserta tipe data dan constraint |
| Tabel referensi contoh data | Sample row data realistis sebagai panduan pengisian |
| Catatan penting | Constraint penting (mis. UNIQUE pada email), NOT NULL fields, dan rekomendasi penggunaan |

Section ini membantu memahami konteks bisnis dari tabel sebelum men-generate file SDF ke filesystem.

---

### Langkah 5: Generate File SDF dari Template

Flag `--generate` digunakan untuk menulis template ke filesystem. Path destination diatur via `--path`. Format default `--lang` adalah `sdf` sehingga flag tersebut tidak perlu ditulis eksplisit.

Jalankan generate:

```bat
npx restforge schema template --domain=generic --table=visitors --generate --path=schema\visitors.js
```

Output sukses menampilkan ringkasan file yang ter-generate:

```
Generated 1 file dari template 'visitors':
  [OK] schema\visitors.js (<ukuran> bytes)
```

Verifikasi file ter-generate:

```bat
dir schema\visitors.js
```

Output `dir` harus menampilkan entri file tanpa error.

Catatan dari handbook:

- File destination yang sudah ada akan menyebabkan error. Apabila perlu overwrite, tambahkan flag `--force`
- Master-detail template (mis. `sales_order` + `sales_order_item`) akan menghasilkan dua file SDF dengan satu pemanggilan `--generate`. Tabel `visitors` adalah single-table sehingga hanya menghasilkan satu file

#### Catatan Alternatif: `schema init`

Apabila tujuan hanya scaffold cepat dengan template `dummy` (master-data generic), tersedia shortcut `npx restforge schema init --path=<PATH>` sesuai handbook `commands/schema/init.md`. Shortcut ini setara dengan `schema template --table=dummy --generate --lang=sdf --path=<PATH>`. Pada skenario ini tidak digunakan karena katalog `schema template` memberi pilihan template lebih relevan dengan domain bisnis.

---

### Langkah 6: Inspeksi File yang Ter-generate

Buka file hasil generate untuk inspeksi visual:

```bat
code schema\visitors.js
```

Pastikan isi file sesuai dengan struktur SDF berikut:

```javascript
module.exports = ({ defineModel }) => defineModel('visitors', {
  fields: {
    visitor_id: 'string:36 pk',
    name:       'string:100 notnull',
    email:      'string:100 unique notnull',
    phone:      'string:20 notnull',
    created_at: 'timestamp default:now()',
    created_by: 'string:100',
    updated_at: 'timestamp',
    updated_by: 'string:100'
  },
  indexes: [
    'name'
  ]
});
```

Apabila ada perbedaan minor pada shorthand field `updated_at` antara file hasil generate dengan referensi di atas, samakan dengan referensi sebelum menutup file. Setelah selesai, simpan file dan lanjutkan ke langkah validasi.

---

### Langkah 7: Validasi SDF

Perintah `schema validate` melakukan single-model validation dan cross-model consistency check pada seluruh file di folder schema.

Jalankan validasi:

```bat
npx restforge schema validate --path=schema
```

Output sukses tidak menampilkan error. Apabila ada kesalahan sintaks atau shorthand yang tidak dikenal, command akan menampilkan pesan error dengan lokasi file dan field yang bermasalah.

---

### Langkah 8: Tampilkan Ringkasan Model

Perintah `schema models` menampilkan daftar models dengan structural summary (fields, primary key, indexes, uniques, relations). Berguna untuk verifikasi cepat bahwa parser memahami struktur SDF sesuai ekspektasi.

Jalankan:

```bat
npx restforge schema models --path=schema
```

Output harus menampilkan satu model `visitors` dengan ringkasan: 8 fields, primary key `visitor_id`, index pada `name`, dan unique constraint pada `email` (dari shorthand `unique`).

---

### Langkah 9 (Opsional): Preview DDL untuk PostgreSQL

Langkah ini bersifat **opsional** dan **edukatif**. Validitas SDF sudah dipastikan melalui Langkah 7 (`schema validate`) dan Langkah 8 (`schema models`), sehingga preview DDL bukan prasyarat untuk Skenario 4. Apply DDL ke database akan otomatis dilakukan oleh `schema migrate` pada skenario berikutnya.

Tujuan menjalankan langkah ini meliputi:

- Memahami bagaimana shorthand logical type (mis. `boolean`, `timestamp`) diterjemahkan ke tipe SQL native per dialect
- Verifikasi awal struktur DDL sebelum migrate
- Generate file DDL terpisah (via `--out=<FILE>`) untuk review tim atau eksekusi manual

Perintah `schema generate-ddl` menghasilkan DDL SQL (`CREATE TABLE`, `CREATE INDEX`, opsional `DROP`) untuk seluruh schema models. Output default ke stdout, atau ke file via `--out=<FILE>`. Perintah ini termasuk read-only dan aman dijalankan kapan saja.

Preview ke stdout dengan dialect PostgreSQL:

```bat
npx restforge schema generate-ddl --path=schema --dialect=postgres
```

Output yang diharapkan mencakup:

- Satu `CREATE TABLE visitors (...)` dengan kolom sesuai shorthand SDF
- Konstrain `PRIMARY KEY` pada `visitor_id`
- Konstrain `UNIQUE` pada `email` (sumber: shorthand `unique`)
- Statement `CREATE INDEX` untuk `name` (constraint `unique` pada `email` sudah implisit menghasilkan index sehingga tidak dibuat dua kali, sesuai aturan deduplikasi pada `catalogs/sdf/constraints.md`)

Untuk menyimpan DDL ke file (opsional):

```bat
npx restforge schema generate-ddl --path=schema --dialect=postgres --out=schema\visitors-postgres.sql
```

Catatan dari handbook `catalogs/sdf/README.md`: tipe shorthand bersifat logical bukan physical. DDL generator memetakan ke tipe SQL native sesuai dialect (mis. `boolean` di PostgreSQL menjadi `BOOLEAN`, di MySQL menjadi `VARCHAR(5)` menyimpan `'true'`/`'false'`).

---

## Langkah Berikutnya (Next Step)

Lanjut ke Skenario 4: apply SDF ke database.

