# Skenario 2: Inisialisasi Project dan Konfigurasi Database

> Tahap kedua onboarding RESTForge. Tujuan akhir skenario ini adalah starter files ter-generate, file `config/db-connection.env` terisi credential PostgreSQL yang valid, serta lolos perintah `restforge validate`.

---

## Tujuan (Objective)

Setelah menyelesaikan skenario ini, kondisi berikut harus tercapai:

1. Starter files RESTForge sudah ter-generate di dalam folder `sandbox\backend`
2. File `config/db-connection.env` sudah terisi dengan credential PostgreSQL yang valid
3. Perintah `npx restforge validate --config=db-connection.env` mengembalikan status sukses
4. File config sudah di-set sebagai default config untuk working directory

---

## Sumber Rujukan (Reference)

Seluruh perintah pada skenario ini bersumber dari handbook berikut:

| Topik | File Handbook |
|-------|---------------|
| Verb global `init` dan daftar file ter-generate | [commands/init.md](https://github.com/restforge/handbook/blob/main/commands/init.md) |
| Validasi config (koneksi database, Redis, Kafka, license) | [commands/runtime/validate.md](https://github.com/restforge/handbook/blob/main/commands/runtime/validate.md) |
| Set default config per working directory | [commands/config/set-default.md](https://github.com/restforge/handbook/blob/main/commands/config/set-default.md) |
| Daftar file `.env` di cwd dan folder `config/` | [commands/config/list.md](https://github.com/restforge/handbook/blob/main/commands/config/list.md) |
| Tampilkan default config aktif | [commands/config/get-default.md](https://github.com/restforge/handbook/blob/main/commands/config/get-default.md) |

---

## Prasyarat (Prerequisite)

| Item | Cara Verifikasi |
|------|-----------------|
| Skenario 1 sudah selesai | Folder `sandbox\backend` sudah berisi `node_modules\@restforgejs\platform`, dan folder `sandbox\frontend` sudah ter-create (kosong) |
| Working directory aktif di `sandbox\backend` | Jalankan `cd` pada cmd, path harus berakhiran `\playbook\sandbox\backend` |
| PostgreSQL server tersedia dan dapat diakses | Tersedia host, port, user, password, dan nama database kosong yang siap digunakan |

Catatan: handbook menyatakan bahwa perintah `restforge validate` akan memvalidasi koneksi database secara aktual. Tanpa PostgreSQL server yang dapat diakses, perintah validasi akan gagal.

---

## Langkah Eksekusi (Execution Steps)

### Langkah 1: Verifikasi Working Directory

Pastikan posisi cmd berada di folder `sandbox\backend` sebelum mengeksekusi perintah berikutnya:

```bat
cd
```

Output harus menunjukkan path yang berakhiran `\playbook\sandbox\backend`. Jika belum berada di folder tersebut, pindah ke folder backend dari folder root project:

```bat
cd playbook\sandbox\backend
```

---

### Langkah 2: Generate Starter Files

Verb global `init` akan men-generate satu file berikut di working directory:

| Path | Keterangan |
|------|-----------|
| `config\db-connection.env` | Template database connection config |

Eksekusi perintah:

```bat
npx restforge init
```

Fokus pada skenario ini adalah setup konfigurasi database, sehingga verifikasi cukup dilakukan pada file `config\db-connection.env`:

```bat
dir config\db-connection.env
```

Perintah `dir` harus mengembalikan entri file tanpa error "File Not Found". 
File `payload\samples.json` dan `payload\query\samples-datatables.sql` yang juga ter-generate pada langkah ini dapat dihapus karena akan dibahas pada skenario selanjutnya yang berkaitan dengan RDF.

---

### Langkah 3: Edit Credential PostgreSQL dan License

Buka file `config\db-connection.env` menggunakan VS Code:

```bat
code config\db-connection.env
```

Template ter-generate berisi banyak section (License, Server, Live Sync, Redis, Export, Kafka, Database, Logging, SQL Logging, Cache, Job Scheduler, Distributed Lock, ID Generator). Pada onboarding awal, fokus penyesuaian hanya pada **dua section**: License dan Database. 
Section lain biarkan pada nilai default karena fitur opsional sudah disabled secara default (`LIVE_SYNC_ENABLED=false`, `KAFKA_ENABLED=false`, `CACHE_ENABLED=false`, `JOB_ENABLED=false`, `LOCK_DISTRIBUTED_ENABLED=false`, `IDGEN_ENABLED=false`).

#### Section License

Section License berada di baris pertama template:

```env
# License
LICENSE=XXXX-XXXX-XXXX-XXXX
```

License key tidak disediakan sebagai dummy oleh RESTForge. License key resmi diperoleh melalui halaman aktivasi pada [https://restforge.dev](https://restforge.dev). Ikuti instruksi yang tertera pada halaman tersebut untuk:

1. Membuat akun atau melakukan login
2. Memilih paket lisensi yang sesuai (trial, developer, atau production sesuai ketersediaan)
3. Mendapatkan license key dalam format `XXXX-XXXX-XXXX-XXXX`

Setelah license key diperoleh, ganti placeholder `XXXX-XXXX-XXXX-XXXX` dengan license key yang valid.

#### Section Database

Section Database berada pada blok berikut dengan default sudah di-set untuk PostgreSQL. Isi nilai sesuai PostgreSQL server yang tersedia:

```env
# Database Configuration
# Supported: postgresql, mysql, oracle, sqlite
DB_TYPE=postgresql
DB_HOST=127.0.0.1
DB_PORT=5432
DB_USER=postgres
DB_PASSWORD=your_password_here
DB_NAME=your_database_name
```

Pemetaan parameter:

| KEY | Contoh Nilai | Keterangan |
|-----|--------------|-----------|
| `DB_TYPE` | `postgresql` | Dalam skenario ini tetap gunakan `postgresql` meskipun begitu nilai DB_TYPE yang didukung adalah `postgresql`, `mysql`, `oracle`, `sqlite` |
| `DB_HOST` | `127.0.0.1` atau `localhost` | Host PostgreSQL server. Gunakan IP server bila PostgreSQL tidak berada di workstation yang sama |
| `DB_PORT` | `5432` | Port default PostgreSQL |
| `DB_USER` | `postgres` | Username PostgreSQL yang memiliki permission CREATE/ALTER pada database target |
| `DB_PASSWORD` | (password user PostgreSQL) | Ganti `your_password_here` dengan password aktual |
| `DB_NAME` | (nama database yang sudah disiapkan) | Ganti `your_database_name` dengan nama database kosong yang siap di-migrate |

Catatan penting:

- Database yang sesuai dengan nama di `DB_NAME` tidak harus dibuat secara manual. RESTForge memiliki fitur auto-create database untuk PostgreSQL dan MySQL yang akan dibahas pada Langkah 4 di bawah.
- User pada `DB_USER` harus memiliki permission untuk membuat database, table, dan menjalankan DDL pada server PostgreSQL.
- Pastikan tidak ada KEY yang dihapus dari template. Cukup ubah VALUE pada KEY yang relevan.

Setelah selesai editing, simpan file dan lanjutkan ke langkah berikutnya.

---

### Langkah 4: Validasi Config dan Auto-Create Database

Perintah `validate` melakukan preflight check terhadap koneksi database, Redis (jika fitur cache/idgen/lock/idempotency/rate-limit/live-sync aktif), Kafka (jika `KAFKA_ENABLED`), dan status license. Pada skenario ini fokus utama adalah koneksi database PostgreSQL.

Eksekusi perintah validasi:

```bat
npx restforge validate --config=db-connection.env
```

#### Output Sukses

Apabila database pada `DB_NAME` sudah ada dan dapat diakses, output akan menampilkan `Status: [OK] Connection successful` untuk Database Connection dan exit code `0`.

#### Kondisi Output yang belum memiliki Database

Apabila database belum ada pada server, `validate` akan menampilkan output seperti berikut:

```
[1] Database Connection
    Type:     postgresql
    Host:     127.0.0.1
    Port:     5432
    User:     postgres
    Database: <nama-database>
    Status:   [FAIL] database "<nama-database>" does not exist

Database '<nama-database>' was not found on the postgresql server.
Do you want to create a new database? (y/N):
```

Pada prompt tersebut, ketik `y` lalu tekan Enter untuk mengizinkan RESTForge membuat database baru secara otomatis. RESTForge akan terhubung ke database administratif `postgres` menggunakan credential yang sama, lalu mengeksekusi `CREATE DATABASE <nama-database>`.

Setelah database berhasil dibuat, command akan menampilkan `[OK] Database '<nama-database>' created successfully.` diikuti instruksi `Please re-run the command:` dan exit `0`. Jalankan kembali perintah validasi:

```bat
npx restforge validate --config=db-connection.env
```

Pada eksekusi kedua, koneksi database harus berstatus `[OK]` karena database sudah ter-create pada eksekusi pertama.

#### Mode Non-Interaktif (CI/CD)

Untuk konteks non-interaktif (mis. CI/CD pipeline), gunakan flag `--auto-create-db` agar RESTForge langsung membuat database tanpa prompt:

```bat
npx restforge validate --config=db-connection.env --auto-create-db
```

#### Troubleshooting

Apabila prompt auto-create dijawab `y` tetapi RESTForge mengembalikan error `Error: User '<name>' does not have privilege to create a database.`, hal tersebut menandakan user PostgreSQL pada `DB_USER` tidak memiliki permission `CREATE DATABASE`. Solusinya:

1. Hubungi administrator PostgreSQL untuk grant privilege, atau
2. Gunakan user PostgreSQL lain yang memiliki privilege (mis. superuser `postgres`), atau
3. Minta administrator membuat database secara manual lalu ulangi validasi tanpa auto-create

---

### Langkah 5: Set Default Config

Command yang menerima `--config` (resource `payload`, `schema`, `query`) akan menggunakan default config dari `.restforge\defaults.json` apabila parameter `--config` tidak ditulis secara eksplisit. 
Setting default mengurangi pengulangan flag pada perintah berikutnya.

Set file config sebagai default:

```bat
npx restforge config set-default --config=db-connection.env
```

Verifikasi default config yang aktif:

```bat
npx restforge config get-default
```

Output harus menampilkan path `config\db-connection.env` sebagai default aktif untuk working directory.

---

### Langkah 6: Verifikasi Daftar File Config

Tampilkan daftar file `.env` yang terdeteksi di working directory dan folder `config/`:

```bat
npx restforge config list
```

Output harus mencantumkan `db-connection.env` sebagai salah satu entri.

---

## Langkah Berikutnya (Next Step)

Lanjut ke Skenario 3: pembuatan dan validasi SDF.

