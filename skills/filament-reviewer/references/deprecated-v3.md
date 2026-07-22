# Filament V3 → V5: API Deprecated & Pengganti

## Form Components

### TextInput
```php
// ❌ V3
TextInput::make('name')
    ->disableLabel()
    ->mask(fn (Mask $mask) => $mask->money('Rp', ',', 2))
    ->disablePlaceholderSelection()
    ->lazy()

// ✅ V5
TextInput::make('name')
    ->hiddenLabel()
    ->mask(RawJs::make(<<<'JS'
        $money($input, ',', 'Rp ', 2)
    JS))
    ->selectablePlaceholder(false)
    ->live(onBlur: true)
```

### FileUpload
```php
// ❌ V3 — disk lokal, nama file tidak aman
FileUpload::make('attachment')
    ->disk('public')
    ->directory('attachments')
    ->preserveFilenames()

// ✅ V5 — S3, FileHelper, urutan hapus yang benar
FileUpload::make('attachment')
    ->disk('s3')
    ->directory('attachments')
    ->visibility('private')
    ->getUploadedFileNameForStorageUsing(
        fn (TemporaryUploadedFile $file): string => FileHelper::generateName($file)
    )
```

### Select
```php
// ❌ V3
Select::make('status')
    ->disablePlaceholderSelection()
    ->isSearchable()

// ✅ V5
Select::make('status')
    ->selectablePlaceholder(false)
    ->searchable()
```

### Repeater
```php
// ❌ V3
Repeater::make('items')
    ->createItemButtonLabel('Tambah Item')
    ->disableItemMovement()

// ✅ V5
Repeater::make('items')
    ->addActionLabel('Tambah Item')
    ->reorderable(false)
```

### RichEditor / MarkdownEditor
```php
// ❌ V3 — tidak ada image upload ke S3
RichEditor::make('content')
    ->fileAttachmentsDisk('public')

// ✅ V5 — wajib S3
RichEditor::make('content')
    ->fileAttachmentsDisk('s3')
    ->fileAttachmentsDirectory('content/attachments')
    ->fileAttachmentsVisibility('private')
```

### DateTimePicker
```php
// ❌ V3
DateTimePicker::make('published_at')
    ->withoutTime()
    ->withoutSeconds()

// ✅ V5
DatePicker::make('published_at')        // pakai DatePicker jika memang hanya tanggal

DateTimePicker::make('published_at')
    ->seconds(false)
```

---

## Table Columns

### BadgeColumn — Dihapus
```php
// ❌ V3 — BadgeColumn sudah dihapus
BadgeColumn::make('status')
    ->colors([
        'success' => 'active',
        'danger' => 'inactive',
    ])

// ✅ V5
TextColumn::make('status')
    ->badge()
    ->color(fn (string $state): string => match ($state) {
        'active' => 'success',
        'inactive' => 'danger',
        default => 'gray',
    })
```

### TagsColumn — Dihapus
```php
// ❌ V3
TagsColumn::make('tags')

// ✅ V5
TextColumn::make('tags')
    ->badge()
    ->separator(',')
```

### BooleanColumn — Dihapus
```php
// ❌ V3
BooleanColumn::make('is_active')

// ✅ V5
IconColumn::make('is_active')
    ->boolean()
```

### ImageColumn
```php
// ❌ V3
ImageColumn::make('avatar')
    ->rounded()
    ->size(40)

// ✅ V5
ImageColumn::make('avatar')
    ->circular()
    ->width(40)
    ->height(40)
    ->disk('s3')      // wajib jika foto di S3
```

### TextColumn
```php
// ❌ V3
TextColumn::make('price')
    ->money('idr', true)
    ->formatStateUsing(fn ($state) => 'Rp ' . number_format($state))

// ✅ V5
TextColumn::make('price')
    ->money('IDR')
    ->numeric(decimalPlaces: 0)
```

---

## Table Actions & Bulk Actions

```php
// ❌ V3 — cara lama
Tables\Actions\EditAction::make()
    ->mountUsing(fn (Form $form, Model $record) => $form->fill([...]))

// ✅ V5 — gunakan fillForm
Tables\Actions\EditAction::make()
    ->fillForm(fn (Model $record): array => [
        'name' => $record->name,
    ])

// ❌ V3 — warna string
Action::make('approve')
    ->color('success')
    ->requiresConfirmation()

// ✅ V5 — masih bisa string, tapi Enum lebih baik
Action::make('approve')
    ->color(Color::Success)
    ->requiresConfirmation()
    ->authorize('approve')     // V5: wajib ada otorisasi
```

