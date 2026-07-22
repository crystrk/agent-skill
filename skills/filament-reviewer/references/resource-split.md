# Pemecahan Resource Filament — Panduan & Contoh

## Kapan Resource Perlu Dipecah?

### 🔴 Wajib Dipecah
- File Resource > **300 baris** total
- Method `form()` > **80 baris** — pecah ke Form class tersendiri
- Method `table()` > **60 baris** — pecah ke Table class tersendiri
- Logic di `mutateFormDataBeforeCreate/Save` > **20 baris** → pindah ke Service
- Ada > **5 RelationManager** di satu Resource

### 🟡 Pertimbangkan Dipecah
- Form schema yang digunakan ulang di beberapa tempat
- Table columns yang identik dengan Resource lain
- Action yang kompleks dengan banyak langkah

---

## Strategi 1 — Pisah Form Schema ke Class Tersendiri

```php
// ❌ SEBELUM: form() 120 baris di dalam Resource
public static function form(Form $form): Form
{
    return $form->schema([
        Section::make('Data Pribadi')->schema([
            TextInput::make('name')...,
            TextInput::make('email')...,
            // ... 20+ field
        ]),
        Section::make('Alamat')->schema([
            // ... 15+ field
        ]),
        Section::make('Upload Dokumen')->schema([
            FileUpload::make('ktp')...,
            // ... 10+ field
        ]),
    ]);
}

// ✅ SESUDAH: Resource menjadi bersih
public static function form(Form $form): Form
{
    return $form->schema([
        ...UserFormSchema::personalSection(),
        ...UserFormSchema::addressSection(),
        ...UserFormSchema::documentSection(),
    ]);
}
```

```php
// app/Filament/Resources/UserResource/Schemas/UserFormSchema.php
namespace App\Filament\Resources\UserResource\Schemas;

use Filament\Forms\Components\Section;
use Filament\Forms\Components\TextInput;
use App\Helpers\FileHelper;
use Livewire\Features\SupportFileUploads\TemporaryUploadedFile;

class UserFormSchema
{
    public static function personalSection(): array
    {
        return [
            Section::make('Data Pribadi')
                ->icon('heroicon-o-user')
                ->schema([
                    TextInput::make('name')
                        ->label('Nama Lengkap')
                        ->required()
                        ->maxLength(255),
                    TextInput::make('email')
                        ->email()
                        ->required()
                        ->unique(ignoreRecord: true),
                    // ... field lainnya
                ])
                ->columns(2),
        ];
    }

    public static function addressSection(): array
    {
        return [
            Section::make('Alamat')
                ->icon('heroicon-o-map-pin')
                ->schema([
                    // ... address fields
                ])
                ->columns(2),
        ];
    }

    public static function documentSection(): array
    {
        return [
            Section::make('Dokumen')
                ->icon('heroicon-o-document')
                ->schema([
                    FileUpload::make('ktp')
                        ->label('KTP')
                        ->disk('s3')
                        ->directory('users/documents/ktp')
                        ->visibility('private')
                        ->image()
                        ->maxSize(2048)
                        ->getUploadedFileNameForStorageUsing(
                            fn (TemporaryUploadedFile $file) => FileHelper::generateName($file)
                        ),
                    // ... file fields lainnya
                ]),
        ];
    }
}
```

---

## Strategi 2 — Pisah Table Columns ke Class Tersendiri

```php
// app/Filament/Resources/UserResource/Schemas/UserTableSchema.php
namespace App\Filament\Resources\UserResource\Schemas;

use Filament\Tables\Columns\TextColumn;
use Filament\Tables\Columns\ImageColumn;
use Filament\Tables\Columns\IconColumn;
use Filament\Tables\Columns\BadgeColumn;
use Filament\Support\Enums\FontWeight;

class UserTableSchema
{
    public static function columns(): array
    {
        return [
            ImageColumn::make('avatar')
                ->circular()
                ->disk('s3')
                ->width(40)->height(40),

            TextColumn::make('name')
                ->weight(FontWeight::Medium)
                ->searchable()
                ->sortable(),

            TextColumn::make('email')
                ->searchable()
                ->copyable(),

            TextColumn::make('role')
                ->badge()
                ->sortable(),

            IconColumn::make('is_active')
                ->boolean()
                ->label('Aktif'),

            TextColumn::make('created_at')
                ->dateTime('d M Y')
                ->sortable()
                ->toggleable(isToggledHiddenByDefault: true),
        ];
    }

    public static function filters(): array
    {
        return [
            // ... filter definitions
        ];
    }

    public static function actions(): array
    {
        return [
            // ... action definitions
        ];
    }
}

// Di Resource:
public static function table(Table $table): Table
{
    return $table
        ->columns(UserTableSchema::columns())
        ->filters(UserTableSchema::filters())
        ->actions(UserTableSchema::actions())
        ->defaultSort('created_at', 'desc');
}
```

---

## Strategi 3 — Pisah Logic ke Service

