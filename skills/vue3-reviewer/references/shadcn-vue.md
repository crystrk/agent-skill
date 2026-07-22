# shadcn-vue — Komponen & Cara Penggunaan

Sumber resmi: https://www.shadcn-vue.com/docs/components

## Cara Install Komponen

```bash
npx shadcn-vue@latest add button
npx shadcn-vue@latest add input dialog table
```

---

## Daftar Komponen & Kapan Menggantinya

### 🔘 Button
```vue
<!-- Ganti: <button class="...custom styles..."> -->
<Button>Click me</Button>
<Button variant="outline" size="sm">Outline</Button>
<Button variant="destructive">Hapus</Button>
<Button variant="ghost" disabled>Disabled</Button>
<Button variant="link">Link style</Button>

<!-- Import -->
import { Button } from '@/components/ui/button'
```
**Variants:** `default` | `outline` | `secondary` | `ghost` | `link` | `destructive`  
**Sizes:** `default` | `sm` | `lg` | `icon`

---

### 📝 Input & Textarea
```vue
<!-- Ganti: <input type="text" class="..."> -->
<Input v-model="value" placeholder="Ketik di sini..." />
<Input type="email" />
<Input type="password" />

<!-- Ganti: <textarea class="..."> -->
<Textarea v-model="content" placeholder="Deskripsi..." rows="4" />

<!-- Import -->
import { Input } from '@/components/ui/input'
import { Textarea } from '@/components/ui/textarea'
```

---

### 🏷️ Label
```vue
<Label for="email">Email</Label>
<Input id="email" type="email" />

import { Label } from '@/components/ui/label'
```

---

### 📋 Form (dengan VeeValidate)
```vue
<script setup lang="ts">
import { useForm } from 'vee-validate'
import { toTypedSchema } from '@vee-validate/zod'
import * as z from 'zod'
import { Form, FormControl, FormField, FormItem, FormLabel, FormMessage } from '@/components/ui/form'

const schema = toTypedSchema(z.object({
  email: z.string().email(),
  password: z.string().min(8)
}))

const form = useForm({ validationSchema: schema })
const onSubmit = form.handleSubmit((values) => console.log(values))
</script>

<template>
  <form @submit="onSubmit">
    <FormField v-slot="{ componentField }" name="email">
      <FormItem>
        <FormLabel>Email</FormLabel>
        <FormControl>
          <Input v-bind="componentField" type="email" />
        </FormControl>
        <FormMessage />
      </FormItem>
    </FormField>
    <Button type="submit">Submit</Button>
  </form>
</template>
```

---

### 🗃️ Card
```vue
<!-- Ganti: <div class="border rounded p-4 shadow"> -->
<Card>
  <CardHeader>
    <CardTitle>Judul</CardTitle>
    <CardDescription>Deskripsi singkat</CardDescription>
  </CardHeader>
  <CardContent>
    <!-- konten -->
  </CardContent>
  <CardFooter>
    <Button>Action</Button>
  </CardFooter>
</Card>

import { Card, CardHeader, CardTitle, CardDescription, CardContent, CardFooter }
  from '@/components/ui/card'
```

---

### 💬 Dialog & AlertDialog
```vue
<!-- Ganti: modal/dialog custom -->
<Dialog v-model:open="isOpen">
  <DialogTrigger as-child>
    <Button>Buka Dialog</Button>
  </DialogTrigger>
  <DialogContent>
    <DialogHeader>
      <DialogTitle>Konfirmasi</DialogTitle>
      <DialogDescription>Apakah Anda yakin?</DialogDescription>
    </DialogHeader>
    <DialogFooter>
      <Button variant="outline" @click="isOpen = false">Batal</Button>
      <Button @click="handleConfirm">Ya, Lanjutkan</Button>
    </DialogFooter>
  </DialogContent>
</Dialog>

<!-- Untuk konfirmasi destruktif -->
<AlertDialog>
  <AlertDialogTrigger as-child>
    <Button variant="destructive">Hapus</Button>
  </AlertDialogTrigger>
  <AlertDialogContent>
    <AlertDialogHeader>
      <AlertDialogTitle>Hapus Data?</AlertDialogTitle>
      <AlertDialogDescription>Tindakan ini tidak bisa dibatalkan.</AlertDialogDescription>
    </AlertDialogHeader>
    <AlertDialogFooter>
      <AlertDialogCancel>Batal</AlertDialogCancel>
      <AlertDialogAction @click="handleDelete">Hapus</AlertDialogAction>
    </AlertDialogFooter>
  </AlertDialogContent>
</AlertDialog>
```

