# Panduan Pemecahan Komponen Vue 3

## Kapan Harus Memecah Komponen?

### Sinyal Kuat (Hampir Pasti Perlu Dipecah)
- File `.vue` lebih dari **200 baris** total
- `<script setup>` lebih dari **80 baris** dengan banyak concern berbeda
- Template memiliki **nested `v-for`** yang merender UI kompleks
- Ada **logika yang duplikat** di beberapa bagian template
- Komponen melakukan lebih dari **satu hal utama** (fetch data + form + tabel + modal dalam satu file)

### Sinyal Sedang (Pertimbangkan Pemecahan)
- Ada section UI yang jelas batasnya (header, sidebar, list, form, card, modal)
- Template memiliki lebih dari **3 level indentasi** yang konsisten
- Ada blok yang bisa **digunakan ulang** di halaman/komponen lain
- `computed` dan `watch` semakin banyak dan tidak berkaitan satu sama lain

### Sinyal Lemah (Opsional)
- File masih di bawah 150 baris tapi sudah punya struktur yang bisa dipecah
- Ingin membuat unit test lebih mudah

---

## Tentukan Lokasi Sebelum Memecah

Periksa struktur dan alias import project sebelum mengusulkan file baru. Jangan
mengasumsikan semua project memakai `src/` atau menaruh komponen di dekat page.

### Laravel + Vue + Inertia

Pada struktur Laravel + Inertia, perlakukan `resources/js/pages/` sebagai
**boundary entry page**. Resolver Inertia memuat file dari folder tersebut
berdasarkan nama page dari backend. Karena itu:

- simpan hanya entry page Inertia di `resources/js/pages/`
- jangan taruh hasil split di `pages/**/components`, `pages/**/composables`, atau
  folder lain di bawah `pages/`
- simpan komponen domain/feature di `resources/js/components/<feature>/`
- pertahankan komponen generated shadcn-vue di `resources/js/components/ui/`
- simpan layout global di lokasi komponen layout yang sudah digunakan project,
  misalnya `resources/js/components/Layouts/`
- simpan composable dan tipe lintas komponen masing-masing di
  `resources/js/composables/` dan `resources/js/types/`
- gunakan alias import project, biasanya `@/components/...`

Aturan ini mencegah komponen internal tercampur dengan namespace page yang
dapat di-resolve oleh Inertia. Folder feature di bawah `components/` boleh
mencerminkan nama page, tetapi tetap berada di luar `pages/`.

```text
resources/js/
├── components/
│   ├── ui/                         ← generated shadcn-vue
│   ├── Layouts/                    ← ikuti casing project
│   │   └── AppLayout.vue
│   └── users/
│       ├── UserTable.vue
│       ├── UserFormDialog.vue
│       └── UserDeleteDialog.vue
├── composables/
│   └── useUsers.ts
├── pages/
│   └── Users/
│       └── Index.vue               ← entry page Inertia saja
├── app.ts
└── types/
```

Untuk project Vue non-Laravel, ikuti struktur project yang sudah ada. Jangan
memaksakan struktur Laravel atau membuat folder baru tanpa kebutuhan.

---

## Strategi Pemecahan

### 1. Pecah Berdasarkan Item List

```
// SEBELUM — UserList.vue (200+ baris)
<template>
  <div v-for="user in users" :key="user.id">
    <div class="avatar">...</div>
    <div class="info">
      <h3>{{ user.name }}</h3>
      <p>{{ user.email }}</p>
      <Badge>{{ user.role }}</Badge>
    </div>
    <div class="actions">
      <button @click="edit(user)">...</button>
      <button @click="delete(user)">...</button>
    </div>
  </div>
</template>

// SESUDAH — UserList.vue (ringkas)
<template>
  <UserCard
    v-for="user in users"
    :key="user.id"
    :user="user"
    @edit="handleEdit"
    @delete="handleDelete"
  />
</template>

// UserCard.vue (komponen baru)
```

### 2. Pecah Berdasarkan Section/Area UI

