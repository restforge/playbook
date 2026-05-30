# Skenario 1: Persiapan Environment dan Instalasi Package

> Tahap pertama onboarding RESTForge. Tujuan akhir skenario ini adalah struktur folder `sandbox\backend\` dan `sandbox\frontend\` terbentuk, package `@restforgejs/platform` terpasang pada folder `sandbox\backend\`, dan binary `restforge` dapat dipanggil melalui `npx`.

---

## Tujuan (Objective)

Setelah menyelesaikan skenario ini, kondisi berikut harus tercapai:

1. Folder `sandbox\backend\` dan `sandbox\frontend\` sudah dibuat sejak awal sebagai workspace pemisah artefak backend dan frontend
2. Folder `sandbox\backend\` menjadi working directory aktif untuk seluruh aktivitas onboarding backend (Skenario 1-7)
3. Package `@restforgejs/platform` ter-install sebagai dependency lokal di dalam folder `sandbox\backend\`
4. Binary `restforge` dapat dipanggil via `npx restforge` dan menampilkan versi terpasang

---

## Sumber Rujukan (Reference)

Seluruh perintah pada skenario ini bersumber dari handbook berikut:

| Topik | File Handbook |
|-------|---------------|
| Perintah instalasi package | [README.md](https://github.com/restforge/handbook/blob/main/README.md) (section Quickstart) |
| Konvensi CLI dan global options | [commands/conventions.md](https://github.com/restforge/handbook/blob/main/commands/conventions.md) |
| Daftar resource CLI | [commands/README.md](https://github.com/restforge/handbook/blob/main/commands/README.md) |

---

## Prasyarat (Prerequisite)

Pemeriksaan berikut perlu dilakukan sebelum melanjutkan ke langkah eksekusi:

| Item | Cara Verifikasi |
|------|-----------------|
| Node.js dan npm terpasang | Jalankan `node --version` dan `npm --version` pada cmd |
| Akses internet ke registry npm | Konfirmasi koneksi internet tersedia untuk download package dari registry |
| Folder root project dapat diakses dan writable | Folder `<root-project>` dapat diakses dan memiliki permission tulis |
| PostgreSQL server terpasang dan dapat diakses | Jalankan `psql --version` pada cmd untuk memastikan PostgreSQL terpasang, serta tersedia host, port, user, dan password yang siap digunakan. Item ini bersifat antisipatif: tidak digunakan pada Skenario 1, namun akan dibutuhkan mulai Skenario 2 untuk validasi koneksi database |

Keterangan placeholder:

- `<root-project>` merujuk pada folder root playbook tempat seluruh aktivitas onboarding dilakukan. Resolusi path absolut dipegang oleh user dan tidak ditulis pada dokumen.

Catatan: handbook tidak menetapkan versi minimum Node.js secara eksplisit. Jika ditemukan error kompatibilitas saat instalasi, hentikan eksekusi dan dokumentasikan permasalahan sebelum mencari solusi alternatif.

Catatan PostgreSQL: instalasi PostgreSQL berada di luar scope playbook ini. Installer resmi tersedia pada [https://www.postgresql.org/download/](https://www.postgresql.org/download/). Penyiapan PostgreSQL sebelum Skenario 1 selesai akan mencegah interupsi onboarding saat memasuki Skenario 2.

---

## Langkah Eksekusi (Execution Steps)

### Langkah 1: Membuat Struktur Folder Sandbox

Sandbox playbook dipisah menjadi dua workspace sejak awal: `backend\` untuk seluruh artefak RESTForge backend (Skenario 1-7) dan `frontend\` untuk artefak RESTForge Designer (Skenario 8-9). Pemisahan ini ditetapkan sejak awal agar tidak diperlukan reorganisasi folder di tengah onboarding.

Lokasi struktur sandbox berada di dalam folder root project pada path relatif `<root-project>\playbook\sandbox`, dengan sub-folder:

| Path | Peran |
|------|-------|
| `<root-project>\playbook\sandbox\backend` | Working directory untuk Skenario 1-7 (instalasi package, init project, SDF, RDF, endpoint, runtime) |
| `<root-project>\playbook\sandbox\frontend` | Workspace kosong yang akan digunakan pada Skenario 8 untuk RESTForge Designer |

Buka cmd, pindah ke folder root project terlebih dahulu, lalu jalankan perintah berikut untuk membuat kedua sub-folder sekaligus:

```bat
mkdir playbook\sandbox\backend
mkdir playbook\sandbox\frontend
```

Setelah folder dibuat, pindah ke folder `backend\` sebagai working directory aktif:

```bat
cd playbook\sandbox\backend
```

Verifikasi posisi working directory:

```bat
cd
```

Output harus menunjukkan path yang berakhiran `\playbook\sandbox\backend` relatif terhadap folder root project.

---

### Langkah 2: Instalasi Package @restforgejs/platform

Perintah instalasi berikut bersumber dari handbook `README.md` section Quickstart:

```bat
npm install @restforgejs/platform
```

Proses instalasi akan men-download package beserta dependency dari registry npm. Tunggu hingga proses selesai tanpa error.

---

### Langkah 3: Verifikasi Binary restforge

Binary `restforge` mendukung flag global `--version` (`-v`) untuk menampilkan versi terpasang dan `--help` (`-h`) untuk menampilkan help message.

Jalankan verifikasi versi:

```bat
npx restforge --version
```

Output harus menampilkan nomor versi package yang ter-install.

Lanjutkan dengan verifikasi help message:

```bat
npx restforge --help
```

Output harus menampilkan daftar resource dan verb yang tersedia (runtime, schema, endpoint, payload, processor, dashboard, kafka, test, key, query, config, catalog, project).

---

## Langkah Berikutnya (Next Step)

Lanjut ke Skenario 2: inisialisasi project dan konfigurasi database.

