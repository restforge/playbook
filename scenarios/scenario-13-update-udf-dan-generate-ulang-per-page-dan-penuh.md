# Skenario 13: Update UDF dan Generate Ulang Per-Page dan Penuh

> Tahap ketiga belas onboarding RESTForge. Setelah aplikasi multi-page terbentuk (Skenario 12), perubahan pada satu halaman atau pada konfigurasi aplikasi menuntut generate ulang. Skenario ini membahas dua mode generate: per-page (`--scope=form`) dan penuh (`--scope=app`), beserta kapan masing-masing dipakai dan cara menangani `index.html`.

---

## Tujuan (Objective)

1. Memahami perbedaan generate per-page (`--scope=form`) dan penuh (`--scope=app`) serta kriteria pemilihannya
2. Mengedit satu fragmen page lalu meregenerasi hanya halaman tersebut tanpa menyentuh halaman lain
3. Melakukan generate penuh untuk perubahan lintas-page (`app-config.json`, `navigation`, penambahan/penghapusan page) termasuk penanganan `index.html`

---

## Sumber Rujukan (Reference)

| Command | Handbook |
|---------|----------|
| `restforge-designer generate` | `restforge-handbook/commands/restforge-frontend/generate.md` |
| `restforge-designer validate` | `restforge-handbook/commands/restforge-frontend/validate.md` |
| Struktur UDF (`extends`, `pages[].include`) | `restforge-handbook/catalogs/udf/payload-envelope.md` |

---

## Prasyarat (Prerequisite)

| Item | Cara Verifikasi |
|------|-----------------|
| Skenario 12 selesai | Aplikasi multi-page tersedia di `sandbox\frontend\apps\visitors-app`; payload split (`app-config.json` + `pages\*.json` + `visitors-app.json`) ada di `sandbox\frontend\payload` |
| RESTForge Designer ter-install dan license valid | `restforge-designer --version` dan `restforge-designer license status` menampilkan `valid` |
| Working directory aktif di `sandbox\frontend` | Path berakhiran `\playbook\sandbox\frontend` |

---

## Langkah Eksekusi (Execution Steps)

### Langkah 1: Pahami Dua Mode Generate

`restforge-designer generate` memiliki opsi `--scope` dengan dua nilai:

