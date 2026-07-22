# Layout Form Filament V5 — Panduan Visual & Menarik

## Prinsip Dasar Layout Form yang Baik

1. **Grouping logis** — field yang berkaitan dikumpulkan dalam satu Section
2. **Grid responsif** — gunakan `->columns()` agar form tidak terlalu panjang ke bawah
3. **Visual hierarchy** — Section dengan ikon, deskripsi, dan warna yang konsisten
4. **Progressive disclosure** — tampilkan field lanjutan hanya jika relevan (`->visible()`)
5. **Feedback kontekstual** — hint, helper text, dan placeholder yang informatif

---

## 1. Section — Blok Utama Layout

### Section Dasar dengan Ikon & Deskripsi

```php
use Filament\Forms\Components\Section;

Section::make('Informasi Pribadi')
    ->description('Data utama yang akan ditampilkan di profil publik.')
    ->icon('heroicon-o-user-circle')
    ->schema([
        TextInput::make('name')->required(),
        TextInput::make('email')->email()->required(),
    ])
    ->columns(2),
```

### Section Collapsible — untuk Form Panjang

```php
// Section yang bisa dilipat — berguna untuk field opsional/lanjutan
Section::make('Pengaturan Lanjutan')
    ->description('Konfigurasi tambahan yang opsional.')
    ->icon('heroicon-o-cog-6-tooth')
    ->schema([...])
    ->collapsible()
    ->collapsed(),          // default tertutup

// Collapsed berdasarkan kondisi
Section::make('Riwayat Audit')
    ->schema([...])
    ->collapsible()
    ->collapsed(fn (?Model $record): bool => $record === null),
```

### Section dengan Aside (2 Kolom Asimetris)

```php
// Layout dengan sidebar kiri (deskripsi) dan konten kanan
Section::make('Foto Profil')
    ->description('Upload foto profil Anda. Format JPG, PNG, atau WebP. Maks 2MB.')
    ->aside()               // ← kunci utama: deskripsi jadi kolom kiri
    ->schema([
        FileUpload::make('avatar')
            ->disk('s3')
            ->directory('avatars')
            ->image()
            ->imageEditor()
            ->circleCropper()
            ->hiddenLabel(),
    ]),

// Contoh penggunaan aside yang elegan untuk settings page
Section::make('Keamanan Akun')
    ->description('Perbarui password dan pengaturan keamanan akun Anda secara berkala.')
    ->aside()
    ->schema([
        TextInput::make('current_password')
            ->password()
            ->revealable()
            ->required(),
        TextInput::make('password')
            ->password()
            ->revealable()
            ->confirmed()
            ->required(),
        TextInput::make('password_confirmation')
            ->password()
            ->revealable()
            ->required(),
    ]),
```

### Section Berwarna / Bertema

```php
// Gunakan extraAttributes untuk visual emphasis
Section::make('⚠️ Zona Berbahaya')
    ->description('Tindakan di sini bersifat permanen dan tidak bisa dibatalkan.')
    ->schema([
        Toggle::make('suspend_account')->label('Tangguhkan akun ini'),
    ])
    ->extraAttributes([
        'class' => 'border-red-300 dark:border-red-700',
    ]),
```

---

## 2. Grid & Columns — Tata Letak Responsif

### Dasar: `->columns()`

```php
// Di level Section
Section::make('Alamat')
    ->schema([
        TextInput::make('address')->columnSpanFull(),   // lebar penuh
        TextInput::make('city'),                        // setengah
        TextInput::make('province'),                    // setengah
        TextInput::make('postal_code'),                 // setengah
        Select::make('country')->columnSpanFull(),      // lebar penuh lagi
    ])
    ->columns(2),

// Responsif: berbeda kolom per breakpoint
Section::make('Kontak')
    ->schema([...])
    ->columns([
        'default' => 1,     // mobile: 1 kolom
        'sm'      => 2,     // tablet: 2 kolom
        'lg'      => 3,     // desktop: 3 kolom
        'xl'      => 4,     // wide: 4 kolom
    ]),
```

### Grid Component — Tanpa Section Wrapper

