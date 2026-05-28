# Skenario 12: Aplikasi Frontend Multi-Page visitor_categories dan visitors

> Tahap kedua belas onboarding RESTForge. Menggabungkan dua resource (`visitor_categories` dan `visitors`) ke dalam satu aplikasi frontend multi-page. Halaman `visitors` dilengkapi dropdown kategori yang memuat data dari endpoint `visitor-categories/lookup`, sehingga proses berurutan: tambah kategori terlebih dahulu, lalu create/update visitor dengan memilih kategori tersebut.

---

## Tujuan (Objective)

1. RDF `visitor-categories.json` dan `visitors.json` di-migrate menjadi dua UDF di `sandbox\frontend\payload`
2. UDF digabung menjadi satu aplikasi multi-page (`app-config.json` + `pages\*.json` + `visitors-app.json`) dengan sidebar navigasi
3. Field `category_id` pada halaman `visitors` dikonfigurasi sebagai dropdown `select` yang memuat opsi dari endpoint `visitor-categories/lookup`
4. Aplikasi frontend ter-generate ke `sandbox\frontend\apps\visitors-app` dengan halaman `visitor-categories.html` dan `visitors.html`
5. Verifikasi end-to-end berurutan di browser: tambah kategori → create/update visitor (pilih kategori) → list `visitors` menampilkan `category_name`

---

## Prasyarat (Prerequisite)

| Item | Cara Verifikasi |
|------|-----------------|
| Skenario 10 & 11 selesai | Endpoint `visitor-categories` dan `visitors` (dengan JOIN kategori) aktif; RDF `payload\visitors.json` memuat `category_code` & `category_name` |
| `fieldNameLookup` visitor_categories aktif | RDF `payload\visitor-categories.json` memuat `fieldNameLookup` (langkah opsional Skenario 10) — wajib agar endpoint `/lookup` mengembalikan `id` + `text` untuk dropdown |
| Skenario 8 selesai | License RESTForge Designer aktif (`restforge-designer license status` → `valid`); `restforge-designer --version` dapat dipanggil |
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

> Urutan langkah bersifat ketat: `visitor_categories` adalah master, `visitors` bergantung padanya. Migrate dan susun kategori lebih dulu, baru `visitors`.

### Langkah 1: Migrate Kedua RDF ke UDF

Dari `sandbox\backend`, migrate kategori lebih dulu, lalu visitors. Flag `--overwrite` diperlukan karena `visitors.json` sudah ada dari Skenario 8:

```bat
npx restforge payload migrate --name=visitor-categories.json --project=myapp --output=..\frontend\payload --config=db-connection.env --overwrite
npx restforge payload migrate --name=visitors.json --project=myapp --output=..\frontend\payload --config=db-connection.env --overwrite
```

Setiap perintah menghasilkan UDF single-page (`Pages: 1`). Output `visitors` kini menyertakan `category_id`; kolom display JOIN (`category_code`, `category_name`) di-exclude dari form sesuai aturan migrasi.

---

### Langkah 2: Susun Struktur Multi-Page

Pindah ke folder payload frontend dan buat folder `pages`:

```bat
cd ..\frontend\payload
mkdir pages
```

Hasil migrate (`visitor-categories.json`, `visitors.json`) adalah UDF lengkap berisi `appConfig` + satu `pages`. Pisahkan menjadi struktur multi-page: `appConfig` bersama ke `app-config.json`, dan tiap object page ke `pages\*.json`.

**a. Buat `app-config.json`** (salin blok `appConfig` dari salah satu hasil migrate):

```json
{
    "appConfig": {
        "appName": "My Application",
        "appCode": "myapp",
        "plugin": "vanilla-js-basic",
        "apiBaseUrl": "http://127.0.0.1:3000/api/myapp",
        "port": 8000
    }
}
```

**b. Buat `pages\visitor-categories.json`** — salin object di dalam `pages[0]` dari hasil migrate `visitor-categories.json` (hanya object page, tanpa pembungkus `appConfig`/`pages`):

```json
{
    "pageId": "visitor-categories",
    "pageTitle": "Visitor Categories",
    "apiPath": "visitor-categories",
    "primaryKey": "category_id",
    "displayField": "category_name",
    "features": { "enableSearch": true, "fieldLayout": "vertical" },
    "fields": [ "... hasil migrate: category_code, category_name, description, default_duration_hours, requires_escort, is_active ..." ]
}
```

**c. Buat `pages\visitors.json`** — salin object `pages[0]` dari hasil migrate `visitors.json`, lalu konfigurasi `category_id` sebagai dropdown (lihat Langkah 3).

---

### Langkah 3: Konfigurasi Dropdown Kategori

Pada `pages\visitors.json`, pastikan field `category_id` bertipe `select` dengan `dataSource` API ke resource `visitor-categories`. Konfigurasi ini wajib agar update visitor dapat memilih kategori dari dropdown (database tidak memiliki FK constraint fisik, sehingga migrate tidak selalu menghasilkan `select` otomatis):

```json
{
    "pageId": "visitors",
    "pageTitle": "Visitors",
    "apiPath": "visitors",
    "primaryKey": "visitor_id",
    "displayField": "name",
    "features": { "enableSearch": true, "fieldLayout": "vertical" },
    "fields": [
        { "name": "name",  "label": "Name",  "type": "text", "required": true, "maxlength": 100 },
        { "name": "email", "label": "Email", "type": "text", "required": true, "maxlength": 100 },
        { "name": "phone", "label": "Phone", "type": "text", "required": true, "maxlength": 20 },
        {
            "name": "category_id",
            "label": "Category",
            "type": "select",
            "required": false,
            "inTable": true,
            "tableField": "category_name",
            "dataSource": {
                "type": "api",
                "resource": "visitor-categories",
                "select": ["category_id", "category_name"]
            }
        }
    ]
}
```

