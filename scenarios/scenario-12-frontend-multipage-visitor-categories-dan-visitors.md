# Skenario 12: Aplikasi Frontend Multi-Page visitor_categories dan visitors

> Tahap kedua belas onboarding RESTForge. Menggabungkan dua resource (`visitor_categories` dan `visitors`) ke dalam satu aplikasi frontend multi-page. Command `payload migrate` melakukan auto-discovery tabel JOIN: migrasi RDF `visitors.json` (yang sudah JOIN ke `visitor_categories` pada Skenario 11) menghasilkan UDF multi-page secara otomatis, lengkap dengan dropdown kategori dan navigasi sidebar. Proses berurutan tetap berlaku: tambah kategori terlebih dahulu, lalu create/update visitor dengan memilih kategori tersebut.

---

## Tujuan (Objective)

1. RDF `visitors.json` (dengan JOIN ke `visitor_categories`) di-migrate menjadi UDF multi-page secara otomatis melalui auto-discovery JOIN, tanpa penyusunan manual
2. Hasil migrate berbentuk struktur split: `app-config.json` + `pages\visitor-categories.json` + `pages\visitors.json` + aggregator `visitors-app.json` dengan sidebar navigasi dua page
3. Field `category_id` pada halaman `visitors` otomatis ter-konfigurasi sebagai dropdown `select` yang memuat opsi dari endpoint `visitor-categories/lookup`
4. Aplikasi frontend ter-generate ke `sandbox\frontend\apps\visitors-app` dengan halaman `visitor-categories.html` dan `visitors.html`
5. Verifikasi end-to-end berurutan di browser: tambah kategori → create/update visitor (pilih kategori) → list `visitors` menampilkan `category_name`

---

## Prasyarat (Prerequisite)

| Item | Cara Verifikasi |
|------|-----------------|
| Skenario 10 & 11 selesai | Endpoint `visitor-categories` dan `visitors` (dengan JOIN kategori) aktif; RDF `payload\visitors.json` memuat `category_code` & `category_name` serta `datatablesQuery` JOIN |
| RDF `visitor-categories.json` tersedia sebagai sibling | File `payload\visitor-categories.json` ada di folder yang sama dengan `payload\visitors.json`. Auto-discovery memuat RDF ini dari sibling folder saat memproses JOIN; bila tidak ada, page kategori tidak ter-generate dan hanya referensi `select` yang dipertahankan |
| `fieldNameLookup` visitor_categories aktif | RDF `payload\visitor-categories.json` memuat `fieldNameLookup` (langkah opsional Skenario 10) — wajib agar endpoint `/lookup` mengembalikan `id` + `text` untuk dropdown |
| Skenario 8 selesai | License RESTForge Designer aktif (`restforge-designer license status` → `valid`); `restforge-designer --version` dapat dipanggil. Struktur UDF single-page dari Skenario 8 sudah ada di `sandbox\frontend\payload` (akan ditimpa) |
| Working directory awal | `sandbox\backend`, path berakhiran `\playbook\sandbox\backend` |

---

## Sumber Rujukan (Reference)