```php
use Filament\Forms\Components\Grid;

Grid::make()
    ->schema([
        TextInput::make('first_name'),
        TextInput::make('last_name'),
        TextInput::make('email')->columnSpan(2),
    ])
    ->columns(2),
```

### columnSpan — Kontrol Lebar Field

```php
// Di dalam grid 3 kolom
Section::make()->schema([
    TextInput::make('title')->columnSpanFull(),         // lebar penuh (3/3)
    TextInput::make('slug')->columnSpan(2),             // 2/3 lebar
    Select::make('status')->columnSpan(1),              // 1/3 lebar

    // Responsif per field
    Textarea::make('excerpt')->columnSpan([
        'default' => 1,
        'md'      => 2,
        'xl'      => 3,
    ]),
])->columns(3),
```

---

## 3. Split — Layout Dua Panel Horizontal

```php
use Filament\Forms\Components\Split;

// Split: panel kiri lebih besar dari kanan
Split::make([
    Section::make('Detail Produk')
        ->schema([
            TextInput::make('name')->required(),
            Textarea::make('description')->rows(4),
            RichEditor::make('content'),
        ]),

    Section::make('Meta & Pengaturan')
        ->schema([
            Select::make('category_id')->relationship('category', 'name'),
            TagsInput::make('tags'),
            Toggle::make('is_featured'),
            Toggle::make('is_published'),
            DateTimePicker::make('published_at'),
        ])
        ->grow(false),  // ← panel kanan tidak ikut melebar
]),
```

---

## 4. Tabs — Form Multi-Tab

```php
use Filament\Forms\Components\Tabs;
use Filament\Forms\Components\Tabs\Tab;

Tabs::make('Form Produk')
    ->tabs([
        Tab::make('Informasi Dasar')
            ->icon('heroicon-o-document-text')
            ->schema([
                TextInput::make('name')->required(),
                Textarea::make('description'),
                RichEditor::make('content')
                    ->fileAttachmentsDisk('s3')
                    ->fileAttachmentsDirectory('products/attachments'),
            ]),

        Tab::make('Media')
            ->icon('heroicon-o-photo')
            ->badge(fn (?Model $record) => $record?->images()->count())  // badge jumlah
            ->schema([
                FileUpload::make('thumbnail')
                    ->disk('s3')
                    ->directory('products/thumbnails')
                    ->image()
                    ->imageEditor()
                    ->getUploadedFileNameForStorageUsing(
                        fn (TemporaryUploadedFile $file) => FileHelper::generateName($file)
                    ),
                FileUpload::make('gallery')
                    ->disk('s3')
                    ->directory('products/gallery')
                    ->multiple()
                    ->image()
                    ->reorderable()
                    ->getUploadedFileNameForStorageUsing(
                        fn (TemporaryUploadedFile $file) => FileHelper::generateName($file)
                    ),
            ]),

        Tab::make('Harga & Stok')
            ->icon('heroicon-o-currency-dollar')
            ->schema([
                TextInput::make('price')
                    ->numeric()
                    ->prefix('Rp')
                    ->required(),
                TextInput::make('stock')
                    ->numeric()
                    ->minValue(0),
                Toggle::make('track_stock'),
            ])
            ->columns(2),

        Tab::make('SEO')
            ->icon('heroicon-o-magnifying-glass')
            ->schema([
                TextInput::make('meta_title')->maxLength(60),
                Textarea::make('meta_description')->maxLength(160)->rows(3),
                TextInput::make('slug')
                    ->unique(ignoreRecord: true)
                    ->required(),
            ]),
    ])
    ->persistTabInQueryString('tab'),   // ← simpan tab aktif di URL
```

---

## 5. Wizard — Form Multi-Step

