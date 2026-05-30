# Skenario 6: Generate Endpoint dan Jalankan Runtime Server

> Tahap keenam onboarding RESTForge. Tujuan akhir skenario ini adalah endpoint REST `visitors` ter-generate dari RDF, project siap dijalankan, dan runtime HTTP server berjalan pada port yang dikonfigurasi.

---

## Tujuan (Objective)

Setelah menyelesaikan skenario ini, kondisi berikut harus tercapai:

1. Endpoint REST `visitors` sudah ter-generate hasil dari `endpoint create` berbasis `payload\visitors.json`
2. Folder `src\models\visitors-app\`, file `metadata\visitors-app.json`, dan folder `examples\visitors-app\visitors\` sudah terbentuk di `sandbox\backend`
3. Runtime HTTP server berjalan pada port `3000` (default) dan menampilkan log siap menerima request
4. Endpoint `visitors` dapat di-test via demo curl pada folder `examples\visitors-app\visitors\curl\` dan mengembalikan response sesuai action (create, datatables, read, first, lookup, update, delete)

Catatan: skenario ini mencakup testing via demo `.bat` script (curl). Testing via UI/GUI tool (Postman, Insomnia, atau browser) akan dibahas pada Skenario 7.

---

## Sumber Rujukan (Reference)

Seluruh perintah pada skenario ini bersumber dari handbook berikut:

| Topik | File Handbook |
|-------|---------------|
| Resource `endpoint` dan daftar sub-command | [commands/endpoint/README.md](https://github.com/restforge/handbook/blob/main/commands/endpoint/README.md) |
| Generate endpoint REST dari RDF | [commands/endpoint/create.md](https://github.com/restforge/handbook/blob/main/commands/endpoint/create.md) |
| Resource `runtime` dan daftar sub-command | [commands/runtime/README.md](https://github.com/restforge/handbook/blob/main/commands/runtime/README.md) |
| Menjalankan runtime HTTP server | [commands/runtime/serve.md](https://github.com/restforge/handbook/blob/main/commands/runtime/serve.md) |
| Aturan validasi schema payload vs database | [catalogs/rdf/validation-rules.md](https://github.com/restforge/handbook/blob/main/catalogs/rdf/validation-rules.md) |

---

## Prasyarat (Prerequisite)

| Item | Cara Verifikasi |
|------|-----------------|
| Skenario 5 sudah selesai | File `payload\visitors.json` sudah ter-generate dan lolos `payload validate` |
| Working directory aktif di `sandbox\backend` | Jalankan `cd` pada cmd, path harus berakhiran `\playbook\sandbox\backend` |
| Tabel `visitors` ada di database | Sudah ter-create pada Skenario 4 |
| Default config sudah di-set | Perintah `npx restforge config get-default` menampilkan `db-connection.env` |
| Port `3000` tersedia | Tidak ada proses lain yang menggunakan port `3000` pada workstation |

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

Output harus menampilkan path `db-connection.env`. Command `endpoint create` akan menggunakan default config tersebut apabila flag `--config` tidak disediakan eksplisit, sehingga perintah pada Langkah 2 dapat dijalankan tanpa `--config`.

---

### Langkah 2: Generate Endpoint REST dari RDF

Perintah `endpoint create` membaca file RDF dan men-generate kode endpoint REST siap dijalankan. Bila project belum ada, project akan dibuat sebagai bagian dari proses.

Tiga flag wajib (selain `--config` yang sudah di-handle default):

| Flag | Nilai pada skenario ini | Keterangan |
|------|------------------------|-----------|
| `--project` | `visitors-app` | Nama project target |
| `--name` | `visitors` | Nama endpoint yang akan dibuat |
| `--payload` | `visitors.json` | Nama file RDF di folder `payload/` |

Jalankan generate:

```bat
npx restforge endpoint create --project=visitors-app --name=visitors --payload=visitors.json
```

Output sukses akan menampilkan ringkasan file yang ter-generate beserta example files (curl, Postman, Insomnia) karena flag default `--create-examples=true`.

#### Validasi Schema Database

Sebelum codegen dimulai, `endpoint create` melakukan cross-check antara RDF dan struktur tabel database aktual. Validasi mencakup:

- Setiap kolom pada `fieldName` harus ada di tabel database
- Kolom audit (`created_at`, `created_by`, `updated_at`, `updated_by`) harus ada di database apabila `auditColumns` aktif
- Kolom database yang tidak ada di RDF dilaporkan sebagai diff

Karena RDF `visitors.json` baru saja di-generate dari schema tabel `visitors` (Skenario 5), validasi diharapkan lolos tanpa drift.

Exit code yang mungkin:

| Code | Arti |
|------|------|
| `0` | Sukses |
| `1` | Schema drift terdeteksi |
| `2` | Usage error (mis. flag wajib tidak disediakan) |
| `3` | Connection error ke database |

---

### Langkah 3: Inspeksi Struktur Folder Hasil Generate

Setelah generate sukses, project `visitors-app` tersebar pada beberapa folder di dalam `sandbox\backend` sesuai konvensi RESTForge. Verifikasi visual disarankan menggunakan IDE seperti VS Code atau Cursor untuk membuka folder `sandbox\backend` dan menelusuri tree struktur folder secara langsung.

Struktur folder yang harus terbentuk setelah `endpoint create`:

| Path | Isi |
|------|-----|
| `src\models\visitors-app\` | Kode endpoint dan query SQL hasil generate untuk project `visitors-app` |
| `metadata\visitors-app.json` | File konfigurasi project `visitors-app` |
| `metadata\visitors-app-workbench.json` | File workbench config project `visitors-app` |
| `metadata\global.json` | File konfigurasi global (di-share lintas project) |
| `examples\visitors-app\visitors\` | Example files untuk endpoint `visitors` (sub-folder `curl\`, `insomnia\`, `postman\`) |

Sub-folder `examples\visitors-app\visitors\curl\` berisi 7 demo `.bat` script siap pakai per action endpoint (create, datatables, delete, first, lookup, read, update) plus `run-all-demos.bat`. File-file ini akan dipakai pada Skenario 7 untuk testing actual request.

Catatan: pada tahap onboarding ini tidak perlu modifikasi pada folder hasil generate. Customisasi akan dibahas pada skenario pengembangan setelah onboarding selesai.

---

### Langkah 4: Jalankan Runtime HTTP Server

Perintah `serve` menjalankan runtime HTTP server untuk project yang sudah ter-generate. Default port `3000` dan default bind address `127.0.0.1`.

Jalankan server untuk project `visitors-app`:

```bat
npx restforge serve --project=visitors-app --config=db-connection.env
```

Server akan start dan menampilkan log informasi runtime. Proses ini bersifat **foreground**, artinya cmd akan tetap berjalan menampilkan log dan tidak kembali ke prompt sampai server di-stop.

#### Output yang Diharapkan

Output sukses terdiri dari dua bagian: **banner ringkasan konfigurasi** dan **log startup**.

```
╔═══════════════════════════════════════════════════════╗
║               RESTFORGE RUNTIME SERVER                ║
╠═══════════════════════════════════════════════════════╣
║  Environment : Node.js                                ║
║  Project     : visitors-app                           ║
║  Port        : 3000                                   ║
║  Config      : db-connection.env                      ║
║  API Key     : NOT ACTIVE                             ║
╚═══════════════════════════════════════════════════════╝