| Mode | Nilai | Cakupan | File yang ditulis |
|------|-------|---------|-------------------|
| Penuh | `--scope=app` (default) | Seluruh aplikasi | Semua page + shared (`index.html`, `js\common.js`, `css\`, `app-start.bat`) |
| Per-page | `--scope=form --page=<pageId>` | Satu halaman | HTML + JS halaman tersebut, plus shared (kecuali ditambah `--skip-shared`) |

Aturan penting yang berlaku pada kedua mode:

- **`--payload` selalu menunjuk ke UDF utama `visitors-app.json`**, bukan ke fragmen `pages\<page>.json`. Fragmen di folder `pages\` tidak memuat `appConfig`, sehingga bila dijadikan payload langsung akan gagal dengan error `appConfig is not defined`. Generator membaca fragmen melalui `include` dari UDF utama, sehingga perubahan pada fragmen otomatis terbawa.
- Generator meng-arsipkan file lama secara otomatis (`*.archive.NNN`) sebelum menimpa, sehingga `--overwrite` aman dijalankan berulang.
- `index.html` tidak ditimpa bila sudah ada (lihat Langkah 5).

#### Ringkasan Pemilihan Mode

| Jenis perubahan | Mode generate | Catatan |
|-----------------|---------------|---------|
| Edit isi satu page di `pages\<page>.json` (label, field, `dataSource`, subtitle) | Per-page (`--scope=form --page=<pageId>`) | Hanya halaman itu yang ditulis; halaman lain dan `index.html` tidak tersentuh |
| Edit beberapa page sekaligus | Penuh (`--scope=app`) | Atau jalankan per-page beberapa kali |
| Edit `app-config.json` (`appName`, `plugin`, `apiBaseUrl`, `port`) | Penuh | Berdampak ke shared files dan seluruh page |
| Tambah/hapus page (ubah `navigation` + `include` di `visitors-app.json`) | Penuh + hapus `index.html` lama | `index.html` tidak ter-overwrite bila sudah ada; daftar card di landing berubah mengikuti set page baru |

---

### Langkah 2: Edit Fragmen Page

Sebagai contoh perubahan dalam-page, edit `pages\visitor-categories.json` dan ubah `pageSubtitle`:

```bat
notepad payload\pages\visitor-categories.json
```

Ubah nilai `pageSubtitle`, misalnya dari `"Manage visitor categories data"` menjadi `"Kelola data kategori pengunjung"`. Perubahan ini bersifat dalam-page (tidak mengubah `navigation` maupun set halaman), sehingga cocok untuk generate per-page.

---

### Langkah 3: Validasi UDF

Validasi selalu dilakukan terhadap UDF utama yang merangkai seluruh fragmen:

```bat
restforge-designer validate --payload=payload/visitors-app.json
```

Output harus `Payload is valid`. Error `does not contain a valid 'pages' array` menandakan fragmen yang diedit kehilangan pembungkus array `pages` (lihat Skenario 12 Langkah 2).

---

### Langkah 4: Generate Ulang Per-Page

Regenerasi hanya halaman `visitor-categories`:

```bat
restforge-designer generate --payload=payload/visitors-app.json --output=./apps/visitors-app --scope=form --page=visitor-categories --overwrite
```

Output menampilkan file yang ditulis ulang, hanya untuk halaman tersebut plus shared:

```
visitor-categories.html
js/visitor-categories.js
js/common.js
css/global-list.css
```

`visitors.html` dan `index.html` tidak tersentuh. Untuk membatasi penulisan hanya pada file halaman (tanpa shared), tambahkan `--skip-shared`:

```bat
restforge-designer generate --payload=payload/visitors-app.json --output=./apps/visitors-app --scope=form --page=visitor-categories --skip-shared --overwrite
```

Verifikasi di browser: jalankan aplikasi (lihat Skenario 9 / Skenario 12 Langkah 6), buka halaman `visitor-categories.html`, dan pastikan perubahan (`pageSubtitle` baru) tampil.

---

### Langkah 5: Generate Ulang Penuh

Generate penuh diperlukan ketika perubahan berdampak lintas-page. Dari `sandbox\frontend`:

```bat
restforge-designer generate --payload=payload/visitors-app.json --output=./apps/visitors-app --overwrite
```

Mode ini dipakai untuk:

- Perubahan pada `app-config.json` (`appName`, `plugin`, `apiBaseUrl`, `port`) karena memengaruhi shared files dan seluruh page
- Perubahan pada banyak page sekaligus
- Penambahan atau penghapusan page

#### Penanganan `index.html` saat Set Page Berubah

`--overwrite` menimpa file page (HTML/JS) dan shared, tetapi **tidak meregenerasi `index.html` yang sudah ada**. Selama set halaman dan `navigation` tidak berubah, hal ini tidak masalah karena `index.html` (daftar card) memang tetap sama.

Namun bila **menambah atau menghapus page** (mengubah `navigation` + `include` di `visitors-app.json`), `index.html` harus diregenerasi agar daftar card sesuai. Karena `index.html` lama tidak ter-overwrite, hapus dulu sebelum generate penuh:

```bat
del apps\visitors-app\index.html
restforge-designer generate --payload=payload/visitors-app.json --output=./apps/visitors-app --overwrite
```

Setelah dihapus, generator membuat ulang `index.html` dengan daftar card yang sesuai set page terkini.

---

## Langkah Berikutnya (Next Step)

Eksplorasi pengembangan lanjutan (master-detail, dashboard, processor, kustomisasi layout) via handbook RESTForge sesuai topik.
