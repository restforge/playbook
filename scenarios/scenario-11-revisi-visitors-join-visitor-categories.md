# Skenario 11: Revisi visitors dengan JOIN ke visitor_categories

> Tahap kesebelas onboarding RESTForge. Merevisi SDF `visitors` agar memiliki foreign key `category_id` ke `visitor_categories`, menerapkan perubahan ke database via `schema apply` (additive, non-destruktif), lalu meng-generate ulang RDF `visitors.json` dengan tampilan JOIN agar `category_name` muncul di endpoint list dan detail.

---

## Tujuan (Objective)

1. SDF `schema\visitors.js` ditambah field FK `category_id` dan lolos `schema validate`
2. Kolom `category_id` ter-tambah ke tabel `visitors` via `schema apply` (data existing tetap aman)
3. RDF `payload\visitors.json` di-generate ulang lalu dilengkapi kolom `category_code` & `category_name` pada `fieldName`, serta property `datatablesQuery`, `viewQuery`, dan `datatablesWhere` untuk JOIN ke `visitor_categories`
4. Endpoint `visitors` di-generate ulang dan endpoint `/datatables` mengembalikan kolom `category_code` dan `category_name`

---

## Prasyarat (Prerequisite)

| Item | Cara Verifikasi |
|------|-----------------|
| Skenario 10 selesai | Tabel dan endpoint `visitor_categories` aktif; `schema\visitor-categories.js` ada di folder `schema\` |
| Working directory aktif di `sandbox\backend` | Jalankan `cd`, path berakhiran `\playbook\sandbox\backend` |
| Default config sudah di-set | `npx restforge config get-default` menampilkan `config\db-connection.env` |

---

## Sumber Rujukan (Reference)

| Topik | File Handbook |
|-------|---------------|
| FK shorthand `fk:TABLE.COLUMN` | [catalogs/sdf/constraints.md](https://github.com/restforge/handbook/blob/main/catalogs/sdf/constraints.md) |
| FK lanjutan dan auto-promotion | [catalogs/sdf/foreign-keys.md](https://github.com/restforge/handbook/blob/main/catalogs/sdf/foreign-keys.md) |
| ALTER incremental additive | [commands/schema/apply.md](https://github.com/restforge/handbook/blob/main/commands/restforge-backend/schema/apply.md) |
| `datatablesQuery`, `viewQuery`, prioritas data source | [catalogs/rdf/data-source.md](https://github.com/restforge/handbook/blob/main/catalogs/rdf/data-source.md) |
| `fieldNameLookup` | [catalogs/rdf/field-lookup.md](https://github.com/restforge/handbook/blob/main/catalogs/rdf/field-lookup.md) |
| Referensi SQL via `file:` prefix | [catalogs/rdf/file-reference.md](https://github.com/restforge/handbook/blob/main/catalogs/rdf/file-reference.md) |
| Generate ulang endpoint dengan `--force` | [commands/endpoint/create.md](https://github.com/restforge/handbook/blob/main/commands/restforge-backend/endpoint/create.md) |

---

## Langkah Eksekusi (Execution Steps)

### Langkah 1: Verifikasi Working Directory

```bat
cd
npx restforge config get-default
```

Path harus berakhiran `\playbook\sandbox\backend` dan default config menampilkan `config\db-connection.env`.

---

### Langkah 2: Revisi SDF visitors

Buka `schema\visitors.js`:

```bat
notepad schema\visitors.js
```

Tambahkan field `category_id` dengan FK shorthand ke `visitor_categories`:

```javascript
module.exports = ({ defineModel }) => defineModel('visitors', {
  fields: {
    visitor_id:  'string:36 pk',
    name:        'string:100 notnull',
    email:       'string:100 unique notnull',
    phone:       'string:20 notnull',
    category_id: 'string:36 fk:visitor_categories.category_id',
    created_at:  'timestamp default:now()',
    created_by:  'string:100',
    updated_at:  'timestamp',
    updated_by:  'string:100'
  },
  indexes: [
    'name'
  ]
});
```

Catatan: `category_id` dibiarkan nullable (tanpa `notnull`) agar record `visitors` existing yang belum memiliki kategori tetap valid saat kolom ditambahkan.

#### Pilihan Penulisan FK: Inline vs relations

SDF menyediakan dua cara mendeklarasikan foreign key yang menghasilkan FK constraint **identik**. Playbook ini memakai bentuk **inline** karena paling ringkas dan target table disebut langsung. Bentuk **`relations`** tersedia sebagai alternatif untuk kebutuhan lanjutan.

**Opsi A — Inline `fk:` shorthand (dipakai playbook):**

```javascript
fields: {
  category_id: 'string:36 fk:visitor_categories.category_id'
}
```

**Opsi B — Property `relations`:**

```javascript
fields: {
  category_id: 'string:36'
},
relations: {
  category: {
    type: 'belongsTo',
    target: 'visitor_categories',
    localKey: 'category_id',
    references: 'category_id'
    // onDelete: 'setNull',  // opsional, hanya tersedia di bentuk relations
    // onUpdate: 'cascade'
  }
}
```

Perbandingan:

| Aspek | Inline `fk:` | `relations` |
|-------|--------------|-------------|
| FK constraint hasil | `fk_visitors_category ... REFERENCES visitor_categories (category_id)` | Sama persis |
| Auto-index | `idx_visitors_category_id` | Sama |
| Penyebutan target table | Langsung di shorthand | Diturunkan dari **nama key relation**; bila key (mis. `category`) ≠ nama tabel, wajib set `target` eksplisit |
| `onDelete` / `onUpdate` | Tidak didukung | Didukung (`cascade`, `restrict`, `setNull`, `noAction`) |
| Relasi `hasOne` / `hasMany` | Tidak didukung | Didukung |
| Keringkasan | Lebih ringkas | Lebih verbose |

Aturan penting:

- Internal-nya, inline `fk:` di-auto-promote menjadi entry `relations` — keduanya setara secara fungsional.
- Satu field tidak boleh memakai kedua cara sekaligus (validator menolak dengan error eksplisit).
- Pemilihan bentuk tidak mempengaruhi keterbatasan `schema apply` pada Langkah 3: FK constraint tetap `deferred` di jalur incremental, terlepas dari cara deklarasi.

Pilih `relations` apabila diperlukan behavior `onDelete`/`onUpdate`, nama relation custom, atau relasi inverse `hasOne`/`hasMany`. Untuk FK sederhana, inline `fk:` sudah memadai.

Validasi (cross-model resolve referensi FK ke `visitor_categories`):

```bat
npx restforge schema validate --path=schema
```

Tidak menampilkan error. Bila `schema\visitor-categories.js` tidak ada di folder `schema\`, validator menolak referensi `fk:visitor_categories.category_id` — pastikan Skenario 10 sudah selesai.

---

### Langkah 3: Apply Perubahan ke Database

Tabel `visitors` sudah ada di database, sehingga perubahan diterapkan secara incremental via `schema apply` (ADD COLUMN), bukan `schema migrate` yang memakai `CREATE TABLE IF NOT EXISTS`.

Dry-run lalu apply:

```bat
npx restforge schema apply --path=schema\visitors.js --config=db-connection.env --table=visitors --dry-run
npx restforge schema apply --path=schema\visitors.js --config=db-connection.env --table=visitors
```

Dry-run menampilkan preview `ALTER TABLE visitors ADD COLUMN category_id ...`. Apply mengeksekusi penambahan kolom; data `visitors` existing tetap utuh.

> **Catatan penting (terverifikasi dari handbook):** `schema apply` bersifat additive konservatif. Penambahan **kolom** `category_id` didukung, namun **penambahan FK constraint fisik termasuk operasi `deferred`** dan di-skip oleh `schema apply` (lihat `apply.md` section "Operasi yang Belum Didukung"). Konsekuensi: relasi terbentuk pada level kolom dan didukung penuh oleh JOIN di RDF (Langkah 5), tanpa FK constraint fisik di database. Apabila FK constraint fisik mutlak diperlukan, satu-satunya jalur saat ini adalah `schema migrate --path=schema\visitors.js --drop=true` yang bersifat **destruktif** (seluruh data `visitors` hilang) dan berada di luar scope skenario ini.

Verifikasi kolom ter-tambah:

```bat
npx restforge schema describe --table=visitors
```

Output menampilkan kolom `category_id` pada struktur tabel `visitors`.

---

### Langkah 4: Sinkronkan RDF dengan Kolom Baru

Tabel `visitors` kini memiliki kolom `category_id` (Langkah 3), sementara RDF `payload\visitors.json` dari Skenario 5 belum memuatnya — kondisi ini terdeteksi sebagai drift. RDF disinkronkan via `payload sync` yang mengarsipkan file lama sebelum menambahkan kolom baru, sehingga aman dijalankan berulang dan kustomisasi tetap terjaga.

Deteksi drift lalu sync:

```bat
npx restforge payload validate --table=visitors
npx restforge payload sync --table=visitors
```

`payload validate` melaporkan `[DRIFT] visitors.json (visitors) — 1 new database column(s) not in payload`. `payload sync` mengarsipkan baseline lama menjadi `visitors.json.archive.001` (`[ARCHIVE]`), lalu menambahkan `category_id` ke array `fieldName` (`[SYNCED]`).

Verifikasi hasil sync:

```bat
npx restforge payload validate --table=visitors
```

Status kembali `[OK]` karena payload kini selaras dengan struktur tabel.

#### Pendekatan Alternatif: regenerate baseline penuh

`payload generate` dapat menimpa `visitors.json` existing secara langsung — tanpa perlu menghapus file lebih dulu — dan menghasilkan baseline penuh dari struktur database terkini:

```bat
npx restforge payload generate --table=visitors
npx restforge payload validate --table=visitors
```

Perbedaan dengan `sync`: `generate` menulis ulang seluruh file (kustomisasi apa pun pada RDF akan hilang) dan tidak membuat archive, sedangkan `sync` hanya menambahkan kolom yang belum ada dan mengarsipkan versi lama. Pada langkah ini `visitors.json` masih baseline murni — kustomisasi JOIN baru ditambahkan pada Langkah 5 — sehingga kedua pendekatan menghasilkan array `fieldName` akhir yang identik. Pilih `generate` untuk reset bersih, atau `sync` (pendekatan utama) bila preservasi kustomisasi dan backup diutamakan.

---

### Langkah 5: Tambahkan Konfigurasi JOIN ke RDF

Buat folder query dan file SQL JOIN. Kolom `category_code` dan `category_name` diambil dari `visitor_categories` melalui LEFT JOIN; nama keduanya sudah unik terhadap kolom `visitors` (`visitor_id`, `name`, `email`, `phone`, `category_id`), sehingga tidak memerlukan alias.

Buat file `payload\query\visitors-join.sql`:

```bat
mkdir payload\query
notepad payload\query\visitors-join.sql
```

Isi:

```sql
SELECT v.visitor_id,
       v.name,
       v.email,
       v.phone,
       v.category_id,
       vc.category_code,
       vc.category_name