```php
// ❌ SEBELUM: logika berat di hook Resource
class EditUser extends EditRecord
{
    protected function mutateFormDataBeforeSave(array $data): array
    {
        // 50 baris logika di sini — SALAH
        if ($data['avatar'] !== $this->record->avatar) {
            // hapus file lama, proses thumbnail, dll
        }
        // hash password jika diisi
        // generate slug dari nama
        // update related records
        return $data;
    }
}

// ✅ SESUDAH: delegate ke Service
class EditUser extends EditRecord
{
    protected string $oldAvatar;

    protected function mutateFormDataBeforeSave(array $data): array
    {
        $this->oldAvatar = $this->record->avatar;

        // Hash password hanya jika diisi
        if (blank($data['password'])) {
            unset($data['password']);
        }

        return $data;
    }

    protected function afterSave(): void
    {
        app(UserService::class)->handleAvatarUpdate(
            $this->record,
            $this->oldAvatar
        );
    }
}
```

---

## Strategi 4 — Cluster untuk Grouping Resource

```php
// Jika ada banyak Resource yang berkaitan, gunakan Cluster
// app/Filament/Clusters/UserManagement.php
namespace App\Filament\Clusters;

use Filament\Clusters\Cluster;

class UserManagement extends Cluster
{
    protected static ?string $navigationIcon = 'heroicon-o-users';
    protected static ?string $navigationLabel = 'Manajemen Pengguna';
    protected static ?int $navigationSort = 2;
}

// Di setiap Resource yang masuk cluster:
class UserResource extends Resource
{
    protected static ?string $cluster = UserManagement::class;
    // ...
}

class RoleResource extends Resource
{
    protected static ?string $cluster = UserManagement::class;
    // ...
}

class PermissionResource extends Resource
{
    protected static ?string $cluster = UserManagement::class;
    // ...
}
```

---

## Strategi 5 — Custom Action Class

```php
// ❌ Action kompleks inline di Resource
Tables\Actions\Action::make('generateReport')
    ->action(function (Order $record): void {
        // 60 baris kode di sini
    }),

// ✅ Action class tersendiri
// app/Filament/Actions/GenerateReportAction.php
namespace App\Filament\Actions;

use App\Models\Order;
use App\Services\ReportService;
use Filament\Tables\Actions\Action;
use Filament\Notifications\Notification;

class GenerateReportAction extends Action
{
    public static function getDefaultName(): ?string
    {
        return 'generateReport';
    }

    protected function setUp(): void
    {
        parent::setUp();

        $this
            ->label('Generate Laporan')
            ->icon('heroicon-o-document-arrow-down')
            ->color('info')
            ->authorize('generate-report')
            ->requiresConfirmation()
            ->modalHeading('Generate Laporan?')
            ->modalDescription('Laporan akan dikirim ke email Anda.')
            ->action(function (Order $record): void {
                app(ReportService::class)->generateAndEmail($record, auth()->user());

                Notification::make()
                    ->title('Laporan sedang diproses')
                    ->body('Akan dikirim ke email dalam beberapa menit.')
                    ->info()
                    ->send();
            });
    }
}

// Di Resource — bersih
->actions([
    GenerateReportAction::make(),
    Tables\Actions\EditAction::make(),
])
```

---

## Struktur Folder Resource yang Direkomendasikan

```
app/Filament/
├── Resources/
│   └── UserResource/
│       ├── Pages/
│       │   ├── ListUsers.php
│       │   ├── CreateUser.php
│       │   ├── EditUser.php
│       │   └── ViewUser.php
│       ├── RelationManagers/
│       │   ├── OrdersRelationManager.php
│       │   └── AddressesRelationManager.php
│       └── Schemas/              ← Tambahkan folder ini jika perlu
│           ├── UserFormSchema.php
│           └── UserTableSchema.php
├── Actions/                      ← Custom action class
│   ├── GenerateReportAction.php
│   └── ApproveOrderAction.php
├── Clusters/                     ← Resource grouping
│   └── UserManagement.php
├── Pages/                        ← Custom pages
│   └── Settings.php
└── Widgets/
    ├── UserStatsWidget.php
    └── UserGrowthChart.php

app/Services/                     ← Business logic
├── UserService.php
├── OrderService.php
└── ReportService.php
```

---

## Format Saran Pemecahan ke User

```
### 📦 Saran Pemecahan: `UserResource.php` (380 baris)

File ini terlalu besar dan melakukan terlalu banyak hal:
1. Form schema 120 baris (personal, address, documents, settings)
2. Table schema 80 baris dengan banyak kolom dan filter
3. Logic update avatar di hook EditUser (40 baris)
4. 4 RelationManager inline

**Struktur yang disarankan:**

UserResource.php                → ~80 baris (orchestrator)
├── Pages/
│   ├── CreateUser.php
│   ├── EditUser.php            → delegate ke UserService untuk file handling
│   ├── ListUsers.php
│   └── ViewUser.php
├── Schemas/
│   ├── UserFormSchema.php      → pecah form per section
│   └── UserTableSchema.php     → columns + filters + actions
└── RelationManagers/
    ├── OrdersRelationManager.php
    └── AddressesRelationManager.php

app/Services/UserService.php    → handleAvatarUpdate, create, update

**Keuntungan:**
- Form schema bisa di-reuse di tempat lain
- Service mudah dites dengan unit test
- Resource jadi mudah dibaca dan dimaintain
```