INFO [16:59:36]: Configuration loaded: db-connection.env
INFO [16:59:36]: 
INFO [16:59:36]: [OK] Project loaded: visitors-app
INFO [16:59:36]: Starting visitors-app project...
INFO [16:59:36]: Starting VisitorsApp module
INFO [16:59:36]: Loading 1 endpoint(s)
INFO [16:59:36]: [OK] Server ready on port 3000
INFO [16:59:36]:   Health: http://127.0.0.1:3000/api/visitors-app/health
INFO [16:59:36]:   Info:   http://127.0.0.1:3000/api/visitors-app/info
INFO [16:59:36]:   URL:    http://127.0.0.1:3000
```

Penjelasan per bagian output:

| Bagian | Keterangan |
|--------|-----------|
| Banner box | Ringkasan konfigurasi runtime aktif: nama project, port, file config, dan status API Key |
| `Configuration loaded` | Konfirmasi file config (`db-connection.env`) berhasil dibaca |
| `[OK] Project loaded: visitors-app` | Project `visitors-app` berhasil di-load dari folder `src\models\visitors-app\` |
| `Loading 1 endpoint(s)` | Jumlah endpoint yang ter-detect pada project. Pada skenario ini bernilai `1` (endpoint `visitors`) |
| `[OK] Server ready on port 3000` | Konfirmasi server sudah listening pada port yang dikonfigurasi |
| `Health`, `Info`, `URL` | URL utility built-in untuk monitoring server. Detail penggunaan dibahas pada Skenario 7 |

Catatan:

- Nilai timestamp (`[16:59:36]`) akan berbeda pada setiap eksekusi sesuai waktu sistem
- Nilai `Loading N endpoint(s)` akan bertambah apabila pada satu project terdapat lebih dari satu endpoint hasil `endpoint create`
- Status `API Key : NOT ACTIVE` adalah default karena fitur autentikasi belum diaktifkan pada onboarding ini

---

### Langkah 5: Verifikasi Server Aktif

Pada cmd yang menjalankan server, log harus tetap menampilkan status running tanpa error fatal. Server siap menerima HTTP request pada port yang dikonfigurasi.

Untuk verifikasi cepat dari cmd lain (buka jendela cmd baru), gunakan perintah berikut untuk memeriksa apakah port `3000` sudah terbuka:

```bat
netstat -ano | findstr :3000
```

Output menampilkan entri `LISTENING` pada `127.0.0.1:3000` apabila server aktif.

#### Cara Stop Server

Server perlu tetap berjalan untuk mendukung Langkah 6 (Test Endpoint REST via curl), dan dibiarkan tetap aktif untuk dilanjutkan ke Skenario 7 (test via Postman) hingga Skenario 9 (integrasi frontend). Skenario-skenario tersebut hanya memverifikasi server sudah running, bukan menjalankannya ulang, sehingga server cukup dijalankan sekali di sini.

Server dihentikan hanya bila ingin mengakhiri sesi onboarding. Untuk menghentikannya, gunakan kombinasi tombol berikut pada cmd yang menjalankan `serve`:

```
Ctrl + C
```

Server akan melakukan graceful shutdown dan cmd kembali ke prompt. Konfirmasi dengan menjalankan kembali `netstat -ano | findstr :3000` pada cmd lain. Entri `LISTENING` pada port `3000` harus hilang setelah server berhenti.

Catatan: jangan menutup jendela cmd secara paksa (mis. close window) saat server sedang berjalan tanpa Ctrl+C terlebih dahulu, karena proses background dapat tertinggal dan mengunci port.

---

### Langkah 6: Test Endpoint REST via curl

Server `visitors-app` (Langkah 4) sudah listening pada port `3000` dan endpoint `visitors` siap menerima request. Folder `examples\visitors-app\visitors\curl\` berisi 7 demo `.bat` script siap pakai per action endpoint plus `run-all-demos.bat` sebagai orkestrator.

#### Daftar Demo Script

| Script | Action | Endpoint Path |
|--------|--------|---------------|
| `demo-create.bat` | create | `POST /api/visitors-app/visitors/create` |
| `demo-datatables.bat` | datatables | `POST /api/visitors-app/visitors/datatables` |
| `demo-read.bat` | read | `POST /api/visitors-app/visitors/read` |
| `demo-first.bat` | first | `POST /api/visitors-app/visitors/first` |
| `demo-lookup.bat` | lookup | `POST /api/visitors-app/visitors/lookup` |
| `demo-update.bat` | update | `POST /api/visitors-app/visitors/update` |
| `demo-delete.bat` | delete | `POST /api/visitors-app/visitors/delete` |

Setiap script membaca file payload JSON dari sub-folder `payload\` dan mengeksekusi `curl -X POST` dengan payload tersebut ke endpoint yang sesuai.

#### Persiapan Eksekusi

Buka **jendela cmd baru** (jangan tutup cmd yang menjalankan server) dan pindah ke folder demo:

```bat
cd playbook\sandbox\backend\examples\visitors-app\visitors\curl
```

#### Eksekusi Demo Tunggal: create

Mulai dari demo `create` untuk membuat record baru:

```bat
demo-create.bat
```

Script akan menampilkan isi `payload\create.json`, mengeksekusi `curl` ke endpoint `/create`, lalu menampilkan response JSON dari server. Contoh response sukses:

```json
{
  "success": true,
  "message": "visitors data successfully added",
  "data": {
    "visitor_id": "019e5c36-53a2-72fc-b287-df210e8e5aae",
    "name": "Demo Name",
    "email": "demo@example.com",
    "phone": "+62812345678"
  },
  "timestamp": "2026-02-19T10:30:00.000Z"
}
```

Catatan: nilai `visitor_id` di-auto-generate oleh runtime dalam format UUID v7, sehingga nilai actual akan berbeda pada setiap eksekusi. Simpan nilai `visitor_id` ini untuk pemakaian pada demo `update`, `delete`, dan `first` di bawah.

#### Verifikasi Cepat: datatables

Setelah `demo-create.bat`, verifikasi record sudah tersimpan via:

```bat
demo-datatables.bat
```

Response menampilkan field `recordsTotal` dan array `data`. Record yang baru saja di-create harus tampil di array tersebut.

#### Eksekusi Demo Action Lain

Action `update`, `delete`, dan `first` memerlukan `visitor_id` valid pada payload-nya. File payload sample (`payload\update.json`, `payload\delete.json`, `payload\first.json`) ter-generate dengan placeholder `"Demo visitor_id"` yang tidak valid.

Sebelum mengeksekusi demo action tersebut, edit file payload dengan UUID actual hasil `demo-create.bat`:

```bat
notepad payload\update.json
notepad payload\delete.json
notepad payload\first.json
```

Ganti nilai placeholder `"Demo visitor_id"` dengan UUID v7 actual, simpan file, lalu jalankan demo:

```bat
demo-first.bat
demo-update.bat
demo-delete.bat
```

Demo `read` dan `lookup` dapat dijalankan langsung tanpa edit payload karena tidak memerlukan `visitor_id` spesifik:

```bat
demo-read.bat
demo-lookup.bat
```

#### Eksekusi Seluruh Demo Sekaligus

Untuk eksekusi sekuensial seluruh action via satu perintah:

```bat
run-all-demos.bat
```

Catatan: tanpa edit payload terlebih dahulu, demo `update`, `delete`, dan `first` pada `run-all-demos.bat` akan mengembalikan response error karena `visitor_id` placeholder tidak valid. Untuk end-to-end success, eksekusi `demo-create.bat` terlebih dahulu, edit payload action lain dengan UUID hasil create, baru jalankan `run-all-demos.bat`.

---

## Langkah Berikutnya (Next Step)

Lanjut ke Skenario 7: test endpoint via Postman.
