# Keamanan & Performa Filament V5

## Keamanan

### 1. Otorisasi — Wajib di Setiap Action

```php
// ❌ Tidak ada otorisasi
Tables\Actions\DeleteAction::make(),
Tables\Actions\Action::make('approve')->action(fn() => ...),

// ✅ Setiap Action wajib otorisasi
Tables\Actions\DeleteAction::make()
    ->authorize('delete'),           // via Policy

Tables\Actions\Action::make('approve')
    ->authorize('approve')           // method di Policy
    ->action(fn (Order $record) => app(OrderService::class)->approve($record)),

// ✅ Atau authorize dengan closure
Tables\Actions\EditAction::make()
    ->authorize(fn (User $record): bool =>
        auth()->user()->can('update', $record)
    ),
```

### 2. Policy — Pola Standar

```php
// app/Policies/UserPolicy.php
namespace App\Policies;

use App\Models\User;
use App\Enums\UserRole;

class UserPolicy
{
    // Filament otomatis memanggil method ini
    public function viewAny(User $auth): bool
    {
        return $auth->hasRole([UserRole::Admin, UserRole::Manager]);
    }

    public function view(User $auth, User $user): bool
    {
        return $auth->hasRole(UserRole::Admin)
            || $auth->id === $user->id
            || $auth->team_id === $user->team_id;
    }

    public function create(User $auth): bool
    {
        return $auth->hasRole(UserRole::Admin);
    }

    public function update(User $auth, User $user): bool
    {
        return $auth->hasRole(UserRole::Admin)
            || ($auth->hasRole(UserRole::Manager) && $auth->team_id === $user->team_id);
    }

    public function delete(User $auth, User $user): bool
    {
        return $auth->hasRole(UserRole::Admin)
            && $auth->id !== $user->id;   // tidak bisa hapus diri sendiri
    }

    // Custom action policy
    public function approve(User $auth, User $user): bool
    {
        return $auth->hasRole(UserRole::Admin);
    }
}
```

### 3. Query Scoping — Wajib untuk Data Isolation

```php
// ✅ Scope query agar user hanya bisa lihat data mereka
public static function getEloquentQuery(): Builder
{
    $query = parent::getEloquentQuery();

    $user = auth()->user();

    // Multi-tenant: scope per team
    if (! $user?->hasRole(UserRole::Admin)) {
        $query->where('team_id', $user?->team_id);
    }

    return $query;
}

// ✅ Untuk Resource dengan soft delete
public static function getEloquentQuery(): Builder
{
    return parent::getEloquentQuery()->withTrashed();
    // Tampilkan trashed hanya di halaman tertentu
}
```

### 4. Mass Assignment Protection

```php
// ✅ Di Model — fillable eksplisit, bukan $guarded = []
protected $fillable = [
    'name', 'email', 'role', 'team_id', 'avatar',
];

// ❌ Terlalu permisif
protected $guarded = [];
```

### 5. Validasi File Upload

```php
FileUpload::make('attachment')
    ->disk('s3')
    ->maxSize(10240)                      // 10MB max
    ->acceptedFileTypes([                 // whitelist mime types
        'image/jpeg',
        'image/png',
        'image/webp',
        'application/pdf',
    ])
    // ❌ Jangan: ->acceptedFileTypes(['*']) atau tanpa maxSize
```

---

## Performa

### 1. Lazy Loading Widget

```php
// ✅ Widget yang berat wajib lazy
class RevenueChartWidget extends ChartWidget
{
    protected static bool $isLazy = true;      // render setelah halaman load

    // Atau per-widget conditional
    public static function canView(): bool
    {
        return auth()->user()?->hasRole('admin') ?? false;
    }
}

class StatsOverview extends BaseWidget
{
    protected static bool $isLazy = true;
    protected int|string|array $columnSpan = 'full';
}
```

### 2. Eager Loading di Table

```php
// ✅ Definisikan relasi yang dibutuhkan di table
public static function table(Table $table): Table
{
    return $table
        ->columns([
            TextColumn::make('team.name'),          // relasi team
            TextColumn::make('manager.name'),       // relasi manager
            ImageColumn::make('avatar')->disk('s3'),
        ])
        // ... Filament otomatis eager load dari kolom relasi
        // Tapi untuk custom getStateUsing, tambahkan manual:
        ->query(
            fn (Builder $query) => $query->with(['team', 'manager', 'roles'])
        );
}

// ✅ Atau override di Resource
public static function getEloquentQuery(): Builder
{
    return parent::getEloquentQuery()->with(['team', 'manager']);
}
```

### 3. Hindari N+1 di getStateUsing

```php
// ❌ N+1 — query baru per baris
TextColumn::make('total_orders')
    ->getStateUsing(fn (User $user) => $user->orders()->count()),

// ✅ Gunakan accessor dengan withCount atau eager load
// Di Model:
public function scopeWithOrderCount(Builder $query): Builder
{
    return $query->withCount('orders');
}

// Di column:
TextColumn::make('orders_count')
    ->label('Total Pesanan')
    ->sortable(),

// Di getEloquentQuery:
public static function getEloquentQuery(): Builder
{
    return parent::getEloquentQuery()->withCount('orders');
}
```

### 4. Pagination & Search

```php
public static function table(Table $table): Table
{
    return $table
        // ✅ Batasi halaman dan default yang wajar
        ->paginated([10, 25, 50, 100])

        // ✅ Search hanya pada kolom yang ada index-nya di DB
        ->columns([
            TextColumn::make('name')->searchable(),       // pastikan ada index
            TextColumn::make('email')->searchable(),      // pastikan ada index
            TextColumn::make('created_at')->sortable(),   // pastikan ada index

            // ❌ Hindari searchable pada kolom JSON atau computed
            // TextColumn::make('meta->full_name')->searchable(),
        ]);
}
```

### 5. Deferred Loading untuk Infolist

```php
// Untuk data yang butuh waktu load
TextEntry::make('stats')
    ->getStateUsing(fn (User $user) => app(UserStatsService::class)->getSummary($user))
    // Pertimbangkan cache atau lazy load
```

### 6. Cache untuk Data yang Jarang Berubah

```php
// Di Widget
protected function getStats(): array
{
    return Cache::remember('dashboard_stats_' . auth()->id(), now()->addMinutes(5), function () {
        return [
            Stat::make('Total', User::count()),
            // ...
        ];
    });
}
```

---

## Checklist Keamanan & Performa

### Keamanan
- [ ] Semua Action (create, edit, delete, custom) punya `->authorize()`
- [ ] Policy terdaftar di `AuthServiceProvider`
- [ ] `getEloquentQuery()` scope data per user/team
- [ ] Model punya `$fillable` eksplisit
- [ ] FileUpload ada `maxSize` dan `acceptedFileTypes`
- [ ] Tidak ada hardcoded credential atau secret di kode

### Performa
- [ ] Widget berat pakai `protected static bool $isLazy = true`
- [ ] Tidak ada N+1 di `getStateUsing()`
- [ ] Relasi yang tampil di tabel sudah di-eager load
- [ ] Search hanya pada kolom yang terindex di database
- [ ] Pagination dibatasi maksimal 100 per halaman
- [ ] Query yang berat dipertimbangkan untuk di-cache
