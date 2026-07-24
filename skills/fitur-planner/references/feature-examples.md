# Contoh Output Dokumen Fitur

File ini berisi contoh FEATURE.md yang sudah diisi dengan baik sebagai referensi kualitas output.

---

## Contoh: Fitur Notifikasi Email

> Konteks: PRD adalah aplikasi task management untuk tim kecil (3-10 orang).  
> Command user: `/fitur notifikasi email yang alurnya task selesai → sistem kirim email ke assignee → user lihat ringkasan harian`

---

```markdown
# Feature: Notifikasi Email

> Dokumen ini adalah rencana pengembangan fitur Notifikasi Email.  
> Dibuat berdasarkan PRD: PRD.md | Versi: 2024-01-15

---

## 1. Overview

**Nama Fitur:** Notifikasi Email  
**Status:** Draft  
**Priority:** Medium  
**Epic/Module:** Komunikasi & Kolaborasi

### Problem Statement
Anggota tim sering melewatkan update task karena hanya mengandalkan notifikasi in-app yang mudah terlewat saat tidak sedang membuka aplikasi. Tidak ada mekanisme untuk mendapatkan ringkasan aktivitas secara pasif.

### Proposed Solution
Sistem akan mengirim email notifikasi otomatis ketika task yang di-assign ke user mengalami perubahan status, ditambah ringkasan harian aktivitas tim setiap pagi.

---

## 2. Alignment dengan PRD

| Aspek | Keterangan |
|-------|------------|
| **Product Goal** | Mendukung goal "meningkatkan visibilitas progress tim" dari PRD section 2.1 |
| **Target User** | Project Member dan Project Lead — kedua persona yang ada di PRD |
| **Scope** | ✅ In scope — PRD menyebut "komunikasi tim" sebagai core feature area |
| **Dependency** | Fitur Task Management harus sudah live (sudah ada) |

---

## 2. User Flow

**Happy Path:**
```
User A menyelesaikan task → Status berubah ke "Done" → Sistem detect perubahan 
→ Cek setting notifikasi User B (assignee) → Jika email notif ON → 
Kirim email ke User B → User B terima email → Klik link di email → 
Diarahkan ke detail task
```

**Happy Path (Ringkasan Harian):**
```
Setiap pukul 08.00 → Sistem generate ringkasan per user → 
Kirim email jika ada aktivitas dalam 24 jam terakhir → 
User buka email → Lihat daftar task yang berubah
```

**Edge Cases:**
- [ ] Apa yang terjadi jika email user tidak valid? → Log error, skip kirim, tampilkan warning di profile
- [ ] Apa yang terjadi jika tidak ada aktivitas dalam 24 jam? → Tidak kirim email ringkasan harian
- [ ] Apa yang terjadi jika user unsubscribe? → Simpan preferensi, hentikan semua email notif

---

## 4. Functional Requirements

### Must Have (MVP)
- [ ] Email terkirim ketika task yang di-assign ke user berubah status
- [ ] Email berisi: nama task, status baru, siapa yang mengubah, link ke task
- [ ] User bisa toggle ON/OFF notifikasi email di Settings
- [ ] Ringkasan harian terkirim pukul 08.00 (timezone user)

### Should Have
- [ ] User bisa pilih jenis event apa saja yang memicu notifikasi (status change, komentar baru, deadline approaching)
- [ ] Unsubscribe link di setiap email (one-click)

### Won't Have (untuk versi ini)
- [ ] Digest mingguan — akan dipertimbangkan di v2
- [ ] Notifikasi SMS — out of scope PRD saat ini

---

## 5. Non-Functional Requirements

| Aspek | Requirement |
|-------|-------------|
| **Performance** | Email terkirim dalam < 30 detik setelah event terjadi |
| **Security** | Unsubscribe token harus signed dan expire dalam 30 hari |
| **Scalability** | Handle queue email hingga 10.000/jam (sesuai estimasi user di PRD) |
| **Availability** | Email queue harus persistent — tidak hilang jika server restart |

---

## 6. UI/UX Notes

**Touchpoints:**
- [ ] Settings page → tambah section "Email Notifications" dengan toggle
- [ ] Email template baru (HTML) untuk notifikasi event
- [ ] Email template baru (HTML) untuk ringkasan harian

**Settings UI (deskripsi):**
```
[Settings] > [Notifications] > [Email]

☑ Task status changes
☑ Comments on my tasks  
☐ Team activity digest (daily at 08:00)

[Save Preferences]
```

---

## 7. Technical Considerations

### API / Endpoints yang Dibutuhkan
| Method | Endpoint | Deskripsi |
|--------|----------|-----------|
| GET | `/api/users/me/notification-settings` | Ambil preferensi notif user |
| PUT | `/api/users/me/notification-settings` | Update preferensi notif user |
| POST | `/api/webhooks/task-updated` | Internal webhook trigger email |
| GET | `/api/unsubscribe?token=xxx` | Handle unsubscribe via link email |

### Data Model
```
NotificationSetting {
  user_id: UUID
  email_on_status_change: Boolean (default: true)
  email_on_comment: Boolean (default: false)
  email_daily_digest: Boolean (default: false)
  updated_at: Timestamp
}

EmailLog {
  id: UUID
  user_id: UUID
  type: ENUM (status_change, comment, daily_digest)
  task_id: UUID (nullable)
  sent_at: Timestamp
  status: ENUM (sent, failed, bounced)
}
```

### Integration
- [ ] SendGrid / AWS SES untuk email delivery
- [ ] Laravel Queue dan Job untuk async email processing, menggunakan queue driver yang sudah dikonfigurasi proyek

---

## 8. Acceptance Criteria

Fitur dinyatakan selesai jika:
- [ ] Email terkirim dalam < 30 detik setelah task diupdate
- [ ] User yang opt-out tidak menerima email sama sekali
- [ ] Ringkasan harian hanya terkirim jika ada aktivitas
- [ ] Semua email memiliki unsubscribe link yang berfungsi
- [ ] Tidak ada regression pada fitur task management existing
- [ ] Email template tampil dengan benar di Gmail, Outlook, Apple Mail

---

## 9. Open Questions

- [ ] Apakah kita pakai SendGrid atau SES? (tergantung budget — tanyakan ke stakeholder)
- [ ] Pukul 08.00 timezone mana untuk digest? Apakah per-user timezone atau fixed WIB?
- [ ] Apakah perlu approval dari user untuk verifikasi email sebelum notif aktif?

---

## 10. Timeline Estimasi

| Fase | Estimasi | Keterangan |
|------|----------|------------|
| Design & Spec | 2 hari | Finalisasi dokumen + review |
| Development | 5 hari | Backend + template email |
| Testing | 2 hari | QA + test di berbagai email client |
| Release | Sprint berikutnya | Setelah QA sign-off |
```

---

## Yang Membuat Dokumen Ini Baik

1. **Problem Statement konkret** — bukan "user butuh notifikasi" tapi menjelaskan *mengapa*
2. **Flow detail** — ada happy path + edge cases
3. **Alignment section terisi** — langsung merujuk ke section PRD yang relevan
4. **Requirements terukur** — "< 30 detik" bukan "cepat"
5. **Open Questions jelas** — bukan pertanyaan retoris, tapi keputusan nyata yang perlu dibuat
