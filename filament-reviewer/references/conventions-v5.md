# Konvensi Filament V5 — Referensi Lengkap

## Struktur Resource yang Benar

```php
// app/Filament/Resources/UserResource.php

namespace App\Filament\Resources;

use App\Filament\Resources\UserResource\Pages;
use App\Filament\Resources\UserResource\RelationManagers;
use App\Models\User;
use Filament\Forms;
use Filament\Forms\Form;
use Filament\Resources\Resource;
use Filament\Tables;
use Filament\Tables\Table;
use Illuminate\Database\Eloquent\Builder;

class UserResource extends Resource
{
    // 1. Model binding
    protected static ?string $model = User::class;

    // 2. Navigasi
    protected static ?string $navigationIcon = 'heroicon-o-users';
    protected static ?string $activeNavigationIcon = 'heroicon-s-users';
    protected static ?string $navigationGroup = 'Manajemen Pengguna';
    protected static ?int $navigationSort = 1;
    protected static ?string $navigationLabel = 'Pengguna';
    protected static ?string $modelLabel = 'Pengguna';
    protected static ?string $pluralModelLabel = 'Daftar Pengguna';

    // 3. Global search
    protected static ?string $recordTitleAttribute = 'name';

    // 4. Scope query — WAJIB untuk multi-tenant atau data scoping
    public static function getEloquentQuery(): Builder
    {
        return parent::getEloquentQuery()
            ->withoutGlobalScopes()   // jika perlu bypass global scope
            ->where('team_id', auth()->user()?->team_id);
    }

    // 5. Form schema
    public static function form(Form $form): Form
    {
        return $form->schema([...]);
    }

    // 6. Table schema
    public static function table(Table $table): Table
    {
        return $table
            ->columns([...])
            ->filters([...])
            ->actions([...])
            ->bulkActions([...]);
    }

    // 7. Relation managers
    public static function getRelations(): array
    {
        return [
            RelationManagers\OrdersRelationManager::class,
        ];
    }

    // 8. Pages
    public static function getPages(): array
    {
        return [
            'index'  => Pages\ListUsers::route('/'),
            'create' => Pages\CreateUser::route('/create'),
            'view'   => Pages\ViewUser::route('/{record}'),
            'edit'   => Pages\EditUser::route('/{record}/edit'),
        ];
    }

    // 9. Global search result detail
    public static function getGlobalSearchResultDetails(Model $record): array
    {
        return [
            'Email' => $record->email,
            'Tim'   => $record->team?->name,
        ];
    }
}
```

---

## Form Schema — Konvensi

```php
public static function form(Form $form): Form
{
    return $form
        ->schema([
            // Gunakan Section untuk grouping field
            Forms\Components\Section::make('Informasi Dasar')
                ->description('Data utama pengguna')
                ->icon('heroicon-o-user')
                ->schema([
                    Forms\Components\TextInput::make('name')
                        ->label('Nama Lengkap')
                        ->required()
                        ->maxLength(255)
                        ->autofocus(),

                    Forms\Components\TextInput::make('email')
                        ->label('Email')
                        ->email()
                        ->required()
                        ->unique(ignoreRecord: true)
                        ->maxLength(255),
                ])
                ->columns(2),

            Forms\Components\Section::make('Pengaturan')
                ->schema([
                    Forms\Components\Select::make('role')
                        ->label('Peran')
                        ->options(UserRole::class)   // gunakan Enum
                        ->required()
                        ->native(false),             // gunakan custom select V5

                    Forms\Components\Toggle::make('is_active')
                        ->label('Aktif')
                        ->default(true),
                ])
                ->columns(2),
        ]);
}
```

---

## Table Schema — Konvensi

