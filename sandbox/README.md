# Sandbox Workspace

> Folder ini adalah area kerja (workspace) untuk seluruh artefak hasil eksekusi playbook RESTForge.

---

## Ikhtisar (Overview)

Folder `sandbox/` adalah tempat membangun project backend dan frontend selama mengikuti [skenario playbook](../scenarios/). Folder `backend/` dan `frontend/` beserta seluruh isinya **ter-create otomatis** saat menjalankan perintah RESTForge pada tiap skenario, jadi tidak perlu dibuat manual.

Seluruh isi folder ini di-ignore oleh git (lihat [.gitignore](../.gitignore) pada root) agar tiap developer dapat bereksekusi secara independen tanpa konflik artefak. File `README.md` ini adalah satu-satunya file yang di-track git di dalam sandbox.

Rincian artefak yang dihasilkan tiap skenario tidak diuraikan di sini; acuannya adalah daftar skenario pada [../README.md](../README.md).

---

## Reset Workspace

Karena sandbox murni area kerja, cara membersihkannya cukup sederhana: **hapus folder `backend/` dan `frontend/`, lalu jalankan ulang dari Skenario 1**. Folder akan ter-create kembali secara otomatis saat skenario dieksekusi.

```cmd
:: Windows CMD
rmdir /S /Q backend
rmdir /S /Q frontend
```

```bash
# Linux / macOS / Git Bash
rm -rf backend frontend
```

Catatan: penghapusan bersifat permanen dan artefak sandbox tidak masuk git. Bila ada file custom penting (mis. SDF buatan sendiri), salin dulu ke lokasi lain sebelum menghapus.

---

## Referensi (Reference)

- Landing page playbook: [../README.md](../README.md)
- Daftar lengkap skenario: [../scenarios/](../scenarios/)
