---
name: vue3-reviewer
description: "Mereview kode Vue 3 untuk konvensi, API deprecated, shadcn-vue, ikon @lucide/vue, Laravel Wayfinder, dan pemecahan komponen. Digunakan saat user meminta review, audit, atau refactor Vue 3, khususnya pada aplikasi Laravel dan Inertia."
---

# Vue 3 Code Reviewer

Skill ini memandu review kode Vue 3 secara sistematis dalam **6 dimensi**:
konvensi, deprecated API, shadcn-vue, `@lucide/vue`, Laravel Wayfinder, dan
pemecahan komponen.

---

## Cara Menggunakan Skill Ini

1. Terima kode Vue dari user (satu file atau beberapa file)
2. Jalankan keenam dimensi review secara berurutan
3. Sajikan temuan dalam format terstruktur (lihat bagian Output)
4. Sertakan kode hasil refactor yang siap pakai

Untuk aturan detail tiap dimensi, baca referensi berikut sesuai kebutuhan:
- `references/conventions.md` — Konvensi Vue 3 & Composition API
- `references/deprecated.md` — API deprecated & penggantinya
- `references/shadcn-vue.md` — Komponen shadcn-vue yang tersedia
- `references/lucide.md` — Konvensi ikon `@lucide/vue`
- `references/wayfinder.md` — Laravel Wayfinder untuk route dan action typed
- `references/component-split.md` — Kapan & bagaimana memecah komponen

---

## Alur Review (Ringkasan Cepat)

### Dimensi 1 — Konvensi Vue 3

Periksa hal-hal berikut:

**Script Setup & Composition API**
- Wajib gunakan `<script setup lang="ts">` (bukan Options API kecuali ada alasan kuat)
- `defineProps` dan `defineEmits` harus menggunakan type-based syntax
- Urutan blok: `<script setup>` → `<template>` → `<style scoped>`
- Gunakan `const` untuk semua variabel reaktif kecuali perlu reassign

**Penamaan**
- Komponen: `PascalCase` di `<script>`, `PascalCase` atau `kebab-case` di template
- Props: `camelCase` di definisi, `kebab-case` di template
- Emits: `kebab-case` (misal: `update:modelValue`)
- Composables: prefix `use` + `camelCase` (misal: `useUserProfile`)
- File komponen: `PascalCase.vue`

**Reaktivitas**
- Gunakan `ref()` untuk primitif, `reactive()` untuk objek kompleks
- Jangan destructure `reactive()` tanpa `toRefs()`
- `computed()` harus pure (tanpa side effect)
- Hindari `watch` yang tidak perlu — pertimbangkan `watchEffect` atau `computed`

**Template**
- Selalu beri `:key` unik pada `v-for` (bukan index jika data bisa berubah urutan)
- Jangan gunakan `v-if` bersamaan dengan `v-for` pada elemen yang sama
- Gunakan `v-bind` shorthand (`:`) dan `v-on` shorthand (`@`)
- Extract logic kompleks ke `computed` atau method, bukan inline di template

**TypeScript**
- Definisikan tipe eksplisit untuk props, emits, dan return value composable
- Gunakan `interface` atau `type` untuk mendefinisikan shape objek
- Hindari `any` — gunakan `unknown` atau tipe yang tepat

---

### Dimensi 2 — API Deprecated

Deteksi dan ganti API yang sudah tidak direkomendasikan. Baca `references/deprecated.md` untuk daftar lengkap. Poin kritis:

| Deprecated | Pengganti |
|---|---|
| `this.$emit` / Options API | `defineEmits()` + `emit()` |
| `this.$refs` | `const el = ref<HTMLElement>()` |
| `Vue.set` / `Vue.delete` | Mutasi langsung (reactivity sudah otomatis di Vue 3) |
| `$listeners` | Sudah merge ke `$attrs` di Vue 3 |
| `v-model.sync` | `v-model:propName` |
| `destroyed` / `beforeDestroy` | `onUnmounted` / `onBeforeUnmount` |
| `created` / `beforeCreate` | Kode langsung di `<script setup>` |
| `Filters` (`\| filterName`) | Method atau computed |
| `$children` | Template refs atau provide/inject |
| `defineComponent` tanpa `<script setup>` | Migrasi ke `<script setup>` |