---

### 📋 Select & DropdownMenu
```vue
<script setup lang="ts">
import { EllipsisVertical } from '@lucide/vue'
</script>

<!-- Ganti: <select> native atau dropdown custom -->
<Select v-model="selected">
  <SelectTrigger>
    <SelectValue placeholder="Pilih opsi..." />
  </SelectTrigger>
  <SelectContent>
    <SelectItem value="option1">Opsi 1</SelectItem>
    <SelectItem value="option2">Opsi 2</SelectItem>
    <SelectGroup>
      <SelectLabel>Grup</SelectLabel>
      <SelectItem value="option3">Opsi 3</SelectItem>
    </SelectGroup>
  </SelectContent>
</Select>

<!-- Dropdown menu untuk action -->
<DropdownMenu>
  <DropdownMenuTrigger as-child>
    <Button variant="ghost" size="icon" aria-label="Buka menu aksi">
      <EllipsisVertical class="size-4" aria-hidden="true" />
    </Button>
  </DropdownMenuTrigger>
  <DropdownMenuContent>
    <DropdownMenuItem @click="handleEdit">Edit</DropdownMenuItem>
    <DropdownMenuSeparator />
    <DropdownMenuItem class="text-destructive" @click="handleDelete">Hapus</DropdownMenuItem>
  </DropdownMenuContent>
</DropdownMenu>
```

---

### 📊 Tabel
```vue
<!-- Ganti: <table> custom atau DataTable manual -->
<Table>
  <TableHeader>
    <TableRow>
      <TableHead>Nama</TableHead>
      <TableHead>Email</TableHead>
      <TableHead class="text-right">Aksi</TableHead>
    </TableRow>
  </TableHeader>
  <TableBody>
    <TableRow v-for="user in users" :key="user.id">
      <TableCell>{{ user.name }}</TableCell>
      <TableCell>{{ user.email }}</TableCell>
      <TableCell class="text-right">
        <Button variant="ghost" size="sm">Edit</Button>
      </TableCell>
    </TableRow>
  </TableBody>
</Table>
```

Untuk DataTable dengan fitur sort/filter/pagination, gunakan TanStack Table:
```bash
npx shadcn-vue@latest add data-table
npm install @tanstack/vue-table
```

---

### 🔔 Toast / Sonner
```vue
<script setup lang="ts">
import { useToast } from '@/components/ui/toast/use-toast'
// ATAU gunakan Sonner (lebih direkomendasikan):
import { toast } from 'vue-sonner'

const { toast: showToast } = useToast()

function handleSuccess() {
  // useToast
  showToast({ title: 'Berhasil', description: 'Data tersimpan' })
  
  // Sonner
  toast.success('Data tersimpan!')
  toast.error('Gagal menyimpan')
  toast.warning('Perhatian!')
}
</script>

<!-- Di App.vue atau layout -->
<Toaster /> <!-- untuk useToast -->
<Sonner />   <!-- untuk vue-sonner -->
```

---

### 🏷️ Badge
```vue
<!-- Ganti: <span class="px-2 py-1 rounded bg-green-100 text-green-800"> -->
<Badge>Default</Badge>
<Badge variant="secondary">Secondary</Badge>
<Badge variant="outline">Outline</Badge>
<Badge variant="destructive">Hapus</Badge>

import { Badge } from '@/components/ui/badge'
```

---

