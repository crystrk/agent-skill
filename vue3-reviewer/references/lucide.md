# Lucide Vue — Konvensi `@lucide/vue` untuk Vue 3

## Install

```bash
npm install @lucide/vue
```

---

## Tujuan

Dokumen ini menjadi referensi penggunaan icon pada project Vue 3.

Project menggunakan **@lucide/vue** sebagai library icon utama.

---

# Konvensi

## 1. Gunakan `@lucide/vue`

```ts
import { Home, User, Search } from '@lucide/vue'
```

Jangan gunakan library ikon lain untuk ikon UI umum. Logo brand dan ilustrasi
custom tetap gunakan asset yang sesuai; jangan memaksakannya menjadi ikon
Lucide.

---

## 2. Gunakan Named Import

✅

```ts
import { Home, Plus, Trash2 } from '@lucide/vue'
```

Hindari import yang tidak diperlukan agar tree-shaking bekerja optimal.

---

## 3. Gunakan Langsung di Template

```vue
<script setup lang="ts">
import { Home } from '@lucide/vue'
</script>

<template>
  <Home class="size-4" />
</template>
```

---

## 4. Dynamic Component Hanya Jika Diperlukan

```vue
<script setup lang="ts">
import { Home, Users } from '@lucide/vue'

const menus = [
  { title: 'Home', icon: Home },
  { title: 'Users', icon: Users },
]
</script>

<template>
  <div v-for="item in menus" :key="item.title">
    <component :is="item.icon" class="size-4" />
    {{ item.title }}
  </div>
</template>
```

---

## 5. Konsistensi Ukuran

Gunakan satu konvensi.

Contoh dengan Tailwind:

```vue
<Home class="size-4" />
<Home class="size-5" />
```

atau menggunakan props:

```vue
<Home :size="18" />
```

Jangan mencampur gaya tanpa alasan.

---

## 6. Stroke Width

Gunakan stroke yang konsisten.

```vue
<Home :stroke-width="2" />
```

---

## 7. Accessibility

Ikon dekoratif:

```vue
<Home class="size-4" aria-hidden="true" />
```

Ikon tanpa teks:

```vue
<button aria-label="Delete">
  <Trash2 class="size-4" aria-hidden="true" />
</button>
```

---

## 8. Shadcn Vue

Ikuti pola yang digunakan shadcn-vue.

```vue
<Button>
  <Plus class="size-4" />
  Tambah
</Button>
```

---

## 9. Registry Icon (Opsional)

Secara default gunakan named import langsung.

Jika project memiliki design system besar atau icon yang sama digunakan di banyak file, boleh membuat barrel file seperti:

```ts
// constants/icons.ts

export {
  Home,
  User,
  Search,
  Plus,
  Trash2,
} from '@lucide/vue'
```

Import:

```ts
import { Home } from "@/constants/icons"
```

Jangan menjadikan registry sebagai aturan wajib.

---

# Checklist Review

Saat mereview kode pastikan:

- menggunakan `@lucide/vue`
- memakai named import
- tidak ada icon yang tidak digunakan
- ukuran icon konsisten
- stroke width konsisten
- accessibility diperhatikan
- dynamic icon hanya bila diperlukan
- konsisten dengan shadcn-vue

---

# Yang Dilarang

- Library ikon lain untuk ikon UI umum
- Font Awesome
- Bootstrap Icons
- Remix Icons
- SVG inline untuk icon umum
- Emoji sebagai icon UI
- Import icon yang tidak digunakan
- Dynamic component tanpa kebutuhan nyata