---

### Dimensi 3 — Komponen shadcn-vue

Baca `references/shadcn-vue.md` untuk daftar komponen lengkap.

Identifikasi elemen UI native HTML atau custom yang bisa diganti dengan komponen shadcn-vue. Prioritas utama:

- `<button>` biasa → `<Button>` dari shadcn-vue
- `<input>` / `<textarea>` → `<Input>` / `<Textarea>`
- Dialog/modal custom → `<Dialog>`
- Dropdown custom → `<DropdownMenu>` atau `<Select>`
- Toast/notifikasi custom → `<Sonner>` atau `useToast()`
- Tabel data → `<Table>` atau DataTable dengan TanStack
- Form dengan validasi → `<Form>` dengan VeeValidate
- Loading/skeleton → `<Skeleton>`
- Badge status → `<Badge>`
- Card wrapper → `<Card>`

**Cara menyarankan:**
```
❌ Ditemukan: <button class="px-4 py-2 bg-blue-500 text-white rounded">
✅ Ganti dengan: <Button variant="default">...</Button>
   Import: import { Button } from '@/components/ui/button'
```

---

### Dimensi 4 — Lucide Icons (WAJIB)

> ⚠️ Semua ikon UI umum pada project harus menggunakan `@lucide/vue`.

Baca `references/lucide.md` untuk aturan lengkap.

**Aturan utama:**

- Gunakan named import dari `@lucide/vue` agar tree-shaking bekerja
- Gunakan komponen ikon langsung di template
- Gunakan penamaan import yang konsisten; default: `Home`, `User`, `Trash2`
- Tetapkan ukuran secara konsisten, misalnya `class="size-4"`
- Pastikan tombol icon-only memiliki accessible name
- Jangan gunakan Heroicons, Font Awesome, Bootstrap Icons, Tabler Icons,
  SVG inline untuk ikon umum, atau emoji sebagai ikon UI

Contoh:

```vue
<script setup lang="ts">
import { Trash2 } from '@lucide/vue'
</script>

<template>
  <Button variant="destructive" size="icon" aria-label="Hapus pengguna">
    <Trash2 class="size-4" aria-hidden="true" />
  </Button>
</template>
```

Registry/barrel ikon bersifat **opsional**, bukan wajib. Gunakan hanya jika sudah
menjadi konvensi project atau benar-benar menyederhanakan ikon dinamis yang
dipakai berulang. Jangan mengimpor seluruh namespace ikon.

---

### Dimensi 5 — Laravel Wayfinder (WAJIB untuk Laravel + Inertia)

Baca `references/wayfinder.md` untuk setup dan pola lengkap.

Pada project Laravel + Inertia, semua URL backend harus berasal dari fungsi
Wayfinder yang generated dan typed:

- Gunakan named import dari `@/actions/...` atau `@/routes/...`
- Berikan hasil fungsi Wayfinder langsung ke `<Link>`, `router`, atau `useForm`
- Gunakan `.url()` hanya saat API benar-benar meminta string URL
- Jangan hardcode URL backend, menyusun path dengan template literal, atau
  memakai helper `route()` dari Ziggy untuk kode baru
- Jangan mengedit file generated di `actions/` atau `routes/` secara manual

```vue
<script setup lang="ts">
import { Link, useForm } from '@inertiajs/vue3'
import { show, store } from '@/actions/App/Http/Controllers/UserController'

const form = useForm({ name: '' })

function submit() {
  form.submit(store())
}
</script>

<template>
  <Link :href="show(1)">Lihat pengguna</Link>
</template>
```

Jika project Vue tidak terhubung ke Laravel, tandai dimensi ini sebagai tidak
berlaku dan jangan memaksakan Wayfinder.

