# API Deprecated Vue 3 — Daftar Lengkap & Pengganti

## Lifecycle Hooks

| Deprecated (Options API) | Pengganti (Composition API) |
|---|---|
| `beforeCreate` | Kode langsung di `<script setup>` |
| `created` | Kode langsung di `<script setup>` |
| `beforeMount` | `onBeforeMount()` |
| `mounted` | `onMounted()` |
| `beforeUpdate` | `onBeforeUpdate()` |
| `updated` | `onUpdated()` |
| `beforeDestroy` | `onBeforeUnmount()` |
| `destroyed` | `onUnmounted()` |
| `errorCaptured` | `onErrorCaptured()` |
| `activated` | `onActivated()` |
| `deactivated` | `onDeactivated()` |

```typescript
// ❌ Options API
export default {
  mounted() { this.init() },
  beforeDestroy() { this.cleanup() }
}

// ✅ Composition API
onMounted(() => init())
onBeforeUnmount(() => cleanup())
```

---

## Instance Properties & Methods

### `$emit`
```typescript
// ❌
this.$emit('click', payload)

// ✅
const emit = defineEmits<{ click: [payload: Payload] }>()
emit('click', payload)
```

### `$refs`
```typescript
// ❌
this.$refs.myInput.focus()

// ✅
const myInput = ref<HTMLInputElement | null>(null)
myInput.value?.focus()
```

### `$parent` / `$children`
```typescript
// ❌ — Sangat tidak direkomendasikan
this.$parent.someMethod()
this.$children[0].data

// ✅ — Gunakan provide/inject
provide('context', { someMethod, data })
// Di child:
const { someMethod, data } = inject('context')
```

### `$listeners`
```typescript
// ❌ — $listeners sudah dihapus di Vue 3
// (sudah merge ke $attrs)

// ✅ — Gunakan $attrs atau defineEmits
// Untuk forward semua event:
<ChildComp v-bind="$attrs" />
```

### `$set` / `Vue.set`
```typescript
// ❌ — Tidak perlu di Vue 3
this.$set(this.obj, 'newKey', value)
Vue.set(array, index, value)

// ✅ — Mutasi langsung sudah reactive di Vue 3
obj.newKey = value
array[index] = value
array.push(newItem)
```

### `$delete` / `Vue.delete`
```typescript
// ❌
Vue.delete(this.obj, 'key')

// ✅
delete obj.key
```

### `$on` / `$off` / `$once` (Event Bus)
```typescript
// ❌ — Dihapus di Vue 3
const bus = new Vue()
bus.$on('event', handler)
bus.$emit('event', data)

// ✅ — Gunakan mitt atau tiny-emitter
import mitt from 'mitt'
export const emitter = mitt()

// Kirim
emitter.emit('event', data)
// Terima
emitter.on('event', handler)
// Bersihkan
onUnmounted(() => emitter.off('event', handler))
```

---

## Directives & Template

### `v-model.sync`
```html
<!-- ❌ Vue 2 -->
<Child :title.sync="pageTitle" />

<!-- ✅ Vue 3 -->
<Child v-model:title="pageTitle" />
```

### `Filters`
```html
<!-- ❌ Vue 2 Filters -->
{{ price | currency }}
{{ date | formatDate('DD/MM/YYYY') }}

<!-- ✅ Vue 3 — gunakan computed atau method -->
```
```typescript
const formattedPrice = computed(() => formatCurrency(price.value))
function formatDate(date: string) { return dayjs(date).format('DD/MM/YYYY') }
```

### `v-bind.prop` modifier
```html
<!-- ❌ Jarang perlu, hindari -->
<div v-bind.prop="value" />

<!-- ✅ Gunakan :prop langsung -->
<div :some-prop="value" />
```

---

## Options API → Composition API

### `data()`
```typescript
// ❌
export default {
  data() {
    return { count: 0, name: '' }
  }
}

// ✅
const count = ref(0)
const name = ref('')
```

### `computed`
```typescript
// ❌
export default {
  computed: {
    fullName() { return `${this.first} ${this.last}` }
  }
}

// ✅
const fullName = computed(() => `${first.value} ${last.value}`)
```

### `methods`
```typescript
// ❌
export default {
  methods: {
    handleClick() { this.count++ }
  }
}

// ✅
function handleClick() { count.value++ }
```

### `watch`
```typescript
// ❌
export default {
  watch: {
    count(newVal, oldVal) { console.log(newVal) }
  }
}

// ✅
watch(count, (newVal, oldVal) => {
  console.log(newVal)
})

// Atau watchEffect untuk efek otomatis
watchEffect(() => {
  console.log(count.value)
})
```

### `mixins`
```typescript
// ❌ — Hindari mixins di Vue 3
export default {
  mixins: [UserMixin, AuthMixin]
}

// ✅ — Gunakan composables
const { user, updateUser } = useUser()
const { isAuthenticated, login } = useAuth()
```

---

## Routing

### Standalone Vue — Vue Router 4

Gunakan pola ini hanya jika project memang memakai Vue Router dan bukan
frontend Laravel + Inertia.

```typescript
// ❌ Vue Router 3
import Vue from 'vue'
import Router from 'vue-router'
Vue.use(Router)
export default new Router({ ... })

// ✅ Vue Router 4
import { createRouter, createWebHistory } from 'vue-router'
const router = createRouter({
  history: createWebHistory(),
  routes: [...]
})

// Di komponen:
// ❌
this.$router.push('/home')
this.$route.params.id

// ✅
import { useRouter, useRoute } from 'vue-router'
const router = useRouter()
const route = useRoute()
router.push('/home')
route.params.id
```

### Laravel + Inertia — Wayfinder

Jangan menambahkan Vue Router untuk route yang dimiliki Laravel. Gunakan router
Inertia dengan fungsi typed dari Wayfinder:

```typescript
import { router } from '@inertiajs/vue3'
import { show } from '@/actions/App/Http/Controllers/UserController'

router.visit(show(1))
```

Baca `references/wayfinder.md` untuk pola `<Link>`, form, query parameter, dan
larangan URL backend hardcoded.

---

## Pinia (pengganti Vuex)

```typescript
// ❌ Vuex
import { useStore } from 'vuex'
const store = useStore()
store.commit('SET_USER', user)
store.dispatch('fetchUser')
store.state.user

// ✅ Pinia
import { useUserStore } from '@/stores/user'
const userStore = useUserStore()
userStore.setUser(user)       // action langsung
await userStore.fetchUser()
userStore.user                // state langsung
```

---

## defineComponent (tanpa script setup)

```typescript
// ⚠️ Boleh tapi tidak direkomendasikan untuk komponen baru
import { defineComponent } from 'vue'
export default defineComponent({
  setup() { ... }
})

// ✅ Lebih ringkas dengan script setup
// <script setup lang="ts">
// Langsung tulis logika di sini
// </script>
```

---

## Teleport (pengganti Portal)

```html
<!-- ✅ Vue 3 built-in -->
<Teleport to="body">
  <Modal v-if="showModal" />
</Teleport>
```

---

## KeepAlive dengan defineComponent

```html
<!-- ✅ Gunakan nama komponen atau ref -->
<KeepAlive :include="['UserProfile', 'Dashboard']">
  <component :is="currentComponent" />
</KeepAlive>
```
