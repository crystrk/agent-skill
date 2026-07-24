---
name: fitur-planner
description: >
  Generate dokumen perencanaan fitur baru yang aligned dengan PRD yang sudah ada.
  Gunakan skill ini setiap kali user mengetik /fitur-planner, meminta rencana fitur baru,
  ingin menambah fitur ke produk yang sudah ada PRD-nya, atau menyebut kata kunci
  seperti "tambah fitur", "buat fitur", "planning fitur", "feature planning",
  "dokumen fitur", atau "spec fitur". Skill ini membaca PRD yang ada lalu menghasilkan
  FEATURE.md yang konsisten dan tidak lari dari scope produk. Skill ini juga membaca
  repository agar rencana teknis mengikuti arsitektur Laravel, Vue, Astro, dan tooling
  yang benar-benar digunakan proyek.
---

# Fitur Planner Skill

Skill ini membantu user merencanakan fitur baru dalam bentuk dokumen markdown terstruktur,
dengan selalu mengacu pada PRD yang sudah ada agar fitur tetap aligned dengan visi produk.

---

## Cara Trigger

User memanggil skill ini dengan format:

```
/fitur-planner [deskripsi singkat fitur] yang alurnya [alur atau flow]
```

Contoh:
```
/fitur-planner saya ingin buat fitur notifikasi email yang alurnya user melakukan aksi → sistem kirim email → user konfirmasi
```

---

## Langkah-langkah Eksekusi

### 1. Baca PRD yang Ada

Cari file PRD di direktori kerja user. Kemungkinan lokasi:
- `PRD.md`
- `docs/PRD.md`
- `prd.md`
- `docs/prd.md`
- `planning/PRD.md`

Jika tidak ditemukan, tanyakan ke user: *"Saya tidak menemukan PRD di direktori ini. Bisa share path PRD-nya atau paste isi PRD-nya?"*

Jika PRD ditemukan, baca dan ekstrak:
- **Tujuan produk** (vision & goals)
- **Target user** (personas)
- **Fitur yang sudah ada / scope saat ini**
- **Non-goals / out of scope**
- **Tech stack** (jika disebutkan)
- **Constraints** (waktu, resource, teknis)

### 2. Deteksi Stack dan Arsitektur Repository

Jangan hanya mengandalkan tech stack yang tertulis di PRD. Periksa repository untuk memahami implementasi yang benar-benar digunakan:

- `composer.json` dan `composer.lock` untuk versi Laravel, PHP, package backend, serta Pest/PHPUnit
- `package.json` dan lockfile untuk Vue, Astro, Inertia, state management, testing, dan tooling frontend
- `vite.config.*`, `astro.config.*`, `phpunit.xml`, `pest.php`, `vitest.config.*`, atau konfigurasi Playwright/Cypress jika tersedia
- Struktur `routes/`, `app/`, `database/`, `resources/js/`, dan `src/` untuk mengenali pola dan konvensi proyek

Klasifikasikan arsitektur yang ditemukan, misalnya:

- Laravel + Blade
- Laravel + Inertia + Vue
- Laravel API + Vue SPA
- Laravel API + Astro
- Astro static site atau Astro SSR

Catat versi berdasarkan manifest/lockfile. Jangan menganggap proyek memakai versi "terbaru" atau menyarankan package baru jika kebutuhan dapat diselesaikan dengan kemampuan dan pola yang sudah ada. Jika repository belum memiliki kode atau manifest, tandai stack sebagai asumsi atau open question.

### 3. Parse Input User

Dari command `/fitur-planner`, ekstrak:
- **Nama fitur**: apa yang ingin dibangun
- **Alur / flow**: bagaimana fitur bekerja dari sisi user
- **Konteks tambahan**: siapa yang pakai, kenapa dibutuhkan (jika ada)

### 4. Validasi Alignment dengan PRD

Sebelum generate dokumen, lakukan quick check:

| Check | Pertanyaan |
|-------|------------|
| **Scope alignment** | Apakah fitur ini masuk dalam scope produk? |
| **User alignment** | Apakah target user fitur ini sama dengan user di PRD? |
| **Goal alignment** | Apakah fitur ini mendukung tujuan produk? |
| **Conflict check** | Apakah ada fitur yang sudah ada yang bertabrakan? |
| **Architecture fit** | Apakah rancangan mengikuti stack dan pola repository yang sudah ada? |

Jika ada **misalignment**, sampaikan ke user sebelum generate:
> "⚠️ Saya perhatikan fitur ini mungkin di luar scope PRD karena [alasan]. Apakah kamu ingin tetap lanjut atau menyesuaikan?"

### 5. Generate FEATURE.md

Buat file dengan nama `FEATURE-[nama-fitur-slug].md` di folder `<root>/repo/fitur/`, terlepas dari lokasi file PRD.