### ⏳ Skeleton
```vue
<!-- Ganti: loading spinner custom atau placeholder manual -->
<Skeleton class="h-4 w-[250px]" />
<Skeleton class="h-4 w-full" />
<Skeleton class="h-32 w-full rounded-lg" />

<!-- Contoh skeleton card -->
<Card>
  <CardContent class="pt-6">
    <Skeleton class="h-4 w-3/4 mb-2" />
    <Skeleton class="h-4 w-1/2 mb-4" />
    <Skeleton class="h-32 w-full" />
  </CardContent>
</Card>

import { Skeleton } from '@/components/ui/skeleton'
```

---

### ✅ Checkbox & Radio
```vue
<div class="flex items-center gap-2">
  <Checkbox id="terms" v-model:checked="accepted" />
  <Label for="terms">Saya setuju dengan syarat dan ketentuan</Label>
</div>

<RadioGroup v-model="selected">
  <div class="flex items-center gap-2">
    <RadioGroupItem value="option1" id="opt1" />
    <Label for="opt1">Opsi 1</Label>
  </div>
</RadioGroup>
```

---

### 🔄 Switch
```vue
<!-- Ganti: toggle button custom -->
<div class="flex items-center gap-2">
  <Switch v-model:checked="isEnabled" />
  <Label>Aktifkan notifikasi</Label>
</div>

import { Switch } from '@/components/ui/switch'
```

---

### 📅 Popover & DatePicker
```vue
<Popover>
  <PopoverTrigger as-child>
    <Button variant="outline">Buka Popover</Button>
  </PopoverTrigger>
  <PopoverContent class="w-80">
    <!-- konten popover -->
  </PopoverContent>
</Popover>
```

---

### 📂 Tabs
```vue
<!-- Ganti: tab custom dengan conditional rendering -->
<Tabs v-model="activeTab">
  <TabsList>
    <TabsTrigger value="profil">Profil</TabsTrigger>
    <TabsTrigger value="keamanan">Keamanan</TabsTrigger>
  </TabsList>
  <TabsContent value="profil">
    <!-- konten profil -->
  </TabsContent>
  <TabsContent value="keamanan">
    <!-- konten keamanan -->
  </TabsContent>
</Tabs>
```

---

### 📊 Progress
```vue
<!-- Ganti: progress bar custom -->
<Progress :model-value="uploadProgress" class="w-full" />

import { Progress } from '@/components/ui/progress'
```

---

### 📜 Sheet (Sidebar/Drawer)
```vue
<!-- Ganti: sidebar atau drawer custom -->
<Sheet v-model:open="isOpen">
  <SheetTrigger as-child>
    <Button>Buka Panel</Button>
  </SheetTrigger>
  <SheetContent side="right">
    <SheetHeader>
      <SheetTitle>Panel Detail</SheetTitle>
    </SheetHeader>
    <!-- konten -->
  </SheetContent>
</Sheet>
```
**Sides:** `top` | `right` | `bottom` | `left`

---

### 🎯 Command (Search/Palette)
```vue
<script setup lang="ts">
import { router } from '@inertiajs/vue3'
import { dashboard } from '@/routes'
</script>

<!-- Untuk search dengan keyboard navigation -->
<Command>
  <CommandInput placeholder="Cari..." />
  <CommandList>
    <CommandEmpty>Tidak ditemukan.</CommandEmpty>
    <CommandGroup heading="Navigasi">
      <CommandItem value="dashboard" @select="router.visit(dashboard())">
        Dashboard
      </CommandItem>
    </CommandGroup>
  </CommandList>
</Command>
```

---

## Setup shadcn-vue di Project Baru

```bash
# Install shadcn-vue CLI
npx shadcn-vue@latest init

# Pilih:
# - TypeScript: Yes
# - Framework: Vite
# - Style: Default atau New York
# - Base color: Slate/Gray/Zinc/dll
# - CSS variables: Yes
```

Ini akan mengatur:
- `tailwind.config.js`
- `src/lib/utils.ts` (dengan `cn()` helper)
- CSS variables di `src/assets/index.css`
- Path alias `@/` di `vite.config.ts`
