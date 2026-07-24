# PRD Alignment Checklist

Gunakan checklist ini saat memvalidasi fitur baru terhadap PRD.

---

## ✅ Checklist Alignment

### 1. Goal Alignment
- [ ] Fitur ini mendukung minimal satu product goal di PRD
- [ ] Fitur ini tidak bertentangan dengan non-goals yang disebutkan PRD
- [ ] Metric sukses fitur ini bisa dikaitkan ke KPI produk di PRD

### 2. User Alignment  
- [ ] Target user fitur ini adalah persona yang ada di PRD
- [ ] Fitur ini menyelesaikan pain point yang disebutkan di PRD (atau turunannya)
- [ ] Fitur ini tidak membuat pengalaman persona lain memburuk

### 3. Scope Alignment
- [ ] Fitur ini tidak masuk kategori "out of scope" / "won't do" di PRD
- [ ] Fitur ini tidak membutuhkan perubahan fundamental pada arsitektur yang sudah ditetapkan PRD
- [ ] Jika ada overlap dengan fitur existing, sudah ada penjelasan bagaimana bedanya

### 4. Technical Alignment
- [ ] Tech stack yang digunakan konsisten dengan PRD (atau ada justifikasi jelas kenapa berbeda)
- [ ] Versi framework dan dependency diverifikasi dari `composer.json`, lockfile, dan `package.json`
- [ ] Pola integrasi Laravel–Vue/Astro sesuai arsitektur repository (Blade, Inertia, SPA/API, static, atau SSR)
- [ ] Rancangan mengikuti konvensi routes, validation, authorization, state management, dan testing yang sudah ada
- [ ] Tidak menambahkan package atau abstraction baru tanpa kebutuhan dan justifikasi yang jelas
- [ ] Tidak ada asumsi infrastruktur baru yang belum dibahas di PRD
- [ ] Dependency ke fitur lain sudah teridentifikasi
- [ ] Dampak migration, queue, scheduler, cache, environment, deployment, dan rollback sudah diperiksa jika relevan

### 5. Timeline & Resource Alignment
- [ ] Estimasi effort realistis dengan resource yang disebutkan di PRD
- [ ] Tidak ada blocker yang akan menghambat roadmap produk lain di PRD

---

## 🚩 Red Flags — Tanda Fitur Mungkin Misaligned

Jika salah satu dari ini terjadi, sampaikan ke user sebelum lanjut:

| Red Flag | Artinya |
|----------|---------|
| PRD eksplisit menyebut ini "won't do" | Fitur di luar scope, butuh approval untuk lanjut |
| Target user berbeda dari semua persona di PRD | Mungkin fitur untuk produk lain |
| Membutuhkan infrastruktur yang tidak disebutkan PRD | Scope creep teknis |
| Berpotensi breaking existing fitur | Perlu impact analysis dulu |
| Estimasi jauh melebihi kapasitas tim di PRD | Perlu dipecah atau diprioritaskan ulang |

---

## Cara Sampaikan Misalignment ke User

Gunakan format ini saat ada red flag:

```
⚠️ Perhatian Alignment

Saya menemukan potensi misalignment antara fitur ini dan PRD:

**Isu:** [Deskripsi masalah]
**Referensi PRD:** [Section/kalimat di PRD yang relevan]
**Dampak:** [Apa yang bisa terjadi jika dilanjutkan]

**Opsi:**
1. Lanjutkan dengan catatan bahwa ini di luar scope PRD saat ini
2. Modifikasi fitur agar aligned dengan PRD
3. Update PRD dulu untuk mengakomodasi fitur ini

Mana yang kamu pilih?
```