- Anggap `<root>` sebagai root workspace atau project yang sedang dikerjakan.
- Buat folder `repo/fitur/` terlebih dahulu jika belum ada.
- Gunakan slug huruf kecil dengan pemisah tanda hubung, misalnya `repo/fitur/FEATURE-notifikasi-email.md`.

Gunakan template di bawah ini:

---

## Template FEATURE.md

```markdown
# Feature: [Nama Fitur]

> Dokumen ini adalah rencana pengembangan fitur [Nama Fitur].  
> Dibuat berdasarkan PRD: [nama file PRD] | Versi: [tanggal hari ini]

---

## 1. Overview

**Nama Fitur:** [Nama Fitur]  
**Status:** Draft  
**Priority:** [High / Medium / Low — tanyakan ke user atau infer dari konteks]  
**Epic/Module:** [Modul mana di produk ini fitur ini masuk]  
**Detected Stack:** [Versi dan integrasi yang terdeteksi, contoh: Laravel 12 + Inertia + Vue 3]

### Problem Statement
[Masalah apa yang fitur ini selesaikan? Infer dari input user + konteks PRD]

### Proposed Solution
[Solusi singkat dalam 2-3 kalimat]

---

## 2. Alignment dengan PRD

| Aspek | Keterangan |
|-------|------------|
| **Product Goal** | [Goal dari PRD yang didukung fitur ini] |
| **Target User** | [Persona dari PRD yang relevan] |
| **Scope** | ✅ In scope / ⚠️ Perlu diskusi |
| **Dependency** | [Fitur lain yang harus ada dulu, jika ada] |

---

## 3. User Flow

[Salin dan perjelas alur yang user berikan, lalu tambahkan detail teknis jika perlu]

**Happy Path:**
```
[Langkah 1] → [Langkah 2] → [Langkah 3] → ... → [Hasil akhir]
```

**Edge Cases:**
- [ ] Apa yang terjadi jika [kondisi error 1]?
- [ ] Apa yang terjadi jika [kondisi error 2]?
- [ ] Apa yang terjadi jika user tidak melanjutkan di tengah flow?

---

## 4. Functional Requirements

### Must Have (MVP)
- [ ] [Requirement 1]
- [ ] [Requirement 2]
- [ ] [Requirement 3]

### Should Have
- [ ] [Requirement nice-to-have 1]
- [ ] [Requirement nice-to-have 2]

### Won't Have (untuk versi ini)
- [ ] [Yang sengaja tidak dimasukkan dan alasannya]

---

## 5. Non-Functional Requirements

| Aspek | Requirement |
|-------|-------------|
| **Performance** | [Contoh: Response < 2 detik] |
| **Security** | [Contoh: Data ter-encrypt, auth required] |
| **Scalability** | [Contoh: Handle X concurrent users] |
| **Availability** | [Contoh: 99.9% uptime] |

---

## 6. UI/UX Notes

[Deskripsi singkat tampilan atau interaksi yang diharapkan. Bisa berupa deskripsi teks atau wireframe ASCII sederhana jika relevan]

**Touchpoints:**
- [ ] Screen / page yang terlibat
- [ ] Komponen baru yang perlu dibuat
- [ ] Komponen existing yang dimodifikasi

---

## 7. Technical Plan

### Existing Architecture
[Ringkas arsitektur dan pola repository yang relevan. Sertakan file konfigurasi atau direktori yang menjadi dasar kesimpulan.]

### Implementation Impact
| Layer | Perubahan | Lokasi/Komponen |
|-------|-----------|-----------------|
| Backend | [Perubahan Laravel jika relevan] | [Routes, Action/Controller, Form Request, Model, Policy, Job, dll.] |
| Frontend | [Perubahan Vue/Astro jika relevan] | [Page, component, composable, store, island, layout, dll.] |
| Database | [Migration/index/backfill jika relevan] | [Tabel atau model] |
| Infrastructure | [Queue, scheduler, cache, storage jika relevan] | [Komponen terkait] |

### Backend — Laravel (hapus jika tidak relevan)
- **Routing:** [web/API route yang berubah]
- **Business logic:** [Action/Service/Job mengikuti pola existing]
- **Validation:** [Form Request atau mekanisme existing]
- **Authorization:** [Policy/Gate/permission yang berlaku]
- **Async processing:** [Laravel Job/Event/Listener/Notification jika diperlukan]

### Frontend — Vue/Astro (hapus yang tidak relevan)
- **Rendering mode:** [Vue SPA/Inertia, Astro static, Astro SSR, atau islands]
- **UI state:** [Local state/store/composable mengikuti pola existing]
- **Data flow:** [Props, loader, API client, atau server endpoint]
- **Components/pages:** [Komponen dan halaman yang dibuat atau diubah]

### API / Endpoints (hanya jika diperlukan)
| Method | Endpoint | Auth | Validasi | Deskripsi |
|--------|----------|------|----------|-----------|
| [GET/POST/etc.] | `/api/...` | [Guard/ability] | [Request/schema] | [...] |

### Data Model (hanya jika diperlukan)
[Field, relasi, index, migration, dan kebutuhan backfill/rollback]

### Security & Privacy
- [ ] Authentication dan authorization sudah didefinisikan
- [ ] Input validation, CSRF/token handling, rate limit, dan output escaping dipertimbangkan sesuai arsitektur
- [ ] Data sensitif, upload, logging, dan mass assignment ditangani jika relevan

### Testing Strategy
| Level | Skenario | Tool Existing |
|-------|----------|---------------|
| Backend | [Feature/unit test dan authorization] | [Pest/PHPUnit yang terdeteksi] |
| Frontend | [Component/unit test] | [Vitest/tool yang terdeteksi] |
| End-to-end | [Critical user flow] | [Playwright/Cypress jika sudah tersedia] |

### Operational Impact
- **Migration/backfill:** [Diperlukan/tidak, risiko, dan rollback]
- **Queue/scheduler/cache:** [Perubahan worker, cron, atau invalidation]
- **Environment:** [Environment variable atau secret baru]
- **Deployment:** [Urutan deploy, build mode, downtime, atau langkah khusus]
- **Observability:** [Log, metric, atau alert yang diperlukan]

### Integration
- [ ] [Third-party/internal service; gunakan dependency existing jika memungkinkan]

---

## 8. Acceptance Criteria

Fitur dinyatakan selesai jika:
- [ ] [Kriteria 1 yang terukur]
- [ ] [Kriteria 2 yang terukur]
- [ ] [Kriteria 3 yang terukur]
- [ ] Authorization dan validation scenario utama teruji
- [ ] Test relevan sesuai tooling existing berhasil dijalankan
- [ ] Tidak ada regression pada fitur existing
- [ ] Dokumentasi diperbarui

---

## 9. Open Questions

> Pertanyaan yang masih perlu dijawab sebelum development dimulai

- [ ] [Pertanyaan 1]
- [ ] [Pertanyaan 2]

---

## 10. Timeline Estimasi

| Fase | Estimasi | Keterangan |
|------|----------|------------|
| Design & Spec | [X hari] | Finalisasi dokumen ini |
| Development | [X hari] | Implementasi |
| Testing | [X hari] | QA & UAT |
| Release | [tanggal target] | |

**Confidence:** [Low/Medium/High] — [jelaskan dependency atau ketidakpastian utama]

---

*Dokumen ini akan terus diperbarui selama proses development.*
```