FROM visitors v
LEFT JOIN visitor_categories vc ON vc.category_id = v.category_id
```

Buka `payload\visitors.json`:

```bat
notepad payload\visitors.json
```

Lakukan dua penyesuaian **pada object `payload\visitors.json` yang sudah ada** (bukan file atau object baru).

**1. Tambahkan `category_code` dan `category_name` ke array `fieldName`.** Kedua kolom berasal dari JOIN ke `visitor_categories`, bukan kolom fisik tabel `visitors`. Karena itu keduanya ditambahkan ke `fieldName` (agar muncul di response endpoint), namun **tidak** ke `fieldValidation` — array tersebut hanya memuat kolom fisik tabel.

**2. Tambahkan property data source berikut**, sejajar dengan `tableName` dan `fieldName`:

- `datatablesQuery` — melayani endpoint `/datatables`
- `viewQuery` — melayani `/read`, `/first`, `/lookup` (sesuai prioritas resolusi `viewName → viewQuery → tableName`)
- `datatablesWhere` — daftar kolom yang dapat dicari di `/datatables`; entri `"all"` mengaktifkan pencarian lintas-kolom pada seluruh kolom yang terdaftar
- `fieldNameLookup` (opsional) — mengatur teks display saat `visitors` menjadi sumber lookup; dapat dilewati bila endpoint `visitors` tidak dipakai sebagai sumber dropdown

Hasil object `payload\visitors.json`:

```json
{
    "tableName": "visitors",
    "primaryKey": "visitor_id",
    "fieldName": [
        "visitor_id",
        "name",
        "email",
        "phone",
        "category_id",
        "category_code",
        "category_name"
    ],
    "datatablesQuery": "file:query/visitors-join.sql",
    "viewQuery": "file:query/visitors-join.sql",
    "datatablesWhere": [
        "name",
        "email",
        "phone",
        "category_code",
        "category_name",
        "all"
    ],
    "fieldNameLookup": {
        "id": "visitor_id",
        "text": "name"
    },
    "action": { ... },
    "fieldValidation": [ ... ]
}
```

Bagian `{ ... }` dan `[ ... ]` pada `action` dan `fieldValidation` mewakili isi existing yang tidak diubah. Property `fieldNameLookup` bersifat opsional — tidak diperlukan agar `category_code`/`category_name` muncul — dan dapat dihapus bila tidak dibutuhkan. `LEFT JOIN` pada file SQL dipilih agar record `visitors` tanpa `category_id` tetap muncul (kolom kategori bernilai `NULL`).

Validasi ulang RDF:

```bat
npx restforge payload validate --table=visitors
```

Status `[OK]`. Validasi SQL keyword dijalankan saat `endpoint create` (Langkah 6).

---

### Langkah 6: Generate Ulang Endpoint visitors

Endpoint `visitors` sudah ada dari Skenario 6, sehingga flag `--force` diperlukan untuk menimpa file existing:

```bat
npx restforge endpoint create --project=myapp --name=visitors --payload=visitors.json --force
```

Generator membaca isi `visitors-join.sql` dan menyisipkannya sebagai SQL literal di module hasil generate. Validasi schema payload-vs-database lolos karena `category_id` sudah ada di tabel.

---

### Langkah 7: Verifikasi JOIN via curl

Jalankan server pada cmd terpisah:

```bat
npx restforge serve --project=myapp --config=db-connection.env
```

Pada cmd lain, uji `/datatables`:

```bat
cd playbook\sandbox\backend\examples\myapp\visitors\curl
demo-datatables.bat
```

Setiap record pada array `data` kini memuat field `category_id`, `category_code`, dan `category_name`. Record tanpa kategori menampilkan nilai `null` pada field kategori (efek `LEFT JOIN`).

Untuk menguji record dengan kategori terisi: ambil `category_id` valid dari endpoint `visitor_categories` (`demo-create.bat` Skenario 10), buat/update record `visitors` dengan `category_id` tersebut, lalu jalankan ulang `demo-datatables.bat` dan pastikan `category_name` terisi.

Stop server via `Ctrl + C` setelah selesai.

---

## Kriteria Selesai (Completion Criteria)

| Item | Kondisi |
|------|---------|
| `schema\visitors.js` | Field `category_id` dengan `fk:visitor_categories.category_id`, lolos `schema validate` |
| `schema apply` | Kolom `category_id` ter-tambah, data existing aman |
| `schema describe --table=visitors` | Menampilkan kolom `category_id` |
| `payload\visitors.json` | `fieldName` memuat `category_code` & `category_name`; ada `datatablesQuery`, `viewQuery`, `datatablesWhere`; `payload validate` `[OK]` |
| `payload\query\visitors-join.sql` | Berisi SQL JOIN yang men-SELECT `category_code` dan `category_name` dari `visitor_categories` |
| `endpoint create --force` | Sukses exit code `0` |
| `demo-datatables.bat` | Response memuat `category_code` dan `category_name` |

---

## Catatan untuk Tahap Berikutnya (Notes for Next Stage)

Endpoint `visitors` kini menyajikan data kategori. Untuk menampilkannya di aplikasi frontend (Skenario 8-9), jalankan ulang `npx restforge payload migrate` untuk `visitors.json` lalu `restforge-designer generate --overwrite` agar UDF dan aplikasi ter-update dengan kolom kategori dan dropdown lookup `visitor_categories`.

---

## Pelaporan Issue (Issue Reporting)

Apabila ditemukan kondisi tidak sesuai ekspektasi, hentikan eksekusi dan dokumentasikan output lengkap perintah yang gagal beserta nomor langkah. Untuk error JOIN, sertakan isi `payload\query\visitors-join.sql` dan `payload\visitors.json`. Untuk error apply, sertakan output `schema apply --dry-run` dan `schema describe --table=visitors`.
