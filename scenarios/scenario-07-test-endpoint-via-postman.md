# Skenario 7: Test Endpoint via Postman

> Tahap ketujuh onboarding RESTForge. Tujuan akhir skenario ini adalah seluruh action endpoint `visitors` ter-test via Postman menggunakan collection yang ter-generate otomatis oleh `endpoint create`.

---

## Tujuan (Objective)

Setelah menyelesaikan skenario ini, kondisi berikut harus tercapai:

1. Collection Postman `Myapp - Visitors` ter-import ke workspace Postman
2. Variable `{{baseUrl}}` ter-set ke `http://localhost:3000`
3. Tujuh action endpoint (`create`, `datatables`, `read`, `first`, `lookup`, `update`, `delete`) berhasil dieksekusi dan mengembalikan response sesuai ekspektasi

---

## Sumber Rujukan (Reference)

| Topik | Sumber |
|-------|--------|
| Postman collection auto-generated | `examples\myapp\visitors\postman\visitors.json` |
| Behavior endpoint `first` dan `read` (resolusi sumber data) | [docs/features/feat-first.md](../../docs/features/feat-first.md), [docs/features/feat-read.md](../../docs/features/feat-read.md) |
| Postman documentation (import, environment, send request) | https://learning.postman.com/docs/ |

---

## Prasyarat (Prerequisite)

| Item | Cara Verifikasi |
|------|-----------------|
| Skenario 6 sudah selesai | Folder `examples\myapp\visitors\postman\` berisi `visitors.json` |
| Postman terinstall di workstation | Aplikasi Postman dapat dibuka (versi free sudah cukup) |
| Server `myapp` running | Eksekusi `serve --project=myapp` pada Skenario 6 Langkah 4 masih aktif dan port `3000` berstatus `LISTENING` |

---

## Langkah Eksekusi (Execution Steps)

### Langkah 1: Import Collection dan Setup Variable

Buka aplikasi Postman, lalu import collection auto-generated:

1. Klik tombol **Import** pada sidebar workspace
2. Drag file `playbook\sandbox\backend\examples\myapp\visitors\postman\visitors.json` ke area drop, atau klik **Choose Files** dan pilih file tersebut
3. Konfirmasi import. Collection `Myapp - Visitors` akan muncul di sidebar dengan 7 request

Setup variable `{{baseUrl}}` pada level collection:

1. Klik kanan collection `Myapp - Visitors`, pilih **Edit**
2. Buka tab **Variables**
3. Tambah variable dengan nilai berikut:

| Variable | Initial Value | Current Value |
|----------|---------------|---------------|
| `baseUrl` | `http://localhost:3000` | `http://localhost:3000` |

4. Klik **Save**

![Setup variable baseUrl](images/scenario-07/postman-langkah-1-variable.png)

*Gambar: tab Variables pada collection Edit, menampilkan baris `baseUrl` dengan nilai `http://localhost:3000`*

---

### Langkah 2: Test Action `create`

Pilih request `visitors/create` pada collection. Detail konfigurasi:

| Properti | Nilai |
|----------|-------|
| Method | `POST` |
| URL | `{{baseUrl}}/api/myapp/visitors/create` |
| Headers | `Content-Type: application/json` (auto-set) |

Body (raw JSON):

```json
{
  "name": "Test Postman",
  "email": "test-postman@example.com",
  "phone": "+62812345678"
}
```

Catatan: payload default pada collection berisi `"visitor_id": "Demo visitor_id"`. Hapus field `visitor_id` tersebut karena runtime akan auto-generate UUID v7. Sisanya tetap.

Klik **Send**. Expected response:

```json
{
  "success": true,
  "message": "visitors data successfully added",
  "data": {
    "visitor_id": "019e5c36-53a2-72fc-b287-df210e8e5aae",
    "name": "Test Postman",
    "email": "test-postman@example.com",
    "phone": "+62812345678"
  },
  "timestamp": "..."
}
```