| Topik | File Handbook |
|-------|---------------|
| `payload migrate` (RDF → UDF) | [commands/restforge-backend/payload/migrate.md](https://github.com/restforge/handbook/blob/main/commands/restforge-backend/payload/migrate.md) |
| Struktur multi-page (`extends`, `pages` include) | [catalogs/udf/payload-envelope.md](https://github.com/restforge/handbook/blob/main/catalogs/udf/payload-envelope.md) |
| Dropdown `select` + `dataSource` API | [catalogs/udf/data-source.md](https://github.com/restforge/handbook/blob/main/catalogs/udf/data-source.md) |
| Sidebar `navigation` | [catalogs/udf/navigation.md](https://github.com/restforge/handbook/blob/main/catalogs/udf/navigation.md) |
| `restforge-designer generate` | [commands/restforge-frontend/generate.md](https://github.com/restforge/handbook/blob/main/commands/restforge-frontend/generate.md) |

---

## Langkah Eksekusi (Execution Steps)

> Karena RDF `visitors` sudah JOIN ke `visitor_categories` (Skenario 11), satu perintah migrate menghasilkan aplikasi dua-page lengkap tanpa penyusunan multi-page manual.

### Langkah 1: Migrate visitors.json (Auto Multi-Page)

Dari `sandbox\backend`, jalankan migrate **hanya** untuk `visitors.json`. Flag `--overwrite` diperlukan karena struktur UDF dari Skenario 8 sudah ada di output folder:

```bat
npx restforge payload migrate --project=visitors-app --name=visitors.json --output=../frontend/payload --config=db-connection.env --overwrite
```

Migrator membaca `datatablesQuery` pada RDF `visitors`, mendeteksi JOIN ke `visitor_categories`, lalu memuat RDF `visitor-categories.json` dari folder payload yang sama dan mengkonversinya menjadi page tersendiri. Output menampilkan `Pages: 2`:

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
  Pages        : 2

  [OK] visitor-categories: 6 field(s), 0 table column(s)
  [OK] visitors: 4 field(s), 4 table column(s)

  Files written:
    - app-config.json
    - pages\visitor-categories.json
    - pages\visitors.json
    - visitors-app.json

  Migration completed successfully.
```

> **Penting:** cukup migrate `visitors.json` saja. Jangan migrate `visitor-categories.json` secara terpisah ke project yang sama. Setiap migrate menulis ulang aggregator `visitors-app.json` untuk RDF utamanya masing-masing, sehingga migrate terpisah akan saling menimpa dan menyisakan aggregator single-page. Page `visitor-categories` sudah otomatis terbentuk via auto-discovery JOIN, dengan urutan benar (master lebih dulu, `visitors` sebagai homepage di urutan terakhir).

---

### Langkah 2: Tinjau Hasil Auto-Generate

Seluruh struktur terbentuk otomatis dari migrate. Pindah ke folder payload frontend untuk memeriksanya:

```bat
dir ..\frontend\payload
dir ..\frontend\payload\pages
```

Struktur yang terbentuk:

| File | Isi |
|------|-----|
| `app-config.json` | Blok `appConfig` bersama (appName, appCode, plugin, apiBaseUrl, port) |
| `pages\visitor-categories.json` | Page kategori (6 field: `category_code`, `category_name`, `description`, `default_duration_hours`, `requires_escort`, `is_active`) dengan `enableStatusFilter` otomatis dari `is_active` |
| `pages\visitors.json` | Page visitors dengan `category_id` sebagai dropdown `select` (lihat di bawah) |
| `visitors-app.json` | Aggregator: `extends` ke `app-config.json`, `homepage: visitors`, `include` kedua page, dan `navigation` dua item |

**Dropdown kategori otomatis.** Field FK `category_id` yang ber-JOIN otomatis dikonversi menjadi dropdown `select` dengan `dataSource` API. Isi `pages\visitors.json` pada bagian `category_id`:

```json
{
    "name": "category_id",
    "label": "Category",
    "type": "select",
    "inTable": true,
    "tableOrder": 6,
    "tableField": "category_name",
    "dataSource": {
        "type": "api",
        "resource": "visitor-categories",
        "select": ["category_id", "category_name"]
    }
}
```

`resource: "visitor-categories"` menghasilkan request `POST /api/visitors-app/visitor-categories/lookup`. `tableField: "category_name"` membuat DataTable menampilkan nama kategori (bukan UUID `category_id`). `select` diambil dari kolom JOIN aktual (`category_id`, `category_name`), bukan tebakan.

**Aggregator otomatis.** Isi `visitors-app.json`:

```json
{
    "extends": "app-config.json",
    "homepage": "visitors",
    "pages": [
        { "include": "pages/visitor-categories.json" },
        { "include": "pages/visitors.json" }
    ],
    "navigation": {
        "items": [
            { "type": "page", "pageRef": "visitor-categories", "label": "Visitor Categories" },
            { "type": "page", "pageRef": "visitors",            "label": "Visitors" }
        ]
    }
}
```

> **Catatan (perilaku `inTable`):** default `inTable` adalah `false`. Selama tidak ada satu pun field ber-`inTable`, DataTable menampilkan seluruh field (mode fallback) — kondisi ini berlaku pada page `visitor-categories` (output `0 table column(s)`). Pada page `visitors`, migrator menetapkan `inTable: true` + `tableOrder` pada `name`, `email`, `phone`, dan `category_id` berdasarkan urutan kolom di `datatablesQuery` (output `4 table column(s)`), sehingga keempat kolom muncul terprediksi tanpa warning. Gap pada `tableOrder` (mis. 1, 2, 3, 6) hanya menentukan urutan relatif dan tidak menimbulkan masalah.

**Kustomisasi opsional.** Bila diperlukan penyesuaian (mis. label, icon, urutan kolom, `pageSubtitle`, atau menambah `navigation.icon`), edit langsung fragmen di `pages\` atau aggregator `visitors-app.json` setelah migrate, lalu lanjut ke validasi dan generate. Penambahan `icon` pada navigation, contohnya:

```json
{ "type": "page", "pageRef": "visitor-categories", "icon": "database", "label": "Visitor Categories" },
{ "type": "page", "pageRef": "visitors",            "icon": "users",    "label": "Visitors" }
```

---

### Langkah 3: Validasi UDF

Pindah ke `sandbox\frontend` (naik satu level dari folder `payload`), lalu validasi aggregator sebelum generate:

```bat
restforge-designer validate --payload=payload/visitors-app.json
```

Validasi harus lolos tanpa error. Error `pageRef ... does not match any pageId` menandakan `include` gagal atau `pageId` tidak sesuai. Error `does not contain a valid 'pages' array` menandakan fragmen di `pages\` kehilangan pembungkus array `pages`.

---

### Langkah 4: Generate Aplikasi Frontend

Dari `sandbox\frontend`. Skenario 8 menghasilkan aplikasi single-page (hanya `visitors`), sehingga set halaman berubah dari satu menjadi dua. Karena `--overwrite` menimpa file page (HTML/JS) tetapi **tidak** meregenerasi `index.html` yang sudah ada, hapus dulu `index.html` lama agar landing page diregenerasi sesuai set page baru:

```bat
del apps\visitors-app\index.html
restforge-designer generate --payload=payload/visitors-app.json --output=./apps/visitors-app --overwrite
```

Tanpa menghapus `index.html`, landing page lama (hanya card `Visitors`) akan tetap muncul. Setelah dihapus, generator membuat ulang `index.html` sebagai landing multi-page berisi card `Visitor Categories` dan `Visitors`. Flag `--payload` selalu menunjuk ke aggregator `visitors-app.json`, bukan ke fragmen `pages\<page>.json`.

Verifikasi hasil:

```bat
dir apps\visitors-app
```

Output memuat shared files (`index.html`, `app-start.bat`, folder `js\`, `css\`) dan dua file per-page: `visitor-categories.html` + `js\visitor-categories.js` dan `visitors.html` + `js\visitors.js`.

---

### Langkah 5: Run dan Verifikasi Berurutan

Pastikan backend server running pada cmd pertama. Skenario 12 hanya mengubah frontend; endpoint backend (`visitors` dan `visitor-categories`) sudah final sejak Skenario 10-11, sehingga server cukup dipastikan aktif di port `3000`, bukan di-refresh:

```bat
netstat -ano | findstr :3000
```

Output harus menampilkan entri `LISTENING`. Bila belum, jalankan backend mengikuti Skenario 6 Langkah 4:

```bat
cd playbook\sandbox\backend
npx restforge serve --project=visitors-app --config=db-connection.env
```

Server yang aktif menampilkan log `Loading 2 endpoint(s)` (`visitors` + `visitor_categories`) dan `[OK] Server ready on port 3000`.

Start frontend pada cmd kedua:

```bat
cd playbook\sandbox\frontend\apps\visitors-app
app-start.bat
```

Buka browser ke `http://localhost:8000/index.html`. Sidebar menampilkan **Visitor Categories** dan **Visitors**.

Verifikasi berurutan (urutan tidak boleh dibalik):

| Urutan | Langkah | Verifikasi |
|--------|---------|------------|
| 1 | Buka **Visitor Categories** → Create kategori (mis. `category_code=VIP`, `category_name=VIP Guest`) → submit | Record muncul di tabel; Network: `POST /api/visitors-app/visitor-categories/create` → `success: true` |
| 2 | Buka **Visitors** → Create/Edit visitor → buka dropdown **Category** | Dropdown memuat kategori dari langkah 1; Network: `POST /api/visitors-app/visitor-categories/lookup` → `200` |
| 3 | Pilih kategori → submit | Network: `POST /api/visitors-app/visitors/create` (atau `/update`) → `success: true` |
| 4 | Amati tabel **Visitors** | Kolom **Category** menampilkan `category_name` terisi (bukan UUID) |

Catatan urutan: dropdown kategori kosong apabila belum ada record `visitor_categories`. Tambah kategori (langkah 1) wajib dilakukan sebelum create/update visitor.

---

### Langkah 6: Data Contoh Visitor Categories

Sebagai bahan pengisian saat Create kategori pada Langkah 5, berikut contoh data `visitor_categories` yang dapat dimasukkan melalui form **Visitor Categories**. Kolom `Status` dibiarkan default (Active):

| Category Code | Category Name | Description | Default Duration Hours | Requires Escort |
|---------------|---------------|-------------|------------------------|-----------------|
| GUEST | Tamu Umum | Tamu biasa seperti klien atau mitra bisnis | 8 | Yes |
| VIP | VIP | Tamu penting seperti direksi atau pejabat | 8 | No |
| VENDOR | Vendor/Supplier | Pihak ketiga penyedia barang atau jasa | 8 | Yes |
| CONTRACTOR | Kontraktor | Pekerja eksternal jangka pendek/proyek | 12 | No |
| INTERVIEWEE | Kandidat | Pelamar kerja yang datang wawancara | 4 | Yes |
| INTERN | Magang | Mahasiswa atau anak magang | 10 | No |
| COURIER | Kurir | Pengantar paket atau dokumen | 1 | Yes |
| MAINTENANCE | Maintenance | Teknisi servis seperti AC, lift, kebersihan | 8 | Yes |
| AUDITOR | Auditor | Auditor eksternal atau inspektor | 8 | No |
| FAMILY | Keluarga Karyawan | Keluarga atau kerabat karyawan | 4 | Yes |

---

## Langkah Berikutnya (Next Step)

Lanjut ke Skenario 13: update UDF dan generate ulang per-page dan penuh.
