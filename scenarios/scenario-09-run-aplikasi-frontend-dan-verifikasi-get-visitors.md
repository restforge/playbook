# Skenario 9: Menjalankan Aplikasi Frontend dan Verifikasi GET Visitors

> Tahap kesembilan onboarding RESTForge. Menjalankan aplikasi frontend hasil generate Skenario 8 bersamaan dengan backend server, lalu memverifikasi bahwa halaman `visitors` dapat memuat data dari backend API melalui browser.

---

## Tujuan (Objective)

1. Backend server RESTForge running di port `3000` dari workspace `sandbox\backend`
2. Aplikasi frontend running di port `8000` dari workspace `sandbox\frontend\apps\visitors-app`
3. Halaman `visitors.html` di browser memuat list visitors melalui call ke endpoint backend `http://127.0.0.1:3000/api/visitors-app/visitors`

---

## Prasyarat (Prerequisite)

| Item | Cara Verifikasi |
|------|-----------------|
| Skenario 8 selesai | Folder `sandbox\frontend\apps\visitors-app\` berisi `index.html`, `visitors.html`, `app-start.bat`, folder `js\`, `css\` |
| Backend server running | `serve --project=visitors-app` dari Skenario 6 Langkah 4 masih aktif; `netstat -ano \| findstr :3000` menampilkan entri `LISTENING` |
| Port `8000` tersedia | `netstat -ano \| findstr :8000` tidak menampilkan entri `LISTENING` |
| Browser tersedia | Chrome, Edge, Firefox, atau browser modern lainnya |

---

## Langkah Eksekusi (Execution Steps)

### Langkah 1: Pastikan Backend Server Running

Skenario ini berfokus pada aplikasi frontend. Backend server merupakan prasyarat yang sudah dijalankan pada Skenario 6 (`serve --project=visitors-app`), bukan dijalankan ulang di sini. Langkah ini hanya memastikan server tersebut masih aktif di port `3000`.

Pada jendela cmd, verifikasi port `3000` sudah berstatus `LISTENING`:

```bat
netstat -ano | findstr :3000
```

Output harus menampilkan entri `LISTENING` pada `127.0.0.1:3000`. Sebagai konfirmasi tambahan, endpoint health dapat diakses melalui browser:

```
http://127.0.0.1:3000/api/visitors-app/health
```

Bila port `3000` belum `LISTENING`, jalankan backend server terlebih dahulu mengikuti Skenario 6 Langkah 4, lalu biarkan jendela cmd tersebut tetap terbuka selama Skenario 9 berjalan.

---

### Langkah 2: Start Aplikasi Frontend

Buka **jendela cmd kedua** (jangan tutup cmd backend). Pindah ke folder aplikasi frontend lalu jalankan `app-start.bat`:

```bat
cd playbook\sandbox\frontend\apps\visitors-app
app-start.bat
```

Output yang diharapkan:

```
Starting application...
Open http://localhost:8000/index.html in your browser
```

Selanjutnya `npx serve` akan menampilkan log siap-melayani pada port `8000`. Biarkan jendela cmd ini juga tetap terbuka.

---

### Langkah 3: Verifikasi GET Visitors di Browser

Buka browser, akses URL berikut:

```
http://localhost:8000/index.html
```

Halaman homepage aplikasi harus tampil. Navigasikan ke halaman `Visitors` (link tersedia pada homepage), atau akses langsung:

```
http://localhost:8000/visitors.html
```

Verifikasi kondisi berikut pada halaman `Visitors`:

| Aspek | Kondisi |
|-------|---------|
| Tabel data | Ter-render dengan kolom `Name`, `Email`, `Phone` (sesuai UDF `visitors.json`) |
| Pemanggilan API | Tab Network di DevTools (F12) menampilkan request ke `http://127.0.0.1:3000/api/visitors-app/visitors` dengan response status `200` |
| Data ter-load | Jika data visitors sudah ada (mis. hasil Create dari Skenario 7), list muncul di tabel. Jika belum ada data, tabel menampilkan pesan empty state tanpa error |
| Console browser | Tab Console di DevTools tidak menampilkan error fatal (mis. `Failed to fetch`, `CORS error`) |

#### Pengujian CRUD Mandiri via Aplikasi Web

Setelah halaman `Visitors` berhasil ter-render dengan data, lanjutkan dengan pengujian operasi CRUD (Create, Read, Update, Delete) secara mandiri langsung melalui aplikasi web yang sudah terbuka di browser. Tujuannya adalah memvalidasi end-to-end integrasi frontend ke backend pada seluruh action endpoint, bukan hanya `GET/datatables`.

Skenario pengujian yang disarankan:

| Action | Langkah Pengujian | Verifikasi Hasil |
|--------|-------------------|------------------|
| Create | Klik tombol **Add** atau **Create** pada halaman `Visitors`. Isi form dengan data baru (`Name`, `Email`, `Phone`) lalu submit | Record baru muncul di tabel tanpa reload manual; tab Network menampilkan request `POST /api/visitors-app/visitors/create` dengan response `success: true` |
| Read (detail) | Klik salah satu baris record (atau tombol **View**/**Detail**) untuk melihat detail | Form detail menampilkan nilai `Name`, `Email`, `Phone` sesuai record; tab Network menampilkan request `POST /api/visitors-app/visitors/first` atau `/read` dengan response `200` |
| Update | Klik tombol **Edit** pada baris record yang sudah ada. Ubah salah satu field (mis. `Name` atau `Phone`) lalu submit | Nilai record di tabel ter-update tanpa reload; tab Network menampilkan request `POST /api/visitors-app/visitors/update` dengan response `success: true` |
| Delete | Klik tombol **Delete** pada baris record. Konfirmasi dialog hapus apabila tersedia | Record hilang dari tabel; tab Network menampilkan request `POST /api/visitors-app/visitors/delete` dengan response `deleted_count: 1` |

Verifikasi tambahan selama pengujian CRUD:

- Tab **Console** browser tetap bersih tanpa error JavaScript fatal
- Tab **Network** browser menampilkan seluruh request CRUD dengan response status `200`/`201` (bukan `4xx`/`5xx`)
- Validasi sisi frontend (mis. field wajib, format email) berfungsi dan menampilkan pesan inline ketika input tidak valid
- Validasi sisi backend (mis. duplicate email apabila ada unique constraint) menghasilkan response error yang ditangani oleh frontend (mis. alert/toast)
- Setelah operasi Create dan Delete, refresh halaman untuk memastikan data persisten di database (bukan hanya state in-memory frontend)

Catatan: layout dan label tombol CRUD pada aplikasi hasil generate mengikuti template default RESTForge Designer. Apabila ditemukan tombol action belum tersedia atau perlu kustomisasi tambahan (mis. konfirmasi delete, success notification), kustomisasi dilakukan via editing manual UDF (`sandbox\frontend\payload\visitors.json`) lalu regenerate aplikasi via `restforge-designer generate --overwrite`.

---

### Langkah 4 (Opsional): Stop atau Lanjutkan

Stop tidak wajib pada tahap ini. Backend server dan aplikasi frontend dapat dibiarkan tetap berjalan untuk dilanjutkan ke Skenario 10 dan seterusnya. Hentikan kedua process hanya bila ingin mengakhiri sesi, dengan menekan `Ctrl + C` pada masing-masing cmd (frontend pada cmd kedua, backend pada cmd pertama).

---

## Langkah Berikutnya (Next Step)

Lanjut ke Skenario 10: tabel visitor_categories dari SDF hingga endpoint.