`tableField: "category_name"` membuat DataTable menampilkan nama kategori (bukan UUID `category_id`). `resource: "visitor-categories"` menghasilkan request `POST /api/myapp/visitor-categories/lookup`.

---

### Langkah 4: Buat UDF Utama Multi-Page

Buat `visitors-app.json` di `sandbox\frontend\payload` sebagai UDF gabungan. File ini menarik `appConfig` via `extends`, menyusun kedua page via `include`, dan mendefinisikan sidebar:

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
            { "type": "page", "pageRef": "visitor-categories", "icon": "database", "label": "Visitor Categories" },
            { "type": "page", "pageRef": "visitors",            "icon": "users",    "label": "Visitors" }
        ]
    }
}
```

Validasi sebelum generate:

```bat
cd ..
restforge-designer validate --payload=payload/visitors-app.json
```

Validasi harus lolos tanpa error. Error `pageRef ... does not match any pageId` menandakan `include` gagal atau `pageId` tidak sesuai.

---

### Langkah 5: Generate Aplikasi Frontend

Dari `sandbox\frontend`:

```bat
restforge-designer generate --payload=payload/visitors-app.json --output=./apps/visitors-app --overwrite
```

Flag `--overwrite` menimpa folder `visitors-app` dari Skenario 8 (sebelumnya single-page `visitors`) menjadi aplikasi multi-page.

Verifikasi hasil:

```bat
dir apps\visitors-app
```

Output memuat shared files (`index.html`, `app-start.bat`, folder `js\`, `css\`) dan dua file per-page: `visitor-categories.html` + `js\visitor-categories.js` dan `visitors.html` + `js\visitors.js`.

---

### Langkah 6: Run dan Verifikasi Berurutan

Start backend pada cmd pertama:

```bat
cd playbook\sandbox\backend
npx restforge serve --project=myapp --config=db-connection.env
```

Log menampilkan `Loading 2 endpoint(s)` dan `[OK] Server ready on port 3000`.

Start frontend pada cmd kedua:

```bat
cd playbook\sandbox\frontend\apps\visitors-app
app-start.bat
```

Buka browser ke `http://localhost:8000/index.html`. Sidebar menampilkan **Visitor Categories** dan **Visitors**.

Verifikasi berurutan (urutan tidak boleh dibalik):

| Urutan | Langkah | Verifikasi |
|--------|---------|------------|
| 1 | Buka **Visitor Categories** → Create kategori (mis. `category_code=VIP`, `category_name=VIP Guest`) → submit | Record muncul di tabel; Network: `POST /api/myapp/visitor-categories/create` → `success: true` |
| 2 | Buka **Visitors** → Create/Edit visitor → buka dropdown **Category** | Dropdown memuat kategori dari langkah 1; Network: `POST /api/myapp/visitor-categories/lookup` → `200` |
| 3 | Pilih kategori → submit | Network: `POST /api/myapp/visitors/create` (atau `/update`) → `success: true` |
| 4 | Amati tabel **Visitors** | Kolom **Category** menampilkan `category_name` terisi (bukan UUID) |

Catatan urutan: dropdown kategori kosong apabila belum ada record `visitor_categories`. Tambah kategori (langkah 1) wajib dilakukan sebelum create/update visitor.

---

### Langkah 7: Stop

Stop frontend (cmd kedua) dan backend (cmd pertama) via `Ctrl + C`.

---

## Kriteria Selesai (Completion Criteria)

| Item | Kondisi |
|------|---------|
| UDF source | `sandbox\frontend\payload` berisi `app-config.json`, `visitors-app.json`, `pages\visitor-categories.json`, `pages\visitors.json` |
| Validasi | `restforge-designer validate --payload=payload/visitors-app.json` lolos tanpa error |
| Field dropdown | `category_id` pada `pages\visitors.json` bertipe `select` dengan `dataSource` API `resource: visitor-categories` |
| Aplikasi | `sandbox\frontend\apps\visitors-app` berisi `index.html`, `visitor-categories.html`, `visitors.html`, `app-start.bat` |
| Sidebar | Menampilkan link **Visitor Categories** dan **Visitors** |
| Dropdown runtime | Halaman Visitors memuat opsi kategori via `POST /api/myapp/visitor-categories/lookup` (`200`) |
| End-to-end | Visitor dengan kategori tersimpan; tabel Visitors menampilkan `category_name` |

---

## Catatan untuk Tahap Berikutnya (Notes for Next Stage)

Aplikasi multi-page kini menggabungkan master (`visitor_categories`) dan transaksi (`visitors`) dalam satu frontend. Pengembangan lanjutan dapat mengeksplorasi `dataFilters` (filter kategori di toolbar Visitors), `fieldRows` (grid layout form), atau penambahan resource lain ke array `pages` dengan pola `include` yang sama.

---

## Pelaporan Issue (Issue Reporting)

Apabila ditemukan kondisi tidak sesuai ekspektasi, hentikan eksekusi dan dokumentasikan output lengkap perintah yang gagal beserta nomor langkah. Untuk error validasi/generate, sertakan output `restforge-designer validate` dan isi `visitors-app.json`. Untuk dropdown kosong di browser, sertakan screenshot tab Network (request `/visitor-categories/lookup`) dan tab Console DevTools.