```php
use Filament\Forms\Components\Wizard;
use Filament\Forms\Components\Wizard\Step;

Wizard::make([
    Step::make('Informasi Akun')
        ->description('Data login dan identitas.')
        ->icon('heroicon-o-user')
        ->schema([
            TextInput::make('name')->required(),
            TextInput::make('email')->email()->unique()->required(),
            TextInput::make('password')->password()->confirmed()->required(),
            TextInput::make('password_confirmation')->password()->required(),
        ])
        ->columns(2),

    Step::make('Profil')
        ->description('Lengkapi profil Anda.')
        ->icon('heroicon-o-identification')
        ->schema([
            FileUpload::make('avatar')
                ->disk('s3')
                ->directory('users/avatars')
                ->image()
                ->circleCropper()
                ->getUploadedFileNameForStorageUsing(
                    fn (TemporaryUploadedFile $file) => FileHelper::generateName($file)
                ),
            TextInput::make('phone'),
            Textarea::make('bio')->rows(3),
        ]),

    Step::make('Konfirmasi')
        ->description('Periksa kembali data Anda.')
        ->icon('heroicon-o-check-circle')
        ->schema([
            Placeholder::make('review')
                ->content('Silakan periksa semua data sebelum menyimpan.'),
            Checkbox::make('agree_terms')
                ->label('Saya menyetujui syarat dan ketentuan.')
                ->accepted(),
        ]),
])
->skippable()           // opsional: izinkan loncat step
->submitAction(new HtmlString('<button type="submit">Daftar Sekarang</button>')),
```

---

## 6. Fieldset — Group Field dengan Border

```php
use Filament\Forms\Components\Fieldset;

// Lebih ringan dari Section — tanpa background, hanya border
Fieldset::make('Dimensi Produk')
    ->schema([
        TextInput::make('weight')->numeric()->suffix('kg'),
        TextInput::make('length')->numeric()->suffix('cm'),
        TextInput::make('width')->numeric()->suffix('cm'),
        TextInput::make('height')->numeric()->suffix('cm'),
    ])
    ->columns(4),
```

---

## 7. Field Styling yang Mempercantik Form

### Prefix, Suffix, dan Inline Prefix

```php
// Prefix/suffix teks
TextInput::make('price')
    ->prefix('Rp')
    ->numeric()
    ->placeholder('0'),

TextInput::make('domain')
    ->suffix('.com')
    ->placeholder('namasitus'),

TextInput::make('weight')
    ->numeric()
    ->suffix('kg')
    ->prefixIcon('heroicon-o-scale'),   // ikon di kiri

// Inline prefix dengan select (misal kode negara + nomor HP)
TextInput::make('phone')
    ->tel()
    ->prefixIcon('heroicon-o-phone')
    ->prefix(
        Select::make('phone_code')
            ->options(['+62' => '🇮🇩 +62', '+65' => '🇸🇬 +65', '+60' => '🇲🇾 +60'])
            ->default('+62')
            ->selectablePlaceholder(false)
            ->extraAttributes(['style' => 'min-width: 110px']),
    ),
```

### Helper Text & Hint

```php
TextInput::make('username')
    ->helperText('Hanya huruf kecil, angka, dan underscore. 3–20 karakter.')
    ->hint('Tidak bisa diubah setelah pendaftaran.')
    ->hintColor('warning')
    ->hintIcon('heroicon-m-exclamation-triangle'),

FileUpload::make('ktp')
    ->helperText('Upload foto KTP yang jelas dan terbaca. Format JPG/PNG, maks 2MB.'),
```

### Placeholder yang Informatif

```php
TextInput::make('email')->placeholder('nama@perusahaan.com'),
TextInput::make('phone')->placeholder('08xx xxxx xxxx'),
Textarea::make('address')->placeholder("Jl. Sudirman No. 123\nJakarta Selatan, 12190"),
```

---

## 8. Conditional Field — Tampilkan/Sembunyikan Dinamis

