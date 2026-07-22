# Laravel 13 Modern Style untuk Filament

> Konsultasi juga skill **`laravel-best-practices`** untuk aturan lengkap.

---

## Prinsip Utama: Controller/Resource Harus Tipis

Filament Resource adalah "controller" — logika bisnis tidak boleh di sini.
Gunakan Service class atau Action class untuk operasi yang kompleks.

```php
// ❌ SALAH — logika bisnis di dalam Filament Action
Tables\Actions\Action::make('approve')
    ->action(function (Order $record): void {
        // ❌ logika berat langsung di sini
        $record->status = 'approved';
        $record->approved_at = now();
        $record->approved_by = auth()->id();
        $record->save();

        // kirim email
        Mail::to($record->user)->send(new OrderApprovedMail($record));

        // update inventory
        foreach ($record->items as $item) {
            $item->product->decrement('stock', $item->quantity);
        }

        // log aktivitas
        ActivityLog::create([...]);
    }),

// ✅ BENAR — delegasikan ke Service
Tables\Actions\Action::make('approve')
    ->authorize('approve')
    ->action(function (Order $record, OrderService $service): void {
        $service->approve($record, auth()->user());

        Notification::make()->title('Pesanan disetujui')->success()->send();
    }),
```

---

## Service Class Pattern

```php
// app/Services/OrderService.php
namespace App\Services;

use App\Models\Order;
use App\Models\User;
use App\Enums\OrderStatus;
use App\Mail\OrderApprovedMail;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Mail;

class OrderService
{
    public function approve(Order $order, User $approver): void
    {
        DB::transaction(function () use ($order, $approver) {
            $order->update([
                'status'      => OrderStatus::Approved,
                'approved_at' => now(),
                'approved_by' => $approver->id,
            ]);

            foreach ($order->items as $item) {
                $item->product->decrement('stock', $item->quantity);
            }
        });

        Mail::to($order->user)->queue(new OrderApprovedMail($order));

        activity()
            ->performedOn($order)
            ->causedBy($approver)
            ->log('Order approved');
    }

    public function create(array $data, User $creator): Order
    {
        return DB::transaction(function () use ($data, $creator) {
            $order = Order::create([
                ...$data,
                'created_by' => $creator->id,
                'order_number' => $this->generateOrderNumber(),
            ]);

            // handle items, etc.
            return $order;
        });
    }

    private function generateOrderNumber(): string
    {
        return 'ORD-' . now()->format('Ymd') . '-' . str_pad(
            Order::whereDate('created_at', today())->count() + 1,
            4, '0', STR_PAD_LEFT
        );
    }
}
```

---

## Enum PHP 8.1+ (Wajib untuk Status/Tipe)

```php
// app/Enums/OrderStatus.php
namespace App\Enums;

use Filament\Support\Contracts\HasColor;
use Filament\Support\Contracts\HasLabel;
use Filament\Support\Contracts\HasIcon;

enum OrderStatus: string implements HasColor, HasLabel, HasIcon
{
    case Pending   = 'pending';
    case Approved  = 'approved';
    case Rejected  = 'rejected';
    case Completed = 'completed';
    case Cancelled = 'cancelled';

    public function getLabel(): string
    {
        return match ($this) {
            self::Pending   => 'Menunggu',
            self::Approved  => 'Disetujui',
            self::Rejected  => 'Ditolak',
            self::Completed => 'Selesai',
            self::Cancelled => 'Dibatalkan',
        };
    }

    public function getColor(): string
    {
        return match ($this) {
            self::Pending   => 'warning',
            self::Approved  => 'success',
            self::Rejected  => 'danger',
            self::Completed => 'info',
            self::Cancelled => 'gray',
        };
    }

    public function getIcon(): string
    {
        return match ($this) {
            self::Pending   => 'heroicon-o-clock',
            self::Approved  => 'heroicon-o-check-circle',
            self::Rejected  => 'heroicon-o-x-circle',
            self::Completed => 'heroicon-o-check-badge',
            self::Cancelled => 'heroicon-o-no-symbol',
        };
    }

    // Helper methods
    public function canTransitionTo(self $newStatus): bool
    {
        return match ($this) {
            self::Pending   => in_array($newStatus, [self::Approved, self::Rejected, self::Cancelled]),
            self::Approved  => in_array($newStatus, [self::Completed, self::Cancelled]),
            default         => false,
        };
    }
}

// Penggunaan di Model cast
protected $casts = [
    'status' => OrderStatus::class,
];
```

---

## Form Request (Validasi di Luar Filament)

```php
// Untuk endpoint API atau controller di luar Filament
// app/Http/Requests/StoreUserRequest.php
namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Validation\Rule;

class StoreUserRequest extends FormRequest
{
    public function authorize(): bool
    {
        return $this->user()->can('create', User::class);
    }

    public function rules(): array
    {
        return [
            'name'     => ['required', 'string', 'max:255'],
            'email'    => ['required', 'email', 'unique:users,email'],
            'role'     => ['required', Rule::enum(UserRole::class)],
            'password' => ['required', 'min:8', 'confirmed'],
        ];
    }

    public function messages(): array
    {
        return [
            'email.unique' => 'Email sudah terdaftar.',
            'role.enum'    => 'Peran tidak valid.',
        ];
    }
}
```

