# Sandbox Workspace

> Folder ini adalah workspace eksekusi untuk seluruh artefak hasil playbook RESTForge.

---

## Ikhtisar (Overview)

Folder `sandbox/` berfungsi sebagai tempat developer membangun project backend dan frontend selama mengikuti [skenario playbook](../scenarios/). Seluruh isi folder ini di-ignore oleh git (lihat [.gitignore](../.gitignore) pada root) agar tiap developer dapat melakukan eksekusi secara independen tanpa konflik artefak antar contributor.

File `README.md` ini adalah satu-satunya file yang di-track oleh git di dalam folder sandbox.

---

## Struktur Folder (Folder Structure)

Selama mengikuti playbook, struktur berikut akan terbentuk di dalam folder sandbox:

```
sandbox/
├── backend/            Working directory untuk Skenario 1-7
│   ├── config/         File konfigurasi project dan database
│   ├── schema/         Schema Definition File (SDF)
│   ├── payload/        Resource Definition File (RDF)
│   ├── metadata/       Output metadata hasil generate
│   ├── src/            Source code endpoint hasil generate
│   ├── logs/           Log runtime server
│   └── package.json    Manifest npm untuk package @restforgejs/platform
│
└── frontend/           Working directory untuk Skenario 8-9
    ├── payload/        UI Definition File (UDF) hasil migrasi RDF
    └── apps/           Aplikasi frontend hasil generate
        └── visitors-app/
            ├── index.html
            ├── visitors.html
            ├── js/
            ├── css/
            └── app-start.bat
```

Struktur di atas tidak perlu dibuat manual. Folder dan file akan ter-create secara otomatis saat menjalankan perintah RESTForge pada masing-masing skenario.

---

## Cara Memulai (Getting Started)

Folder `backend/` dan `frontend/` dibuat pada Skenario 1. Mulai dari dokumen berikut:

- [Skenario 1: Persiapan Environment dan Instalasi Package](../scenarios/scenario-01-persiapan-dan-instalasi.md)

Setelah Skenario 1 selesai, folder ini akan berisi struktur dasar workspace yang siap digunakan untuk skenario berikutnya.

---

## Aturan Penggunaan (Usage Rules)

| Aturan | Penjelasan |
|--------|------------|
| Jangan commit isi sandbox | Seluruh artefak hasil generate bersifat lokal per developer |
| Jangan modifikasi file ini secara sembarangan | File README.md ini di-track git dan dipakai sebagai placeholder folder |
| Reset workspace boleh dilakukan kapan saja | Hapus folder `backend/` dan `frontend/` untuk memulai ulang playbook dari awal |
| Backup artefak penting secara manual | Karena tidak masuk git, file penting (mis. custom SDF) sebaiknya disalin ke lokasi lain |

---

## Reset Workspace

Untuk memulai playbook dari kondisi bersih, hapus folder `backend/` dan `frontend/` di dalam sandbox:

```powershell
# Windows PowerShell
Remove-Item -Recurse -Force backend, frontend -ErrorAction SilentlyContinue
```

```bash
# Linux / macOS / Git Bash
rm -rf backend frontend
```

Setelah reset, eksekusi dapat dimulai kembali dari Skenario 1.

---

## Referensi (Reference)

- Landing page playbook: [../README.md](../README.md)
- Daftar lengkap skenario: [../scenarios/](../scenarios/)
- Konfigurasi ignore: [../.gitignore](../.gitignore)