---

## Instruksi Pengisian Template

Saat mengisi template, ikuti aturan ini:

1. **Jangan hasilkan placeholder kosong** — semua section harus diisi, bahkan dengan asumsi yang masuk akal. Tandai asumsi dengan `*[asumsi: ...]*`

2. **User Flow harus konkret** — ambil alur dari input user dan perjelas dengan langkah-langkah yang actionable

3. **Requirements harus spesifik** — hindari kalimat seperti "sistem harus cepat". Ganti dengan "response time < 2 detik"

4. **Alignment section wajib diisi** — ini yang membedakan dokumen ini dari spec biasa. Selalu hubungkan ke PRD

5. **Open Questions** — jika ada hal yang ambigu dari input user, masukkan ke sini, jangan diasumsikan sembarangan

6. **Ikuti repository, bukan asumsi framework** — gunakan versi, package, struktur folder, naming, dan pola implementasi yang terdeteksi. Jangan otomatis menambahkan repository pattern, service layer, Pinia, Inertia, queue driver, atau dependency lain yang tidak dipakai proyek

7. **Buat template kondisional** — hapus subsection Laravel, Vue, Astro, API, database, atau infrastructure yang tidak relevan. Jangan membuat endpoint atau tabel hanya untuk mengisi template

8. **Bedakan fakta dan usulan** — sertakan path file sebagai bukti untuk kondisi existing. Tandai keputusan yang belum pasti sebagai `*[usulan: ...]*` dan informasi yang tidak tersedia sebagai open question

9. **Testing mengikuti tooling existing** — jangan memperkenalkan Pest, Vitest, Playwright, atau tool lain jika repository belum menggunakannya, kecuali sebagai usulan yang disertai alasan

10. **Estimasi harus jujur** — gunakan rentang dan confidence level. Jangan memberi presisi palsu jika requirement, desain, atau dependency belum jelas

---

## Output

Setelah generate, sampaikan ke user:
1. **Path file** yang dibuat: `<root>/repo/fitur/FEATURE-[slug].md`
2. **Summary singkat** alignment dengan PRD (1-2 kalimat)
3. **Stack dan pola arsitektur** yang terdeteksi
4. **Daftar Open Questions** yang perlu dijawab user
5. Tawarkan untuk **merevisi section tertentu** jika ada yang kurang tepat

---

## Referensi

- Lihat `references/feature-examples.md` untuk contoh output dokumen fitur yang baik
- Lihat `references/alignment-checklist.md` untuk checklist alignment PRD yang lebih detail
