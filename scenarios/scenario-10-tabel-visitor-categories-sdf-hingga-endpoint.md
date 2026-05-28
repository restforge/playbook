# Skenario 10: Tabel visitor_categories dari SDF hingga Endpoint

> Tahap kesepuluh onboarding RESTForge. Menambahkan tabel master baru `visitor_categories` (klasifikasi data `visitors`) dengan mendefinisikan SDF, lalu menjalankan lifecycle penuh: SDF → database → RDF → endpoint REST.

---

## Tujuan (Objective)

1. File `schema\visitor-categories.js` didefinisikan dan lolos `schema validate`
2. Tabel `visitor_categories` ter-create di database via `schema migrate`
3. File `payload\visitor-categories.json` ter-generate dan lolos `payload validate`
4. Endpoint REST `visitor_categories` ter-generate dan dapat di-test via curl

---

## Prasyarat (Prerequisite)

| Item | Cara Verifikasi |
|------|-----------------|
| Skenario 1-6 selesai | Backend `myapp` sudah ter-generate, tabel `visitors` aktif di database |
| Working directory aktif di `sandbox\backend` | Jalankan `cd`, path berakhiran `\playbook\sandbox\backend` |
| Default config sudah di-set | `npx restforge config get-default` menampilkan `config\db-connection.env` |

---

## Sumber Rujukan (Reference)

