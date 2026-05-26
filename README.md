# RESTForge Playbook

> Panduan onboarding praktis untuk mempelajari RESTForge dari nol hingga aplikasi full-stack berjalan di browser.

Repository ini berisi kumpulan skenario terstruktur yang membimbing developer melalui seluruh alur kerja RESTForge: instalasi, definisi schema, generate endpoint backend, hingga generate aplikasi frontend. Setiap skenario disusun sebagai modul mandiri yang dapat dieksekusi secara berurutan dari Skenario 1 sampai Skenario 9.

---

## Ikhtisar (Overview)

RESTForge adalah platform code generator berbasis definition file (SDF, RDF, UDF) yang menghasilkan endpoint REST API dan aplikasi frontend secara otomatis. Playbook ini didesain sebagai jalur belajar resmi bagi developer yang baru pertama kali berinteraksi dengan toolchain RESTForge.

Target akhir playbook adalah kondisi sebagai berikut:

- Backend server RESTForge berjalan pada port `3000` dengan endpoint REST `visitors` aktif
- Aplikasi frontend berjalan pada port `8000` dan menampilkan data dari backend API
- Pemahaman menyeluruh atas siklus hidup definition file: SDF → database → RDF → endpoint → UDF → frontend

---

## Daftar Skenario (Scenarios)

| No | Judul Skenario | Output Utama |
|----|----------------|--------------|
| 01 | Persiapan Environment dan Instalasi Package | Struktur folder sandbox terbentuk, `@restforgejs/platform` terpasang |
| 02 | Inisialisasi Project dan Konfigurasi Database | File konfigurasi project dan koneksi database siap pakai |
| 03 | Pembuatan dan Validasi SDF | Schema Definition File ter-validasi sesuai spec handbook |
| 04 | Apply SDF ke Database dan Drift Detection | Tabel database ter-create sesuai SDF, drift terdeteksi |
| 05 | Pembuatan dan Validasi RDF | Resource Definition File ter-validasi |
| 06 | Generate Endpoint dan Jalankan Runtime Server | Endpoint REST API ter-generate, runtime server berjalan |
| 07 | Test Endpoint via Postman | Verifikasi endpoint GET/POST/PUT/DELETE melalui Postman |
| 08 | Migrasi RDF ke UDF dan Generate Aplikasi Frontend | UDF terbentuk, aplikasi frontend ter-generate |
| 09 | Menjalankan Aplikasi Frontend dan Verifikasi GET Visitors | Halaman `visitors.html` memuat data dari backend |

Seluruh file skenario tersedia di folder [scenarios/](scenarios/).

---

## Struktur Folder (Folder Structure)

```
restforge-playbook/
├── scenarios/          Dokumentasi skenario onboarding (Skenario 1-9)
├── sandbox/            Workspace eksekusi untuk artefak hasil playbook
│   ├── backend/        Working directory untuk Skenario 1-7 (akan dibuat saat eksekusi)
│   └── frontend/       Working directory untuk Skenario 8-9 (akan dibuat saat eksekusi)
└── README.md           Dokumen ini
```

Folder `sandbox/` sengaja di-ignore dari git agar setiap developer dapat mengeksekusi playbook secara independen tanpa konflik artefak. Detail isi sandbox dijelaskan pada [sandbox/README.md](sandbox/README.md).

---

## Prasyarat (Prerequisite)

| Item | Versi / Keterangan |
|------|--------------------|
| Node.js dan npm | Versi LTS terbaru, dapat dipanggil via terminal |
| Database server | PostgreSQL atau MySQL (sesuai konfigurasi Skenario 2) |
| Postman | Untuk pengujian endpoint pada Skenario 7 |
| Browser modern | Chrome, Edge, atau Firefox untuk verifikasi frontend pada Skenario 9 |
| Akses internet | Koneksi aktif ke registry npm untuk download package |

---

## Cara Memulai (Getting Started)

1. Clone repository ini ke local machine
2. Buka folder `restforge-playbook/` melalui terminal
3. Mulai dari [scenarios/scenario-01-persiapan-dan-instalasi.md](scenarios/scenario-01-persiapan-dan-instalasi.md)
4. Ikuti instruksi setiap skenario secara berurutan
5. Seluruh artefak hasil eksekusi akan tersimpan di folder `sandbox/`

---

## Konvensi (Conventions)

- Seluruh perintah CLI mengikuti pattern `npx restforge <resource> <verb> [--flag=value]`
- Reference resmi tersedia di [RESTForge Handbook](https://github.com/restforge/handbook)
- Definition file menggunakan format JSON dengan validasi schema bawaan
- Working directory aktif untuk eksekusi backend adalah `sandbox/backend/`
- Working directory aktif untuk eksekusi frontend adalah `sandbox/frontend/`

---

## Referensi (Reference)

| Topik | Sumber |
|-------|--------|
| RESTForge Handbook | https://github.com/restforge/handbook |
| Spec SDF | `handbook/catalogs/sdf/` |
| Spec RDF | `handbook/catalogs/rdf/` |
| Spec UDF | `handbook/catalogs/udf/` |
| CLI Reference | `handbook/commands/` |

---

## Lisensi (License)

Materi playbook dirilis sebagai dokumentasi pembelajaran internal RESTForge Systems. Hak cipta atas implementasi platform RESTForge mengikuti lisensi yang ditetapkan pada repository `@restforgejs/platform`.