**PENTING**: catat nilai `visitor_id` dari response untuk digunakan pada Langkah 5, 7, dan 8.

![Test action create](images/scenario-07/postman-langkah-2-create.png)

*Gambar: tab request `visitors/create` menampilkan method POST, URL, Body JSON, dan response sukses 201 dengan visitor_id UUID v7*

---

### Langkah 3: Test Action `datatables`

Pilih request `visitors/datatables`. Detail konfigurasi:

| Properti | Nilai |
|----------|-------|
| Method | `POST` |
| URL | `{{baseUrl}}/api/myapp/visitors/datatables` |
| Headers | `Content-Type: application/json` (auto-set) |

Body (sudah ter-set oleh collection, biarkan apa adanya):

```json
{
  "draw": 1,
  "start": 0,
  "length": 10,
  "search": {
    "value": ""
  }
}
```

Klik **Send**. Expected response menampilkan `recordsTotal` dan array `data` yang memuat record hasil Langkah 2.

![Test action datatables](images/scenario-07/postman-langkah-3-datatables.png)

*Gambar: response datatables dengan recordsTotal dan data array berisi record hasil create*

---

### Langkah 4: Test Action `read`

Pilih request `visitors/read`. Detail konfigurasi:

| Properti | Nilai |
|----------|-------|
| Method | `POST` |
| URL | `{{baseUrl}}/api/myapp/visitors/read` |
| Headers | `Content-Type: application/json` (auto-set) |

Body:

```json
{
  "page": 1,
  "per_page": 10,
  "search_value": "",
  "search_by": "name"
}
```

Klik **Send**. Expected response memuat array `data`, field `count`, dan object `pagination`.

![Test action read](images/scenario-07/postman-langkah-4-read.png)

*Gambar: response read dengan data array dan pagination metadata*

---

### Langkah 5: Test Action `first`

Pilih request `visitors/first`. Detail konfigurasi:

| Properti | Nilai |
|----------|-------|
| Method | `POST` |
| URL | `{{baseUrl}}/api/myapp/visitors/first` |
| Headers | `Content-Type: application/json` (auto-set) |

Body (**edit**: ganti placeholder `"Demo visitor_id"` dengan UUID hasil Langkah 2):

```json
{
  "where": {
    "key": "visitor_id",
    "value": "<paste UUID dari Langkah 2 di sini>"
  }
}
```

Klik **Send**. Expected response memuat 1 record dengan `visitor_id` yang sesuai.

![Test action first](images/scenario-07/postman-langkah-5-first.png)

*Gambar: request first dengan visitor_id actual dan response data single record*

---

### Langkah 6: Test Action `lookup`

Pilih request `visitors/lookup`. Detail konfigurasi:

| Properti | Nilai |
|----------|-------|
| Method | `POST` |
| URL | `{{baseUrl}}/api/myapp/visitors/lookup` |
| Headers | `Content-Type: application/json` + **`X-Request-Mode: static`** |

**PENTING**: header `X-Request-Mode: static` **wajib** untuk action `lookup` dengan method POST. Tanpa header ini, runtime mengembalikan HTTP 400 dengan pesan `"X-Request-Mode header must be set to static for POST method"`. Collection auto-generated sudah memasukkan header ini, namun perlu diverifikasi pada tab **Headers**.

Body (sudah ter-set oleh collection):

```json
{
  "select": ["visitor_id", "name"]
}
```

Klik **Send**. Expected response berformat dropdown-friendly `{id, text}`:

```json
{
  "success": true,
  "count": N,
  "data": [
    {"id": "019e...", "text": "Test Postman"}
  ]
}
```

![Test action lookup](images/scenario-07/postman-langkah-6-lookup.png)

*Gambar: tab Headers menampilkan X-Request-Mode: static aktif, plus response lookup format {id, text}*

---

### Langkah 7: Test Action `update`

Pilih request `visitors/update`. Detail konfigurasi:

