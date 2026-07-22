---
name: git-push
description: "Cek perubahan file terakhir di staging area, rekomendasikan commit message yang relevan dalam Bahasa Indonesia, dan commit saja dulu"
license: MIT
---

# Git Push Skill (/git-push)

Use this skill when the user triggers the `/git-push` command or asks to commit and push their changes.

## Workflow

### 1. Cek Staging Area
- Jalankan perintah `git diff --cached` atau `git status` untuk melihat daftar file yang berada di staging area (staged changes).
- Jika staging area kosong (tidak ada file yang di-stage), jalankan `git status` untuk mendeteksi perubahan yang belum di-stage (unstaged changes) dan tanyakan kepada user apakah mereka ingin men-stage perubahan tersebut terlebih dahulu (misal: `git add .`).

### 2. Analisis Perubahan
- Periksa detail perubahan pada file yang telah di-stage menggunakan `git diff --cached` (batasi output jika terlalu panjang agar tidak menghabiskan token context).
- Pahami tujuan dari perubahan kode tersebut (apakah menambahkan fitur baru, memperbaiki bug, refaktorisasi, melakukan upgrade versi, dll.).

### 3. Rekomendasikan Commit Message
Generate beberapa rekomendasi commit message dalam **Bahasa Indonesia** mengikuti format Conventional Commits yang rapi. Berikan minimal 2-3 variasi yang relevan dengan format berikut:
- **`feat(<nama-fitur>) : <deskripsi>`**: jika menambahkan fitur baru.
- **`fix(<nama-fix>) : <deskripsi>`**: jika memperbaiki bug atau error.
- **`refactor(<nama-komponen>) : <deskripsi>`**: jika merestrukturisasi atau merapikan kode tanpa mengubah fungsionalitas.
- **`upgrade(chore) : <deskripsi>`**: jika melakukan upgrade versi dependensi atau konfigurasi lingkungan (environment/seeding/testing).

### 4. Konfirmasi dan Eksekusi
- Tampilkan opsi rekomendasi tersebut ke user dan minta konfirmasi pilihan mereka (atau izinkan user menulis kustom pesan sendiri).
- Setelah user memilih atau memberikan persetujuan pesan commit:
  1. Jalankan `git commit -m "<pesan-commit-yang-dipilih>"`
- Laporkan status keberhasilan commit ke user.