---

## Model — Konvensi Modern

```php
// app/Models/User.php
namespace App\Models;

use App\Enums\UserRole;
use App\Enums\UserStatus;
use Illuminate\Database\Eloquent\Casts\Attribute;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\SoftDeletes;
use Illuminate\Database\Eloquent\Relations\HasMany;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class User extends Authenticatable
{
    use HasFactory, SoftDeletes;

    // ✅ Fillable eksplisit — jangan $guarded = []
    protected $fillable = [
        'name', 'email', 'password', 'role',
        'status', 'avatar', 'team_id',
    ];

    protected $hidden = ['password', 'remember_token'];

    // ✅ Gunakan cast dengan Enum
    protected function casts(): array
    {
        return [
            'email_verified_at' => 'datetime',
            'password'          => 'hashed',
            'role'              => UserRole::class,
            'status'            => UserStatus::class,
        ];
    }

    // ✅ Accessor modern (PHP 8+)
    protected function avatarUrl(): Attribute
    {
        return Attribute::make(
            get: fn () => $this->avatar
                ? Storage::disk('s3')->temporaryUrl($this->avatar, now()->addMinutes(30))
                : null,
        );
    }

    // ✅ Relasi eksplisit dengan return type
    public function team(): BelongsTo
    {
        return $this->belongsTo(Team::class);
    }

    public function orders(): HasMany
    {
        return $this->hasMany(Order::class);
    }

    // ✅ Scope query yang dapat dichain
    public function scopeActive(Builder $query): Builder
    {
        return $query->where('status', UserStatus::Active);
    }

    public function scopeByRole(Builder $query, UserRole $role): Builder
    {
        return $query->where('role', $role);
    }
}
```

---

## Observer — Untuk Side Effect Model

```php
// app/Observers/UserObserver.php
namespace App\Observers;

use App\Models\User;
use App\Services\FileService;
use Illuminate\Support\Facades\Storage;

class UserObserver
{
    public function __construct(
        private readonly FileService $fileService,
    ) {}

    // Cleanup file S3 saat record dihapus
    public function deleted(User $user): void
    {
        if ($user->avatar) {
            // Hapus dari S3 setelah record terhapus
            Storage::disk('s3')->delete($user->avatar);
        }
    }

    // Jika soft delete, hapus file saat force delete
    public function forceDeleted(User $user): void
    {
        if ($user->avatar) {
            Storage::disk('s3')->delete($user->avatar);
        }
    }
}

// Daftarkan di AppServiceProvider atau EventServiceProvider
User::observe(UserObserver::class);
```

---

## Route Model Binding

```php
// ✅ Gunakan route model binding — bukan find() manual
Route::get('/users/{user}', [UserController::class, 'show']);

// Di controller
public function show(User $user): Response   // otomatis inject & 404 jika tidak ada
{
    return response()->json($user->load('orders'));
}

// ❌ Cara lama
public function show(int $id): Response
{
    $user = User::findOrFail($id);  // tidak perlu lagi
    return response()->json($user);
}

// Untuk custom binding key
// Di Model:
public function getRouteKeyName(): string
{
    return 'slug';   // default adalah 'id'
}
```

---

## Query Scoping di Filament Resource

```php
// ✅ Selalu scope query berdasarkan konteks user/tenant
public static function getEloquentQuery(): Builder
{
    $query = parent::getEloquentQuery();

    // Multi-tenant: scope per team
    if (auth()->user()?->team_id) {
        $query->where('team_id', auth()->user()->team_id);
    }

    // Soft delete: tampilkan semua termasuk yang terhapus untuk admin
    if (auth()->user()?->hasRole('admin')) {
        $query->withTrashed();
    }

    return $query;
}
```

---

## Readonly DTO untuk Transfer Data

```php
// app/Data/CreateUserData.php
namespace App\Data;

use App\Enums\UserRole;

final readonly class CreateUserData
{
    public function __construct(
        public string   $name,
        public string   $email,
        public string   $password,
        public UserRole $role,
        public ?string  $avatar = null,
        public ?int     $teamId = null,
    ) {}

    // Dari array (misal dari Filament form state)
    public static function fromArray(array $data): self
    {
        return new self(
            name:     $data['name'],
            email:    $data['email'],
            password: $data['password'],
            role:     UserRole::from($data['role']),
            avatar:   $data['avatar'] ?? null,
            teamId:   $data['team_id'] ?? null,
        );
    }
}

// Di Service
public function create(CreateUserData $data): User
{
    return User::create([
        'name'     => $data->name,
        'email'    => $data->email,
        'password' => Hash::make($data->password),
        'role'     => $data->role,
        'avatar'   => $data->avatar,
        'team_id'  => $data->teamId,
    ]);
}
```