| Topik | File Handbook |
|-------|---------------|
| Field types (`string`, `text`, `integer`, `boolean`, `timestamp`) | [catalogs/sdf/field-types.md](https://github.com/restforge/handbook/blob/main/catalogs/sdf/field-types.md) |
| Constraint (`pk`, `notnull`, `unique`, `default`) | [catalogs/sdf/constraints.md](https://github.com/restforge/handbook/blob/main/catalogs/sdf/constraints.md) |
| Migrate, list, describe, diff | [commands/schema/](https://github.com/restforge/handbook/blob/main/commands/restforge-backend/schema/) |
| Generate dan validasi RDF | [commands/payload/](https://github.com/restforge/handbook/blob/main/commands/restforge-backend/payload/) |
| Generate endpoint | [commands/endpoint/create.md](https://github.com/restforge/handbook/blob/main/commands/restforge-backend/endpoint/create.md) |

---

## Langkah Eksekusi (Execution Steps)

### Langkah 1: Verifikasi Working Directory

```bat
cd
npx restforge config get-default
```

Path harus berakhiran `\playbook\sandbox\backend` dan default config menampilkan `config\db-connection.env`.

---

### Langkah 2: Buat File SDF

Buat file `schema\visitor-categories.js`:

```bat
notepad schema\visitor-categories.js
```

Isi dengan struktur berikut:

```javascript
module.exports = ({ defineModel }) => defineModel('visitor_categories', {
  fields: {
    category_id:            'string:36 pk',
    category_code:                   'string:50 unique notnull',
    category_name:                   'string:100 notnull',
    description:            'text',
    default_duration_hours: 'integer default:8',
    requires_escort:        'boolean default:false',
    is_active:              'boolean default:true',
    created_at:             'timestamp default:now()',
    created_by:             'string:100',
    updated_at:             'timestamp',
    updated_by:             'string:100'
  }
});
```

Catatan: constraint `unique` pada `category_code` sudah menghasilkan index implisit, sehingga array `indexes` tidak diperlukan.

> **Catatan konvensi penamaan:** file SDF memakai kebab-case (`visitor-categories.js`), sedangkan nama model pada `defineModel` beserta seluruh field memakai snake_case (`visitor_categories`, `category_id`). Pola ini konsisten di seluruh playbook — semua file definisi (SDF `.js` dan payload `.json`) memakai kebab-case, sementara identifier database (nama tabel dan kolom) memakai snake_case.

---

### Langkah 3: Validasi SDF

```bat
npx restforge schema validate --path=schema
npx restforge schema models --path=schema
```

`schema validate` tidak menampilkan error. `schema models` menampilkan model `visitor_categories` dengan 11 fields, PK `category_id`, unique `category_code`. Model `visitors` (Skenario 3) tetap muncul tanpa terpengaruh.

---

### Langkah 4: Migrate SDF ke Database

Dry-run (opsional) lalu apply per file:

```bat
npx restforge schema migrate --path=schema\visitor-categories.js --dry-run
npx restforge schema migrate --path=schema\visitor-categories.js
```

Dry-run menampilkan preview `CREATE TABLE visitor_categories (...)` (exit code `2`, normal). Apply menampilkan status `✓` per statement.

---

### Langkah 5: Verifikasi Tabel di Database

```bat
npx restforge schema list
npx restforge schema describe --table=visitor_categories
npx restforge schema diff --path=schema\visitor-categories.js
```

| Perintah | Kondisi |
|----------|---------|
| `schema list` | Mencantumkan `visitor_categories` |
| `schema describe` | 11 kolom, PK `category_id`, unique `category_code` |
| `schema diff` | Exit code `0` (tidak ada drift) |

---

### Langkah 6: Generate dan Validasi RDF

```bat
npx restforge payload generate --table=visitor_categories
npx restforge payload validate --table=visitor_categories
```

File `payload\visitor-categories.json` ter-generate. Validasi mengembalikan `[OK]` dengan exit code `0`.

> **Catatan:** sesuai konvensi penamaan (Langkah 2), argumen `--table=visitor_categories` (snake_case) menghasilkan file `visitor-categories.json` (kebab-case) secara otomatis. Property `tableName` di dalam RDF tetap snake_case mengikuti nama tabel database, bukan nama file.

#### Konfigurasi Lookup (Opsional, untuk Skenario 11)

`fieldNameLookup` adalah property top-level RDF (sejajar dengan `tableName`, `fieldName`, `action`) yang mengatur kolom `id` dan `text` pada response endpoint `/lookup`. Berguna saat tabel ini menjadi sumber dropdown kategori di Skenario 11.

Property ini **disisipkan ke dalam object `payload\visitor-categories.json` yang sudah ada** — bukan file atau object baru. Posisi yang umum adalah tepat sebelum `action`:

```json
{
    "tableName": "visitor_categories",
    "primaryKey": "category_id",
    "fieldName": [ ... ],
    "fieldValidation": [ ... ],
    "fieldNameLookup": {
        "id": "category_id",
        "text": "category_code||' - '||category_name as display_text"
    },
    "action": { ... }
}
```

Bagian `[ ... ]` dan `{ ... }` mewakili isi existing yang tidak diubah; hanya blok `fieldNameLookup` yang ditambahkan. Operator concatenation `||` (PostgreSQL) di-translate otomatis ke `CONCAT()` pada MySQL.

---

### Langkah 7: Generate Endpoint REST

```bat
npx restforge endpoint create --project=myapp --name=visitor-categories --payload=visitor-categories.json
```

Validasi schema payload-vs-database lolos (RDF baru saja di-generate dari tabel yang sama). Endpoint ter-generate beserta example files curl/Postman/Insomnia.

---

### Langkah 8: Test Endpoint via curl

Jalankan server pada cmd terpisah:

```bat
npx restforge serve --project=myapp --config=db-connection.env
```

Log harus menampilkan `Loading 2 endpoint(s)` (`visitors` + `visitor_categories`) dan `[OK] Server ready on port 3000`.

Pada cmd lain, eksekusi demo create lalu datatables:

```bat
cd playbook\sandbox\backend\examples\myapp\visitor-categories\curl
demo-create.bat
demo-datatables.bat
```

`demo-create.bat` mengembalikan `success: true` dengan `category_id` UUID v7. `demo-datatables.bat` menampilkan record tersebut pada array `data`.

Stop server via `Ctrl + C` setelah selesai.

---

## Kriteria Selesai (Completion Criteria)

| Item | Kondisi |
|------|---------|
| `schema\visitor-categories.js` | Didefinisikan, lolos `schema validate` |
| `schema migrate` | Sukses, tabel `visitor_categories` ter-create |
| `schema diff` | Exit code `0` |
| `payload\visitor-categories.json` | Ter-generate, `payload validate` status `[OK]` |
| `endpoint create` | Sukses exit code `0`, folder `examples\myapp\visitor-categories\` terbentuk |
| Server | `Loading 2 endpoint(s)`, `demo-create.bat` + `demo-datatables.bat` sukses |

---

## Catatan untuk Tahap Berikutnya (Notes for Next Stage)

Skenario 11 merevisi SDF `visitors` agar memiliki foreign key ke `visitor_categories`, menerapkan perubahan via `schema apply` (additive), lalu meng-generate ulang RDF `visitors.json` dengan tampilan JOIN.

---

## Pelaporan Issue (Issue Reporting)

Apabila ditemukan kondisi tidak sesuai ekspektasi, hentikan eksekusi dan dokumentasikan output lengkap perintah yang gagal beserta nomor langkah. Sertakan isi `schema\visitor-categories.js` bila terkait validasi SDF, atau `payload\visitor-categories.json` bila terkait validasi RDF.