---

### Dimensi 6 — Pemecahan Komponen

Baca `references/component-split.md` untuk panduan lengkap dan contoh.

**Aturan lokasi untuk Laravel + Inertia:**
- `resources/js/pages/` hanya berisi entry page yang di-resolve oleh Inertia
- Jangan membuat komponen hasil split, composable, atau helper di dalam
  `pages/`, termasuk subfolder `pages/**/components`
- Tempatkan komponen reusable/feature di `resources/js/components/`, komponen
  shadcn-vue di `resources/js/components/ui/`, layout di folder komponen/layout
  yang sudah dipakai project, dan composable di `resources/js/composables/`
- Pertahankan casing folder dan alias import project yang sudah ada; jangan
  memindahkan file generated shadcn-vue secara manual

**Sinyal bahwa komponen perlu dipecah:**
- File `.vue` > 200 baris
- Template memiliki lebih dari 2 level nesting yang kompleks
- Ada blok `v-for` yang merender UI kompleks (pecah jadi item component)
- Logika yang sama diulang di beberapa bagian template
- `<script setup>` > 80 baris dengan banyak concern berbeda
- Ada section UI yang jelas (header, sidebar, list, form, card) dalam satu file

**Strategi pemecahan:**
```
resources/js/
├── pages/
│   └── Users/Index.vue               ← entry page/orchestrator Inertia
├── components/
│   └── users/
│       ├── UserList.vue               ← list
│       ├── UserCard.vue               ← item/card
│       └── UserForm.vue               ← form
└── composables/
    └── useUserData.ts                 ← logika data
```

---

## Format Output Review

Gunakan format ini untuk menyajikan hasil review:

```
## 🔍 Hasil Review Vue 3

### ✅ Sudah Baik
- [hal-hal yang sudah benar]

---

### 🔴 Kritis (Wajib Diperbaiki)
**[Nama Masalah]**
- Lokasi: baris X / section Y
- Masalah: [penjelasan singkat]
- Solusi:
  [kode perbaikan]

---

### 🟡 Perlu Diperhatikan
[temuan medium priority]

---

### 🔵 Saran Peningkatan
[opsional tapi direkomendasikan]

---

### 📦 Saran Pemecahan Komponen
[jika file perlu dipecah]

---

### 🧭 Laravel Wayfinder
[kepatuhan route/action typed; tulis "Tidak berlaku" untuk project non-Laravel]

---

### ✨ Kode Hasil Refactor
[kode lengkap yang sudah diperbaiki]
```

---

## Catatan Penting

- Selalu berikan **kode konkret** sebagai pengganti, bukan hanya saran abstrak
- Jika menemukan library ikon lain, SVG inline untuk ikon umum, atau emoji
  sebagai ikon UI, tunjukkan penggantinya dari `@lucide/vue`
- Jika menemukan URL Laravel hardcoded atau helper route selain Wayfinder,
  tunjukkan generated import dan pemanggilan Wayfinder yang tepat
- Untuk pemecahan komponen, sertakan **struktur folder** yang disarankan
- Pada Laravel + Inertia, jangan pernah menyarankan hasil split di bawah
  `resources/js/pages/`; pertahankan `pages/` sebagai kumpulan entry page
- Prioritaskan masalah Kritis → Perlu Diperhatikan → Saran
- Jika kode sudah baik di suatu dimensi, nyatakan dengan jelas agar user tahu

### Konsistensi dengan Project

Sebelum memberikan saran:

- jangan mengganti pola yang sudah menjadi standar project
- gunakan `@lucide/vue` sebagai standar ikon dan Wayfinder sebagai standar route
  Laravel; untuk area lain, jangan menyarankan library baru bila project sudah
  memiliki solusi
- prioritaskan konsistensi dibanding preferensi pribadi
- bila ada skill lain yang mengatur konvensi (misalnya `inertia-vue-development` atau `shadcn-vue`), ikuti konvensi pada skill tersebut