---

## Page & Navigation

```php
// ❌ V3 — NavigationItem manual yang verbose
NavigationItem::make('Dashboard')
    ->url(route('filament.admin.pages.dashboard'))
    ->icon('heroicon-o-home')
    ->activeIcon('heroicon-s-home')
    ->isActiveWhen(fn () => request()->routeIs('filament.admin.pages.dashboard'))

// ✅ V5 — definisi di Resource/Page class
protected static ?string $navigationIcon = 'heroicon-o-home';
protected static ?string $activeNavigationIcon = 'heroicon-s-home';
// Otomatis active detection

// ❌ V3 — Panel konfigurasi di AppServiceProvider
Filament::serving(function () {
    Filament::registerPages([...]);
    Filament::registerWidgets([...]);
});

// ✅ V5 — semua di PanelProvider
public function panel(Panel $panel): Panel
{
    return $panel
        ->discoverResources(in: app_path('Filament/Resources'), for: 'App\\Filament\\Resources')
        ->discoverPages(in: app_path('Filament/Pages'), for: 'App\\Filament\\Pages')
        ->discoverWidgets(in: app_path('Filament/Widgets'), for: 'App\\Filament\\Widgets');
}
```

---

## Auth & Policy

```php
// ❌ V3 — override method di Resource
public static function canCreate(): bool
{
    return auth()->user()->hasRole('admin');
}

// ✅ V5 — gunakan Policy (lebih testable)
// UserPolicy.php
public function create(User $user): bool
{
    return $user->hasRole('admin');
}

// Resource.php — daftarkan policy
protected static string $model = User::class;
// Policy otomatis terbinding jika naming convention benar
// UserResource → User model → UserPolicy

// Atau eksplisit:
public static function getEloquentQuery(): Builder
{
    return parent::getEloquentQuery()
        ->where('team_id', auth()->user()->team_id);  // scope per user
}
```

---

## Infolist — Pengganti Form Read-Only

```php
// ❌ V3 — form dipakai untuk view-only
TextInput::make('name')->disabled()->dehydrated(false)
TextInput::make('email')->disabled()->dehydrated(false)

// ✅ V5 — gunakan Infolist
use Filament\Infolists\Components\TextEntry;
use Filament\Infolists\Components\ImageEntry;
use Filament\Infolists\Components\Section;
use Filament\Infolists\Infolist;

public function infolist(Infolist $infolist): Infolist
{
    return $infolist
        ->schema([
            Section::make('Informasi Pengguna')
                ->schema([
                    TextEntry::make('name')->label('Nama'),
                    TextEntry::make('email'),
                    TextEntry::make('status')
                        ->badge()
                        ->color(fn (string $state) => match ($state) {
                            'active' => 'success',
                            default => 'gray',
                        }),
                    ImageEntry::make('avatar')
                        ->circular()
                        ->disk('s3'),
                ])->columns(2),
        ]);
}
```

---

## Traits & Concerns

```php
// ❌ V3 — import manual trait yang sudah digabung
use Filament\Forms\Concerns\InteractsWithForms;
use Filament\Tables\Concerns\InteractsWithTable;

class ListUsers extends Page
{
    use InteractsWithForms;
    use InteractsWithTable;
}

// ✅ V5 — gunakan HasForms / HasTable concern, atau extend class yang tepat
class ListUsers extends ListRecords  // sudah include semua yang perlu
{
    protected static string $resource = UserResource::class;
}

// Untuk custom page dengan form:
class CustomPage extends Page implements HasForms
{
    use InteractsWithForms;  // masih perlu jika implement manual
}
```

---

## Notifications

```php
// ❌ V3
$this->notify('success', 'Data berhasil disimpan');

// ✅ V5
use Filament\Notifications\Notification;

Notification::make()
    ->title('Berhasil')
    ->body('Data berhasil disimpan.')
    ->success()
    ->send();

// Notifikasi ke user lain
Notification::make()
    ->title('Pesanan baru masuk')
    ->warning()
    ->sendToDatabase($targetUser);
```