```php
// Field muncul berdasarkan pilihan lain
Select::make('payment_method')
    ->options([
        'bank_transfer' => 'Transfer Bank',
        'credit_card'   => 'Kartu Kredit',
        'cash'          => 'Tunai',
    ])
    ->live()    // ← wajib agar perubahan memicu re-render
    ->required(),

// Field bank transfer — muncul hanya jika payment_method = bank_transfer
Section::make('Detail Transfer Bank')
    ->schema([
        Select::make('bank_name')
            ->options(['BCA' => 'BCA', 'BNI' => 'BNI', 'Mandiri' => 'Mandiri'])
            ->required(),
        TextInput::make('account_number')->required(),
        TextInput::make('account_name')->required(),
    ])
    ->visible(fn (Get $get): bool => $get('payment_method') === 'bank_transfer')
    ->columns(3),

// Field kartu kredit — muncul hanya jika payment_method = credit_card
Section::make('Detail Kartu Kredit')
    ->schema([
        TextInput::make('card_number')->mask('9999 9999 9999 9999'),
        TextInput::make('card_holder'),
        TextInput::make('expiry')->mask('99/99'),
        TextInput::make('cvv')->mask('999'),
    ])
    ->visible(fn (Get $get): bool => $get('payment_method') === 'credit_card')
    ->columns(2),
```

---

## 9. Repeater — Input Data Berulang

```php
use Filament\Forms\Components\Repeater;

Repeater::make('addresses')
    ->label('Daftar Alamat')
    ->schema([
        Select::make('type')
            ->options(['home' => 'Rumah', 'office' => 'Kantor', 'other' => 'Lainnya'])
            ->required()
            ->columnSpan(1),
        TextInput::make('label')
            ->placeholder('Contoh: Rumah Utama')
            ->columnSpan(1),
        Textarea::make('address')
            ->required()
            ->rows(2)
            ->columnSpanFull(),
        TextInput::make('city')->required(),
        TextInput::make('postal_code'),
        Toggle::make('is_default')->label('Jadikan alamat utama'),
    ])
    ->columns(2)
    ->addActionLabel('Tambah Alamat')
    ->reorderable()
    ->collapsible()
    ->cloneable()           // ← fitur V5: bisa duplikasi item
    ->itemLabel(fn (array $state): ?string => $state['label'] ?? null)  // label per item
    ->defaultItems(1)
    ->maxItems(5),
```

---

## 10. Placeholder — Konten Statis dalam Form

```php
use Filament\Forms\Components\Placeholder;

// Tampilkan info yang tidak bisa diedit
Placeholder::make('created_at')
    ->label('Dibuat pada')
    ->content(fn (?Model $record): string => $record?->created_at->format('d M Y, H:i') ?? '-'),

Placeholder::make('subscription_info')
    ->label('Status Langganan')
    ->content(new HtmlString('<span class="text-success-600 font-medium">✓ Aktif hingga 31 Des 2025</span>')),

// Info box dengan styling
Placeholder::make('notice')
    ->hiddenLabel()
    ->content(new HtmlString('
        <div class="p-4 bg-blue-50 dark:bg-blue-950 border border-blue-200 dark:border-blue-800 rounded-lg text-sm text-blue-700 dark:text-blue-300">
            <strong>ℹ️ Informasi:</strong> Perubahan email memerlukan verifikasi ulang.
        </div>
    ')),
```

---

## 11. Pola Form Lengkap yang Menarik

Contoh form pengguna yang menggunakan semua teknik di atas:

```php
public static function form(Form $form): Form
{
    return $form
        ->schema([
            // ── Panel utama + sidebar ──────────────────────────
            Split::make([

                // Kolom kiri: konten utama
                \Filament\Forms\Components\Group::make()
                    ->schema([
                        // Tab untuk konten yang panjang
                        Tabs::make()
                            ->tabs([
                                Tab::make('Profil')
                                    ->icon('heroicon-o-user')
                                    ->schema([
                                        Section::make('Identitas')
                                            ->schema([
                                                TextInput::make('name')
                                                    ->prefixIcon('heroicon-o-user')
                                                    ->required()
                                                    ->columnSpanFull(),
                                                TextInput::make('email')
                                                    ->prefixIcon('heroicon-o-envelope')
                                                    ->email()
                                                    ->unique(ignoreRecord: true)
                                                    ->required(),
                                                TextInput::make('phone')
                                                    ->prefixIcon('heroicon-o-phone')
                                                    ->tel(),
                                            ])
                                            ->columns(2),

                                        Section::make('Bio')
                                            ->schema([
                                                Textarea::make('bio')
                                                    ->hiddenLabel()
                                                    ->rows(4)
                                                    ->placeholder('Ceritakan sedikit tentang diri Anda...')
                                                    ->maxLength(500)
                                                    ->helperText(fn ($state) => strlen($state ?? '') . '/500 karakter'),
                                            ]),
                                    ]),

                                Tab::make('Alamat')
                                    ->icon('heroicon-o-map-pin')
                                    ->schema([
                                        Repeater::make('addresses')
                                            ->hiddenLabel()
                                            ->schema([
                                                TextInput::make('street')->required()->columnSpanFull(),
                                                TextInput::make('city')->required(),
                                                TextInput::make('postal_code'),
                                                Toggle::make('is_primary')->label('Alamat utama'),
                                            ])
                                            ->columns(2)
                                            ->addActionLabel('Tambah Alamat')
                                            ->collapsible()
                                            ->itemLabel(fn (array $state) => $state['city'] ?? 'Alamat Baru'),
                                    ]),

                                Tab::make('Keamanan')
                                    ->icon('heroicon-o-lock-closed')
                                    ->schema([
                                        Section::make()
                                            ->schema([
                                                TextInput::make('password')
                                                    ->password()
                                                    ->revealable()
                                                    ->confirmed()
                                                    ->helperText('Minimal 8 karakter.'),
                                                TextInput::make('password_confirmation')
                                                    ->password()
                                                    ->revealable(),
                                            ])
                                            ->columns(2),
                                    ]),
                            ])
                            ->persistTabInQueryString('tab'),
                    ])
                    ->columnSpan(['lg' => 2]),

                // Kolom kanan: sidebar metadata
                \Filament\Forms\Components\Group::make()
                    ->schema([
                        Section::make('Foto Profil')
                            ->schema([
                                FileUpload::make('avatar')
                                    ->disk('s3')
                                    ->directory('users/avatars')
                                    ->visibility('private')
                                    ->image()
                                    ->imageEditor()
                                    ->circleCropper()
                                    ->hiddenLabel()
                                    ->getUploadedFileNameForStorageUsing(
                                        fn (TemporaryUploadedFile $file) => FileHelper::generateName($file)
                                    ),
                            ]),

                        Section::make('Status & Peran')
                            ->schema([
                                Select::make('role')
                                    ->options(UserRole::class)
                                    ->required()
                                    ->native(false),
                                Toggle::make('is_active')
                                    ->label('Akun Aktif')
                                    ->default(true),
                            ]),

                        Section::make('Info Akun')
                            ->schema([
                                Placeholder::make('created_at')
                                    ->label('Terdaftar')
                                    ->content(fn ($record) => $record?->created_at?->diffForHumans() ?? '-'),
                                Placeholder::make('last_login')
                                    ->label('Login Terakhir')
                                    ->content(fn ($record) => $record?->last_login_at?->diffForHumans() ?? 'Belum pernah'),
                            ])
                            ->hidden(fn ($record) => $record === null),
                    ])
                    ->columnSpan(['lg' => 1]),

            ])->from('lg'),  // ← Split aktif mulai breakpoint lg
        ]);
}
```

---

## Ringkasan: Pilihan Layout Berdasarkan Kebutuhan

| Kebutuhan | Komponen |
|---|---|
| Grouping field dengan header | `Section::make()` |
| Form panjang yang bisa dilipat | `Section::make()->collapsible()` |
| Deskripsi di kiri, form di kanan | `Section::make()->aside()` |
| Form di kiri, metadata di kanan | `Split::make([..., ...])` |
| Form multi-topik panjang | `Tabs::make()` |
| Form pendaftaran/onboarding | `Wizard::make()` |
| Group kecil dengan border tipis | `Fieldset::make()` |
| Kolom tanpa wrapper visual | `Grid::make()` |
| Input berulang (alamat, item) | `Repeater::make()` |
| Teks/info statis dalam form | `Placeholder::make()` |
| Field kondisional | `->visible(fn(Get $get) => ...)` |