```
// SEBELUM — DashboardPage.vue (300+ baris)
<template>
  <header>...</header>
  <aside>...</aside>
  <main>
    <section class="stats">...</section>
    <section class="chart">...</section>
    <section class="recent-activity">...</section>
  </main>
</template>

// SESUDAH
<template>
  <DashboardHeader />
  <DashboardSidebar />
  <main>
    <StatsGrid :stats="stats" />
    <ActivityChart :data="chartData" />
    <RecentActivityList :items="activities" />
  </main>
</template>
```

### 3. Pecah Logika ke Composable

```typescript
// SEBELUM — semua di <script setup>
const users = ref([])
const isLoading = ref(false)
const error = ref(null)
const searchQuery = ref('')
const currentPage = ref(1)
const totalPages = ref(0)

async function fetchUsers() { ... }
async function deleteUser(id) { ... }
async function updateUser(id, data) { ... }

const filteredUsers = computed(() => ...)
const paginatedUsers = computed(() => ...)

watch(searchQuery, () => { currentPage.value = 1; fetchUsers() })
watch(currentPage, fetchUsers)

onMounted(fetchUsers)

// SESUDAH — composable
// composables/useUsers.ts
export function useUsers() {
  const users = ref([])
  const isLoading = ref(false)
  const error = ref<Error | null>(null)

  async function fetchUsers(params?: UserParams) { ... }
  async function deleteUser(id: string) { ... }
  async function updateUser(id: string, data: Partial<User>) { ... }

  return { users, isLoading, error, fetchUsers, deleteUser, updateUser }
}

// composables/useUserSearch.ts
export function useUserSearch(users: Ref<User[]>) {
  const searchQuery = ref('')
  const currentPage = ref(1)
  const pageSize = ref(10)

  const filteredUsers = computed(() => ...)
  const paginatedUsers = computed(() => ...)
  const totalPages = computed(() => ...)

  watch(searchQuery, () => { currentPage.value = 1 })

  return { searchQuery, currentPage, pageSize, filteredUsers, paginatedUsers, totalPages }
}

// Di komponen — jauh lebih bersih
const { users, isLoading, fetchUsers, deleteUser } = useUsers()
const { searchQuery, paginatedUsers, totalPages } = useUserSearch(users)
onMounted(fetchUsers)
```

### 4. Pecah Form Kompleks

```
// SEBELUM — satu form besar
resources/js/pages/Users/Form.vue
// 400 baris — data pribadi + alamat + preferensi + foto

// SESUDAH
resources/js/
├── pages/Users/
│   └── Form.vue                    ← entry page/orchestrator
├── components/users/
│   ├── UserFormPersonal.vue        ← Nama, email, telepon
│   ├── UserFormAddress.vue         ← Alamat, kota, provinsi
│   ├── UserFormPreferences.vue     ← Notifikasi, bahasa, tema
│   └── UserFormAvatar.vue          ← Upload & crop foto
└── composables/
    └── useUserForm.ts              ← State & validasi terpusat
```

### 5. Pisahkan Modal/Dialog

```
// ❌ Modal inline dalam komponen parent
<template>
  <div>
    <!-- konten utama -->
    ...
    
    <!-- modal terkubur di bawah -->
    <div v-if="showDeleteModal" class="modal">
      <!-- 50 baris markup modal -->
    </div>
  </div>
</template>

// ✅ Modal sebagai komponen tersendiri
<template>
  <div>
    <!-- konten utama -->
    ...
    
    <DeleteConfirmDialog
      v-model:open="showDeleteModal"
      :item-name="selectedUser?.name"
      @confirm="handleDeleteConfirm"
    />
  </div>
</template>
```

---

## Struktur Folder yang Disarankan

