---
name: filament-reviewer
description: >
  Review dan audit kode Filament V5 secara menyeluruh. Gunakan skill ini setiap
  kali user meminta review, audit, refactor, atau pengecekan kode Filament —
  termasuk ketika user paste kode Resource, Form, Table, Action, Widget, Panel,
  atau file Laravel terkait Filament. Trigger saat ada kata kunci: filament,
  resource, panel, form schema, table columns, actions, widgets, infolist,
  livewire filament, filament v3 deprecated, filament v5, HasForms, HasTable,
  InteractsWithForms, InteractsWithTable, TextInput filament, TextColumn filament,
  upload file filament, S3 filament, FileUpload filament.
  Skill ini mencakup 7 dimensi: migrasi V3→V5, konvensi Filament V5, Laravel 13
  modern style, upload file S3 + FileHelper, layout form yang menarik, pemecahan
  Resource, dan integrasi skill terkait (laravel-best-practices, inertia-vue-development,
  shadcn-vue). Trigger juga saat ada kata kunci: form layout filament, section filament,
  tabs form filament, wizard filament, split layout, repeater filament, form menarik.
---

# Filament V5 Code Reviewer

Skill ini memandu review kode Filament V5 dalam **7 dimensi** secara berurutan.

---

## 🔗 Skill Terkait — Baca Jika Relevan

| Skill | Kapan Dikonsultasi |
|---|---|
| **`laravel-best-practices`** | Untuk konvensi Laravel 13: Service class, Repository, Form Request, Policy, Route model binding |
| **`inertia-vue-development`** | Jika panel menggunakan Inertia atau ada custom Vue page di dalam Filament |
| **`shadcn-vue`** | Jika ada custom Livewire/Volt component di luar Filament yang butuh shadcn-vue |
| **`vue3-reviewer`** | Jika ada custom Vue component yang diembed di dalam Filament panel |
| **`tailwindcss-development`** | Untuk custom styling, tema panel, atau custom view Filament |
| **`pest-testing:`** | Jika membuat testing prioritaskan menggunakan Pest |

---

## Alur Review — 6 Dimensi

### Dimensi 1 — Migrasi Filament V3 → V5
📄 Detail: `references/deprecated-v3.md`

Deteksi cepat kode V3 yang sudah deprecated:

| V3 (Deprecated) | V5 (Pengganti) |
|---|---|
| `Forms\Components\FileUpload` | `Forms\Components\FileUpload` (API baru) |
| `->rules([...])` on field | `->rule(...)` atau `->rules(...)` (chain langsung) |
| `TextInput::make()->mask(...)` | `->mask(RawJs::make(...))` |
| `Forms\Components\Builder` | `Forms\Components\Builder` (API berubah) |
| `Tables\Columns\BadgeColumn` | `Tables\Columns\TextColumn::make()->badge()` |
| `Tables\Columns\ImageColumn` | `Tables\Columns\ImageColumn` (tetap, API berubah) |
| `Tables\Columns\TagsColumn` | `Tables\Columns\TextColumn::make()->badge()->separator(',')` |
| `Tables\Filters\Filter::make()->form([...])` | `Tables\Filters\Filter::make()->form([...])` (tetap) |
| `->translateLabel()` | Sudah otomatis — hapus |
| `static::getModel()` | `static::getModel()` tetap, tapi prefer `$this->getOwnerRecord()` di relasi |
| `InteractsWithForms` trait manual | Otomatis via `HasForms` concern |
| `Filament\Pages\Page` extend | `Filament\Panel\PanelProvider` untuk konfigurasi |
| `->disableLabel()` | `->hiddenLabel()` |
| `->disablePlaceholderSelection()` | `->selectablePlaceholder(false)` |
| `->isRequired()` → `required()` | Sudah built-in via `->required()` |
| `Action::make()->color('success')` | `->color(Color::Success)` atau string tetap bisa |
| `->hidden(fn() => ...)` closure | `->visible(fn() => ...)` / `->hidden(fn() => ...)` tetap |
| `->afterStateUpdated(fn($state) => ...)` | `->afterStateUpdated(fn($state, $set) => ...)` — injeksi `$set`, `$get` |

### Dimensi 2 — Konvensi Filament V5
📄 Detail: `references/conventions-v5.md`

Cek cepat hal kritis:
- `Resource` harus punya `$model`, `$navigationIcon`, `$navigationGroup`
- Form schema wajib dalam method `form(Form $form): Form`
- Table columns wajib dalam method `table(Table $table): Table`
- Setiap `Action` harus punya `->authorize()` atau via Policy
- Gunakan `->relationship()` untuk relasi, bukan query manual
- `->searchable()`, `->sortable()`, `->filterable()` pada column yang relevan
- Infolist gunakan `Infolists\Components\*` bukan form read-only
- Widget extend `Filament\Widgets\*Widget` sesuai tipe
- Gunakan `->columnSpan()` / `->columns()` untuk layout responsif

### Dimensi 3 — Laravel 13 Modern Style
📄 Detail: `references/laravel13-style.md`

> Konsultasi juga skill **`laravel-best-practices`** untuk aturan lengkap.

Cek kritis:
- **Controller harus tipis** — logika bisnis di Service class atau Action class
- Gunakan **Form Request** untuk validasi (`php artisan make:request`)
- Gunakan **Policy** untuk otorisasi (`php artisan make:policy`)
- Gunakan **Route Model Binding** — bukan `Model::find($id)` manual
- Gunakan **Enum** untuk status/tipe (PHP 8.1+)
- Hindari `DB::table()` raw — gunakan Eloquent + Scope
- Gunakan **readonly properties** dan **constructor promotion** di DTO/Value Object
- Middleware di route, bukan di controller constructor
- `$fillable` eksplisit di Model — jangan `$guarded = []`