```php
public static function table(Table $table): Table
{
    return $table
        ->columns([
            Tables\Columns\TextColumn::make('name')
                ->label('Nama')
                ->searchable()
                ->sortable()
                ->weight(FontWeight::Medium),

            Tables\Columns\TextColumn::make('email')
                ->searchable()
                ->copyable()
                ->copyMessage('Email disalin'),

            Tables\Columns\TextColumn::make('role')
                ->badge()
                ->color(fn (UserRole $state): string => $state->color()),

            Tables\Columns\IconColumn::make('is_active')
                ->boolean()
                ->label('Aktif'),

            Tables\Columns\ImageColumn::make('avatar')
                ->circular()
                ->disk('s3'),

            Tables\Columns\TextColumn::make('created_at')
                ->dateTime('d M Y, H:i')
                ->sortable()
                ->toggleable(isToggledHiddenByDefault: true),
        ])
        ->filters([
            Tables\Filters\SelectFilter::make('role')
                ->options(UserRole::class),

            Tables\Filters\TernaryFilter::make('is_active')
                ->label('Status')
                ->trueLabel('Aktif')
                ->falseLabel('Nonaktif'),

            Tables\Filters\Filter::make('created_at')
                ->form([
                    Forms\Components\DatePicker::make('from')->label('Dari'),
                    Forms\Components\DatePicker::make('until')->label('Sampai'),
                ])
                ->query(fn (Builder $query, array $data) => $query
                    ->when($data['from'], fn ($q) => $q->whereDate('created_at', '>=', $data['from']))
                    ->when($data['until'], fn ($q) => $q->whereDate('created_at', '<=', $data['until']))
                ),
        ])
        ->actions([
            Tables\Actions\ViewAction::make(),
            Tables\Actions\EditAction::make(),
            Tables\Actions\DeleteAction::make()
                ->requiresConfirmation()
                ->authorize('delete'),   // V5: selalu ada otorisasi
        ])
        ->bulkActions([
            Tables\Actions\BulkActionGroup::make([
                Tables\Actions\DeleteBulkAction::make()
                    ->requiresConfirmation(),
            ]),
        ])
        ->defaultSort('created_at', 'desc')
        ->striped()
        ->paginated([10, 25, 50]);
}
```

---

## Actions — Konvensi

```php
// Action dengan modal form
Tables\Actions\Action::make('changeStatus')
    ->label('Ubah Status')
    ->icon('heroicon-o-arrow-path')
    ->color(Color::Warning)
    ->authorize('update')                    // wajib
    ->requiresConfirmation()
    ->form([
        Forms\Components\Select::make('status')
            ->options(UserStatus::class)
            ->required(),
        Forms\Components\Textarea::make('reason')
            ->label('Alasan')
            ->required(),
    ])
    ->action(function (Model $record, array $data, UserService $service): void {
        // ✅ Delegasikan ke Service — jangan logika berat di sini
        $service->changeStatus($record, $data['status'], $data['reason']);

        Notification::make()
            ->title('Status berhasil diubah')
            ->success()
            ->send();
    }),

// Header action di List page
protected function getHeaderActions(): array
{
    return [
        Actions\CreateAction::make()
            ->authorize('create'),
        Actions\Action::make('export')
            ->label('Export Excel')
            ->icon('heroicon-o-arrow-down-tray')
            ->action(fn () => $this->exportData()),
    ];
}
```

---

## Enum untuk Status/Tipe (PHP 8.1+)

```php
// app/Enums/UserRole.php
namespace App\Enums;

use Filament\Support\Contracts\HasColor;
use Filament\Support\Contracts\HasIcon;
use Filament\Support\Contracts\HasLabel;

enum UserRole: string implements HasColor, HasIcon, HasLabel
{
    case Admin    = 'admin';
    case Manager  = 'manager';
    case Staff    = 'staff';
    case Viewer   = 'viewer';

    public function getLabel(): ?string
    {
        return match ($this) {
            self::Admin   => 'Administrator',
            self::Manager => 'Manajer',
            self::Staff   => 'Staf',
            self::Viewer  => 'Viewer',
        };
    }

    public function getColor(): string|array|null
    {
        return match ($this) {
            self::Admin   => 'danger',
            self::Manager => 'warning',
            self::Staff   => 'success',
            self::Viewer  => 'gray',
        };
    }

    public function getIcon(): ?string
    {
        return match ($this) {
            self::Admin   => 'heroicon-o-shield-check',
            self::Manager => 'heroicon-o-briefcase',
            self::Staff   => 'heroicon-o-user',
            self::Viewer  => 'heroicon-o-eye',
        };
    }
}
```

---

## RelationManager — Konvensi