### Laravel + Inertia (Direkomendasikan)
```
resources/js/
├── components/
│   ├── ui/                         ← generated shadcn-vue; jangan dipindah
│   ├── Layouts/                    ← layout global; ikuti casing project
│   │   └── AppLayout.vue
│   ├── shared/                     ← dipakai lintas feature
│   │   ├── DataTable.vue
│   │   ├── PageHeader.vue
│   │   └── SearchBar.vue
│   ├── users/                      ← komponen domain users
│   │   ├── UserList.vue
│   │   ├── UserCard.vue
│   │   ├── UserForm.vue
│   │   └── UserDeleteDialog.vue
│   └── products/
├── composables/
│   ├── useUsers.ts
│   ├── useUserForm.ts
│   └── usePagination.ts
├── lib/
│   └── utils.ts                    ← helper cn() shadcn-vue
├── pages/                          ← hanya entry page Inertia
│   ├── Users/
│   │   └── Index.vue
│   └── Dashboard.vue
├── app.ts
└── types/
    ├── user.ts
    └── index.ts
```

### Vue non-Laravel (Ikuti Struktur Existing)
```
src/
├── components/
│   ├── ui/                    ← shadcn-vue components
│   ├── layout/               ← Header, Sidebar, Footer
│   │   ├── AppHeader.vue
│   │   ├── AppSidebar.vue
│   │   └── AppFooter.vue
│   └── features/             ← Komponen spesifik fitur
│       ├── UserCard.vue
│       ├── ProductGrid.vue
│       └── OrderTable.vue
├── composables/
├── stores/                   ← Pinia stores
├── types/
└── pages/
```

---

## Cara Menyarankan Pemecahan ke User

Saat menemukan komponen yang perlu dipecah, sajikan dalam format ini:

```
### 📦 Saran Pemecahan Komponen: `UserManagement.vue`

File ini (287 baris) melakukan terlalu banyak hal:
1. Fetch & manage data users
2. Render tabel dengan sorting/filter
3. Handle form tambah/edit user
4. Handle dialog konfirmasi hapus

**Saran struktur baru:**

resources/js/
├── pages/Users/Index.vue                  ← ~60 baris; orchestrator
├── components/users/
│   ├── UserTable.vue                      ← tabel + sorting + filter
│   ├── UserFormDialog.vue                 ← dialog form tambah/edit
│   └── UserDeleteDialog.vue               ← dialog konfirmasi hapus
└── composables/
    ├── useUsers.ts                        ← fetch, CRUD operations
    └── useUserTable.ts                    ← sorting, filter, pagination

**Keuntungan:**
- Setiap file fokus pada satu tanggung jawab
- Mudah dites secara terpisah
- UserFormDialog bisa dipakai ulang di halaman lain
- Tim bisa mengerjakan komponen berbeda secara paralel
```

---

## Pola yang Sering Ditemukan & Solusinya

### Pola: Page God Component
```
// ❌ Satu file halaman yang mengelola segalanya
<script setup>
// 150 baris logika
</script>
<template>
<!-- 200 baris template dengan berbagai section -->
</template>

// ✅ Page sebagai orchestrator
<script setup>
// ~20 baris — hanya koordinasi antar bagian
</script>
<template>
  <PageHeader :title="title" />
  <DataSection :items="items" @select="handleSelect" />
  <SidePanel :selected="selectedItem" @close="clearSelection" />
</template>
```

### Pola: Inline Modal Hell
```
// ❌ Banyak v-if untuk berbagai modal
<div v-if="showAddModal">...</div>
<div v-if="showEditModal">...</div>
<div v-if="showDeleteModal">...</div>
<div v-if="showDetailModal">...</div>

// ✅ Tiap modal jadi komponen tersendiri
<AddUserDialog v-model:open="showAddModal" @created="refresh" />
<EditUserDialog v-model:open="showEditModal" :user="selectedUser" @updated="refresh" />
<DeleteDialog v-model:open="showDeleteModal" :name="selectedUser?.name" @confirmed="deleteUser" />
```

### Pola: Repeated Template Blocks
```html
<!-- ❌ Blok yang sama diulang -->
<div class="stat-card">
  <span>Total Users</span>
  <strong>{{ totalUsers }}</strong>
  <span class="change positive">+12%</span>
</div>
<div class="stat-card">
  <span>Revenue</span>
  <strong>{{ revenue }}</strong>
  <span class="change negative">-3%</span>
</div>

<!-- ✅ Komponen reusable -->
<StatCard
  v-for="stat in stats"
  :key="stat.key"
  :label="stat.label"
  :value="stat.value"
  :change="stat.change"
/>
```