### Dimensi 4 — Upload File: S3 + FileHelper (WAJIB)
📄 Detail: `references/file-upload-s3.md`

> ⚠️ **ATURAN KERAS**: Semua upload file **wajib** menggunakan S3 sebagai storage.
> Wajib menggunakan `FileHelper` untuk penamaan file yang aman.
> Saat update file: upload baru ke S3 **dulu**, verifikasi sukses, **baru** hapus file lama.

Pola yang WAJIB:
```php
// Dalam Filament Form
FileUpload::make('photo')
    ->disk('s3')
    ->directory('users/photos')
    ->visibility('private')           // atau 'public' sesuai kebutuhan
    ->getUploadedFileNameForStorageUsing(
        fn(TemporaryUploadedFile $file) => FileHelper::generateName($file)
    )
    ->deleteUploadedFileUsing(function (string $file) {
        // JANGAN hapus otomatis — kelola manual di Observer/Service
    })
    ->storeFileNamesIn('photo_original_name'),
```

Yang DILARANG:
```php
// ❌ Storage lokal
->disk('public')
->disk('local')

// ❌ Penamaan file tidak aman (nama asli user)
// tanpa FileHelper

// ❌ Hapus file lama sebelum upload baru berhasil
Storage::delete($oldFile);
$newPath = $request->file('photo')->store('photos');
```

### Dimensi 5 — Layout Form yang Menarik
📄 Detail: `references/form-layout.md`

Evaluasi apakah layout form sudah optimal secara visual dan UX:

| Temuan | Saran |
|---|---|
| Semua field jejer ke bawah tanpa grouping | Gunakan `Section::make()` per topik |
| Section tanpa ikon atau deskripsi | Tambahkan `->icon()` dan `->description()` |
| Form > 10 field dalam satu Section | Pecah ke `Tabs::make()` atau beberapa Section |
| Form registrasi/onboarding panjang | Gunakan `Wizard::make()` multi-step |
| Konten utama dan metadata dicampur | Gunakan `Split::make()` — konten kiri, meta kanan |
| Field opsional/lanjutan selalu tampil | Gunakan `Section->collapsible()->collapsed()` |
| FileUpload tanpa deskripsi panduan | Tambahkan `->helperText()` format & ukuran |
| Field berulang (alamat, kontak, item) | Gunakan `Repeater` dengan `->collapsible()` dan `->itemLabel()` |
| Field kondisional belum pakai `->live()` | Tambahkan `->live()` pada trigger field |
| Prefix/suffix field tidak konsisten | Standarkan `->prefixIcon()` atau `->prefix()` teks |

Cek cepat aspek visual:
- `->prefixIcon()` pada field yang punya makna visual (email, phone, harga, berat)
- `->helperText()` pada field yang butuh panduan (format, batasan, contoh)
- `->hint()` + `->hintIcon()` untuk peringatan inline
- `->placeholder()` yang informatif — bukan hanya "Masukkan ..."
- `->columnSpanFull()` pada field yang membutuhkan ruang lebih (Textarea, RichEditor)
- `Section->aside()` untuk form settings dengan deskripsi panjang di kiri
- `->persistTabInQueryString()` agar tab aktif tidak hilang saat refresh

### Dimensi 6 — Pemecahan Resource
📄 Detail: `references/resource-split.md`

Sinyal Resource perlu dipecah:
- File Resource > **300 baris**
- `form()` method > **80 baris** — pecah ke Form class tersendiri
- `table()` method > **60 baris** — pecah ke Table class tersendiri
- Ada > **5 RelationManager** — pertimbangkan grouping
- Logic di `mutateFormDataBeforeCreate` / `mutateFormDataBeforeSave` > 20 baris → pindah ke Service

### Dimensi 7 — Keamanan & Performa
📄 Detail: `references/security-performance.md`

Cek kritis:
- Setiap Resource wajib definisikan `getEloquentQuery()` untuk scope data user
- Setiap Action wajib ada `->authorize()` atau Policy check
- Lazy load relation di Table — jangan eager load semua
- Gunakan `->deferLoading()` pada Widget yang berat
- Hindari N+1 di `->getStateUsing()` — gunakan `->relationship()` atau eager load

---

## Format Output Review

```
## 🔍 Hasil Review Filament V5

### ✅ Sudah Baik
[hal yang sudah benar — jangan kosongkan]

---
### 🔴 Kritis — Wajib Diperbaiki
**[Nama Masalah]** | File: X, Baris: Y
- Masalah: [penjelasan]
- Solusi:
```php
[kode perbaikan]
```

---
### 🟡 Perlu Diperhatikan
[temuan medium priority]

---
### 🔵 Saran Peningkatan
[opsional tapi direkomendasikan]

---
### 📦 Saran Pemecahan Resource/File
[jika ada file yang perlu dipecah]

---
### ✨ Kode Hasil Refactor
[kode lengkap yang sudah diperbaiki]
```

---

## Checklist Sebelum Selesai

- [ ] 7 dimensi sudah semua dijalankan
- [ ] Setiap kode V3 deprecated sudah ada penggantinya yang konkret
- [ ] Upload file sudah pakai S3 + FileHelper + urutan upload-baru-dulu
- [ ] Controller/Resource tidak berisi logika bisnis berat
- [ ] Setiap Action sudah ada otorisasi
- [ ] Form layout sudah dievaluasi: Section, Tabs, Split, Wizard sesuai konteks
- [ ] Field punya prefixIcon / helperText / placeholder yang informatif
- [ ] Jika ada kode Laravel → sudah mention skill `laravel-best-practices`
- [ ] Jika ada custom Vue → sudah mention skill `vue3-reviewer`
