# Laravel Wayfinder — Konvensi Vue 3 dan Inertia

Wayfinder menghasilkan fungsi TypeScript yang typed dari controller dan named
route Laravel. Gunakan fungsi generated sebagai sumber kebenaran URL dan HTTP
method pada frontend Laravel + Inertia.

## Setup

Periksa setup yang sudah ada sebelum menyarankan instalasi baru.

```bash
composer require laravel/wayfinder
npm install --save-dev @laravel/vite-plugin-wayfinder
```

```ts
// vite.config.ts
import { wayfinder } from '@laravel/vite-plugin-wayfinder'

export default defineConfig({
  plugins: [
    wayfinder(),
    // plugin lain
  ],
})
```

Plugin Vite membuat ulang definisi saat route/controller berubah dan ketika
build dijalankan. Untuk generate manual:

```bash
php artisan wayfinder:generate
```

Jangan mengedit file generated di direktori `actions/` atau `routes/` secara
manual. Perbaiki route/controller Laravel, lalu generate ulang.

---

## Import Generated Functions

Utamakan named import agar tree-shaking tetap efektif.

```ts
// Controller actions
import { show, store, update, destroy } from '@/actions/App/Http/Controllers/UserController'

// Named routes
import { index } from '@/routes/users'
```

Gunakan alias/path generated yang benar-benar tersedia di project. Jangan
menebak nama export; periksa file generated atau TypeScript autocomplete.

```ts
show(1)           // { url: '/users/1', method: 'get' }
show.url(1)       // '/users/1'
update({ user: 1 })
index({ query: { page: 2 } })
```

Wayfinder mendukung route model binding. Berikan parameter sesuai signature
generated, termasuk key seperti `slug` jika route memakai `{post:slug}`.

---

## Inertia Link

Berikan object Wayfinder langsung ke `href`. Jangan panggil `.url()` jika
komponen Inertia dapat menerima object tersebut.

```vue
<script setup lang="ts">
import { Link } from '@inertiajs/vue3'
import { index, show } from '@/routes/users'
</script>

<template>
  <Link :href="index()">Semua pengguna</Link>
  <Link :href="show(1)">Detail pengguna</Link>
</template>
```

```vue
<!-- ❌ URL hardcoded -->
<Link href="/users/1">Detail pengguna</Link>

<!-- ❌ Ziggy pada kode baru -->
<Link :href="route('users.show', 1)">Detail pengguna</Link>
```

---

## Inertia Router

Router Inertia menerima object Wayfinder dan dapat menyimpulkan URL serta
method-nya.

```ts
import { router } from '@inertiajs/vue3'
import { show, destroy } from '@/actions/App/Http/Controllers/UserController'

router.visit(show(1))
router.delete(destroy(1), {
  preserveScroll: true,
})
```

Gunakan callback Inertia (`onSuccess`, `onError`, `onFinish`) bila perlu.
Jangan memakai `window.location`, reload manual, atau `setTimeout` untuk
menyinkronkan hasil request.

---

## Inertia useForm dan Form

Berikan object Wayfinder langsung ke `form.submit`:

```vue
<script setup lang="ts">
import { useForm } from '@inertiajs/vue3'
import { store, update } from '@/actions/App/Http/Controllers/UserController'

const props = defineProps<{ userId?: number }>()

const form = useForm({
  name: '',
  email: '',
})

function submit() {
  form.submit(props.userId ? update(props.userId) : store())
}
</script>
```

Dengan komponen `Form` Inertia:

```vue
<script setup lang="ts">
import { Form } from '@inertiajs/vue3'
import { store } from '@/actions/App/Http/Controllers/UserController'
</script>

<template>
  <Form :action="store()">
    <input name="name" />
    <button type="submit">Simpan</button>
  </Form>
</template>
```

---

## URL String untuk API Non-Inertia

Gunakan `.url()` hanya jika consumer mensyaratkan string, misalnya `fetch`,
`axios`, atribut HTML native, atau integrasi library pihak ketiga.

```ts
import { show } from '@/routes/users'

const response = await fetch(show.url(1))
```

Pastikan HTTP method tetap mengikuti definisi generated. Untuk request mutasi
dalam aplikasi Inertia, utamakan `router`, `useForm`, atau `Form` daripada
merakit request manual.

---

## Query Parameters

Gunakan option `query` agar query string tetap typed dan tidak dirakit manual.

```ts
import { index } from '@/routes/users'

index({
  query: {
    page: 2,
    search: 'Budi',
  },
})
```

Gunakan `mergeQuery` bila parameter baru harus digabung dengan query URL saat
ini sesuai kontrak generated project.

---

## Checklist Review

- Project Laravel + Inertia menggunakan fungsi generated Wayfinder
- Import berasal dari `@/actions/...` atau `@/routes/...` yang tersedia
- Named import digunakan bila memungkinkan
- `<Link>`, `router`, `useForm`, dan `Form` menerima object Wayfinder langsung
- `.url()` hanya dipakai ketika consumer memerlukan string
- Tidak ada URL backend hardcoded atau template literal path
- Tidak ada helper Ziggy `route()` pada kode baru
- Parameter route sesuai signature generated dan route model binding
- Query parameter memakai `query`/`mergeQuery`, bukan concatenation string
- File generated tidak diedit manual
- Wayfinder tidak dipaksakan pada project Vue yang bukan frontend Laravel
