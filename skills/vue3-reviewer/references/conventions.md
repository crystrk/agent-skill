# Konvensi Vue 3 — Referensi Lengkap

## 1. Struktur File SFC

Urutan blok yang benar:
```vue
<script setup lang="ts">
// 1. Import eksternal (vue, library)
// 2. Import internal (composables, utils, types)
// 3. Import komponen
// 4. Props & Emits
// 5. State (ref, reactive)
// 6. Computed
// 7. Methods
// 8. Lifecycle hooks
// 9. Watch
</script>

<template>
  <!-- template -->
</template>

<style scoped>
/* style */
</style>
```

---

## 2. Props — Type-Based Syntax

```typescript
// ✅ BENAR — type-based (Vue 3.3+)
const props = defineProps<{
  title: string
  count?: number
  user: User
  items: string[]
}>()

// Dengan default value (Vue 3.3+)
const props = withDefaults(defineProps<{
  title: string
  count?: number
}>(), {
  count: 0
})

// ❌ SALAH — runtime syntax (boleh tapi tidak direkomendasikan di TS project)
const props = defineProps({
  title: { type: String, required: true },
  count: { type: Number, default: 0 }
})
```

---

## 3. Emits — Type-Based Syntax

```typescript
// ✅ BENAR
const emit = defineEmits<{
  'update:modelValue': [value: string]
  'item-click': [id: number, item: Item]
  'close': []
}>()

// Penggunaan
emit('update:modelValue', newValue)
emit('item-click', item.id, item)
```

---

## 4. Reaktivitas

```typescript
// Primitif → ref()
const count = ref(0)
const name = ref<string>('')
const isLoading = ref(false)

// Objek kompleks → reactive()
const form = reactive({
  email: '',
  password: ''
})

// JANGAN destructure reactive() langsung
const { email } = form           // ❌ kehilangan reaktivitas
const { email } = toRefs(form)   // ✅ reaktivitas terjaga

// computed — harus pure
const fullName = computed(() => `${firstName.value} ${lastName.value}`)

// computed writable
const modelValue = computed({
  get: () => props.value,
  set: (val) => emit('update:modelValue', val)
})
```

---

## 5. Template Refs

```typescript
// ✅ BENAR
const inputEl = ref<HTMLInputElement | null>(null)
const myComp = ref<InstanceType<typeof MyComponent> | null>(null)

onMounted(() => {
  inputEl.value?.focus()
})
```

```html
<input ref="inputEl" />
<MyComponent ref="myComp" />
```

---

## 6. Composables

```typescript
// useCounter.ts
import { ref, computed } from 'vue'

export function useCounter(initialValue = 0) {
  const count = ref(initialValue)
  const doubled = computed(() => count.value * 2)

  function increment() {
    count.value++
  }

  function decrement() {
    count.value--
  }

  return { count, doubled, increment, decrement }
}
```

Aturan composable:
- Nama selalu prefix `use`
- Kembalikan objek (bukan array, kecuali seperti useState React)
- Dapat dipanggil hanya di `<script setup>` atau composable lain
- Jangan panggil composable secara conditional

---

## 7. Provide / Inject

```typescript
// Parent
import { provide, ref } from 'vue'
import type { InjectionKey } from 'vue'

interface ThemeContext {
  theme: Ref<string>
  setTheme: (t: string) => void
}

// Buat key yang typed
export const ThemeKey: InjectionKey<ThemeContext> = Symbol('theme')

const theme = ref('light')
provide(ThemeKey, {
  theme,
  setTheme: (t) => { theme.value = t }
})

// Child
import { inject } from 'vue'
import { ThemeKey } from '@/composables/useTheme'

const { theme, setTheme } = inject(ThemeKey)!
```

---

## 8. v-model pada Custom Komponen

```vue
<!-- Parent -->
<MyInput v-model="name" />
<MyInput v-model:title="pageTitle" v-model:content="pageContent" />

<!-- MyInput.vue -->
<script setup lang="ts">
const props = defineProps<{
  modelValue: string
}>()
const emit = defineEmits<{
  'update:modelValue': [value: string]
}>()
</script>

<template>
  <input
    :value="props.modelValue"
    @input="emit('update:modelValue', ($event.target as HTMLInputElement).value)"
  />
</template>
```

---

## 9. Async & Suspense

```vue
<script setup lang="ts">
// Top-level await diizinkan di <script setup>
const data = await fetchData()
</script>

<!-- Parent perlu Suspense -->
<Suspense>
  <AsyncComponent />
  <template #fallback>
    <Skeleton />
  </template>
</Suspense>
```

---

## 10. defineOptions (Vue 3.3+)

```typescript
// Untuk set nama komponen atau inheritAttrs di <script setup>
defineOptions({
  name: 'MyComponent',
  inheritAttrs: false
})
```

---

## 11. Penamaan Event Handler

```html
<!-- ✅ Deskriptif -->
@click="handleSubmit"
@change="handleStatusChange"
@keydown.enter="handleEnterKey"

<!-- ❌ Terlalu generic -->
@click="click"
@change="change"
```

---

## 12. Slot

```vue
<!-- Komponen dengan slot typed (defineSlots — Vue 3.3+) -->
<script setup lang="ts">
defineSlots<{
  default(props: { item: Item }): any
  header(): any
}>()
</script>

<template>
  <div>
    <slot name="header" />
    <slot v-for="item in items" :item="item" />
  </div>
</template>
```