```php
// app/Filament/Resources/UserResource/RelationManagers/OrdersRelationManager.php

namespace App\Filament\Resources\UserResource\RelationManagers;

use Filament\Forms;
use Filament\Forms\Form;
use Filament\Resources\RelationManagers\RelationManager;
use Filament\Tables;
use Filament\Tables\Table;

class OrdersRelationManager extends RelationManager
{
    protected static string $relationship = 'orders';
    protected static ?string $title = 'Pesanan';

    public function form(Form $form): Form
    {
        return $form->schema([
            // form fields
        ]);
    }

    public function table(Table $table): Table
    {
        return $table
            ->recordTitleAttribute('order_number')
            ->columns([
                Tables\Columns\TextColumn::make('order_number'),
                Tables\Columns\TextColumn::make('total')->money('IDR'),
            ])
            ->headerActions([
                Tables\Actions\CreateAction::make()
                    ->authorize('create'),
            ])
            ->actions([
                Tables\Actions\EditAction::make(),
                Tables\Actions\DeleteAction::make(),
            ]);
    }
}
```

---

## Widget — Konvensi

```php
// Stats Overview Widget
class UserStatsWidget extends BaseWidget
{
    protected static ?int $sort = 1;
    protected int|string|array $columnSpan = 'full';

    protected function getStats(): array
    {
        return [
            Stat::make('Total Pengguna', User::count())
                ->description('Semua pengguna terdaftar')
                ->descriptionIcon('heroicon-m-users')
                ->color('success')
                ->chart(User::selectRaw('count(*) as total')
                    ->groupBy('DATE(created_at)')
                    ->latest('created_at')
                    ->take(7)
                    ->pluck('total')
                    ->toArray()
                ),
        ];
    }

    // Widget berat wajib pakai deferLoading
    protected static bool $isLazy = true;
}

// Chart Widget
class UserGrowthChart extends ChartWidget
{
    protected static ?string $heading = 'Pertumbuhan Pengguna';
    protected static bool $isLazy = true;   // wajib untuk chart

    protected function getData(): array
    {
        $data = User::selectRaw('DATE(created_at) as date, count(*) as total')
            ->groupBy('date')
            ->latest('date')
            ->take(30)
            ->get();

        return [
            'datasets' => [[
                'label' => 'Pengguna Baru',
                'data'  => $data->pluck('total')->toArray(),
            ]],
            'labels' => $data->pluck('date')->toArray(),
        ];
    }

    protected function getType(): string
    {
        return 'line';
    }
}
```

---

## Custom Page — Konvensi

```php
namespace App\Filament\Pages;

use Filament\Pages\Page;
use Filament\Forms\Contracts\HasForms;
use Filament\Forms\Concerns\InteractsWithForms;

class Settings extends Page implements HasForms
{
    use InteractsWithForms;

    protected static ?string $navigationIcon = 'heroicon-o-cog-6-tooth';
    protected static ?string $navigationGroup = 'Pengaturan';
    protected static ?string $title = 'Pengaturan Sistem';
    protected static string $view = 'filament.pages.settings';

    // Authorize akses ke page
    public static function canAccess(): bool
    {
        return auth()->user()?->hasRole('admin') ?? false;
    }

    // Form state
    public ?array $data = [];

    public function mount(): void
    {
        $settings = app(GeneralSettings::class);
        $this->form->fill($settings->toArray());
    }

    public function form(Form $form): Form
    {
        return $form
            ->schema([...])
            ->statePath('data');
    }

    public function save(SettingsService $service): void
    {
        $data = $this->form->getState();
        $service->save($data);

        Notification::make()->title('Pengaturan disimpan')->success()->send();
    }
}
```

---

## Panel Provider — Konvensi

```php
// app/Providers/Filament/AdminPanelProvider.php
public function panel(Panel $panel): Panel
{
    return $panel
        ->default()
        ->id('admin')
        ->path('admin')
        ->login()
        ->colors(['primary' => Color::Indigo])
        ->font('Inter')
        ->discoverResources(in: app_path('Filament/Resources'), for: 'App\\Filament\\Resources')
        ->discoverPages(in: app_path('Filament/Pages'), for: 'App\\Filament\\Pages')
        ->discoverWidgets(in: app_path('Filament/Widgets'), for: 'App\\Filament\\Widgets')
        ->discoverClusters(in: app_path('Filament/Clusters'), for: 'App\\Filament\\Clusters')
        ->middleware([
            EncryptCookies::class,
            AddQueuedCookiesToResponse::class,
            StartSession::class,
            AuthenticateSession::class,
            ShareErrorsFromSession::class,
            VerifyCsrfToken::class,
            SubstituteBindings::class,
            DisableBladeIconComponents::class,
            DispatchServingFilamentEvent::class,
        ])
        ->authMiddleware([Authenticate::class])
        ->databaseNotifications()           // aktifkan notifikasi database
        ->databaseNotificationsPolling('30s');
}
```