| Properti | Nilai |
|----------|-------|
| Method | `POST` |
| URL | `{{baseUrl}}/api/myapp/visitors/update` |
| Headers | `Content-Type: application/json` (auto-set) |

Body (**edit**: ganti `"Demo visitor_id"` dengan UUID hasil Langkah 2):

```json
{
  "visitor_id": "<paste UUID dari Langkah 2 di sini>",
  "name": "Test Postman UPDATED",
  "email": "test-postman-updated@example.com",
  "phone": "+62899887766"
}
```

Klik **Send**. Expected response memuat data yang ter-update dengan `success: true` dan `message: "visitors data successfully updated"`.

![Test action update](images/scenario-07/postman-langkah-7-update.png)

*Gambar: request update dengan data baru dan response sukses 200*

---

### Langkah 8: Test Action `delete`

Pilih request `visitors/delete`. Detail konfigurasi:

| Properti | Nilai |
|----------|-------|
| Method | `POST` |
| URL | `{{baseUrl}}/api/myapp/visitors/delete` |
| Headers | `Content-Type: application/json` (auto-set) |

Body (**edit**: ganti `"Demo visitor_id"` dengan UUID hasil Langkah 2):

```json
{
  "where": [
    {
      "key": "visitor_id",
      "value": "<paste UUID dari Langkah 2 di sini>"
    }
  ]
}
```

Klik **Send**. Expected response:

```json
{
  "success": true,
  "message": "Successfully deleted 1 record(s)",
  "deleted_count": 1,
  "deleted_items": [
    {"visitor_id": "019e..."}
  ]
}
```

Verifikasi via re-eksekusi request `datatables` (Langkah 3). Nilai `recordsTotal` harus berkurang 1 dibanding sebelumnya.

![Test action delete](images/scenario-07/postman-langkah-8-delete.png)

*Gambar: response delete dengan deleted_count: 1 dan deleted_items berisi visitor_id yang dihapus*

---

## Kriteria Selesai (Completion Criteria)

Skenario 7 dianggap selesai apabila seluruh kondisi berikut terpenuhi:

| Action | Status Diharapkan |
|--------|-------------------|
| `create` | HTTP 201, response `success: true` dengan `visitor_id` UUID v7 |
| `datatables` | HTTP 200, `recordsTotal >= 1` |
| `read` | HTTP 200, response berisi `data` array dan `pagination` |
| `first` | HTTP 200, response berisi 1 record dengan `visitor_id` sesuai |
| `lookup` | HTTP 200, response berformat `{id, text}` (header `X-Request-Mode: static` ter-set) |
| `update` | HTTP 200, response memuat data yang ter-update |
| `delete` | HTTP 200, `deleted_count: 1` |

---

## Catatan untuk Tahap Berikutnya (Notes for Next Stage)

Skenario 7 menutup track backend onboarding RESTForge. Pada titik ini, kondisi yang tercapai:

- Project `myapp` ter-generate dari SDF dan RDF di workspace `sandbox\backend`
- Endpoint REST `visitors` berjalan pada runtime server
- Seluruh action endpoint tervalidasi via curl (Skenario 6) dan Postman (Skenario 7)

Skenario 8 akan melanjutkan ke track frontend: migrasi payload RDF backend (`sandbox\backend\payload\visitors.json`) menjadi UDF frontend (`sandbox\frontend\payload\visitors.json`), aktivasi license RESTForge Designer, dan generate aplikasi frontend ke `sandbox\frontend\apps\visitors-app\`.

Untuk pengembangan lanjutan di luar track onboarding, referensi tersedia pada handbook RESTForge sesuai topik (master-detail, processor, kafka, dashboard, dll.).

---

## Pelaporan Issue (Issue Reporting)

Apabila ditemukan kondisi tidak sesuai ekspektasi, hentikan eksekusi dan dokumentasikan permasalahan beserta detail request (method, URL, headers, body) dan response (status code, body) pada langkah yang gagal. Sertakan screenshot tab Postman bila terkait konfigurasi request.
