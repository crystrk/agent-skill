# Upload File: S3 + FileHelper — Aturan Wajib

## Prinsip Utama

1. **Semua file wajib di S3** — tidak ada disk `public` atau `local`
2. **Penamaan file wajib via `FileHelper`** — tidak pakai nama asli dari user
3. **Urutan update file: upload baru → verifikasi → hapus lama** — jangan terbalik
4. **File lama tidak dihapus otomatis oleh Filament** — kelola via Observer atau Service

---

## FileHelper — Implementasi Standar

```php
// app/Helpers/FileHelper.php
namespace App\Helpers;

use Livewire\Features\SupportFileUploads\TemporaryUploadedFile;
use Illuminate\Http\UploadedFile;
use Illuminate\Support\Str;

class FileHelper
{
    /**
     * Generate nama file yang aman untuk disimpan di S3.
     * Format: {timestamp}_{random16char}.{ext}
     * Contoh: 1720425600_a3f8b2c91d4e7f0g.jpg
     */
    public static function generateName(
        TemporaryUploadedFile|UploadedFile $file
    ): string {
        $extension = strtolower($file->getClientOriginalExtension());
        $extension = self::sanitizeExtension($extension);

        return sprintf(
            '%d_%s.%s',
            now()->timestamp,
            Str::random(16),
            $extension
        );
    }

    /**
     * Generate nama dari string arbitrary (misal nama model).
     */
    public static function generateNameFromString(string $context, string $extension): string
    {
        $extension = self::sanitizeExtension(strtolower($extension));
        $slug      = Str::slug($context);

        return sprintf('%s_%d_%s.%s', $slug, now()->timestamp, Str::random(8), $extension);
    }

    /**
     * Whitelist ekstensi yang diizinkan.
     */
    private static function sanitizeExtension(string $ext): string
    {
        $allowed = ['jpg', 'jpeg', 'png', 'gif', 'webp', 'pdf', 'doc', 'docx',
                    'xls', 'xlsx', 'csv', 'zip', 'mp4', 'mp3', 'svg'];

        return in_array($ext, $allowed) ? $ext : 'bin';
    }

    /**
     * Hapus file dari S3 dengan aman (tidak throw jika tidak ada).
     */
    public static function deleteFromS3(?string $path): void
    {
        if (! $path) return;

        try {
            Storage::disk('s3')->delete($path);
        } catch (\Throwable $e) {
            // Log tapi jangan throw — file mungkin memang sudah tidak ada
            logger()->warning("Failed to delete S3 file: {$path}", ['error' => $e->getMessage()]);
        }
    }

    /**
     * Verifikasi file ada di S3.
     */
    public static function existsOnS3(string $path): bool
    {
        return Storage::disk('s3')->exists($path);
    }

    /**
     * Dapatkan temporary URL (untuk file private).
     */
    public static function temporaryUrl(string $path, int $minutes = 30): string
    {
        return Storage::disk('s3')->temporaryUrl($path, now()->addMinutes($minutes));
    }

    /**
     * Dapatkan public URL (untuk file public).
     */
    public static function publicUrl(string $path): string
    {
        return Storage::disk('s3')->url($path);
    }
}
```

---

## FileUpload di Filament Form — Pola Wajib

```php
use App\Helpers\FileHelper;
use Livewire\Features\SupportFileUploads\TemporaryUploadedFile;

// ✅ Single file upload — wajib
FileUpload::make('avatar')
    ->label('Foto Profil')
    ->disk('s3')
    ->directory('users/avatars')
    ->visibility('private')                  // default private
    ->image()
    ->imageEditor()
    ->maxSize(2048)                          // 2MB
    ->acceptedFileTypes(['image/jpeg', 'image/png', 'image/webp'])
    ->getUploadedFileNameForStorageUsing(
        fn (TemporaryUploadedFile $file): string => FileHelper::generateName($file)
    )
    ->deleteUploadedFileUsing(function (string $file): void {
        // Kosongkan — penghapusan dikelola oleh Observer/Service
        // agar urutan upload-baru-dulu-hapus-lama terjamin
    }),

// ✅ Multiple file upload
FileUpload::make('attachments')
    ->label('Lampiran')
    ->disk('s3')
    ->directory('documents/attachments')
    ->visibility('private')
    ->multiple()
    ->maxFiles(5)
    ->maxSize(10240)                         // 10MB per file
    ->acceptedFileTypes(['application/pdf', 'image/*'])
    ->getUploadedFileNameForStorageUsing(
        fn (TemporaryUploadedFile $file): string => FileHelper::generateName($file)
    )
    ->reorderable()
    ->appendFiles(),                         // V5: tambah file tanpa replace semua

// ✅ File publik (misal thumbnail produk)
FileUpload::make('thumbnail')
    ->disk('s3')
    ->directory('products/thumbnails')
    ->visibility('public')                   // file bisa diakses publik
    ->image()
    ->imageResizeMode('cover')
    ->imageCropAspectRatio('16:9')
    ->imageResizeTargetWidth('1280')
    ->imageResizeTargetHeight('720')
    ->getUploadedFileNameForStorageUsing(
        fn (TemporaryUploadedFile $file): string => FileHelper::generateName($file)
    ),
```

---

## Urutan Update File — Wajib Ikuti

```php
// app/Services/UserService.php

use App\Helpers\FileHelper;
use Illuminate\Support\Facades\Storage;
use Illuminate\Support\Facades\DB;

class UserService
{
    /**
     * Update profil user dengan penggantian avatar yang aman.
     *
     * URUTAN WAJIB:
     * 1. Simpan path file lama
     * 2. Update record dengan file baru
     * 3. Verifikasi file baru ada di S3
     * 4. Hapus file lama HANYA jika file baru sudah terverifikasi
     */
    public function updateAvatar(User $user, ?string $newAvatarPath): void
    {
        $oldAvatarPath = $user->avatar;

        // Jika tidak ada perubahan file, lewati
        if ($oldAvatarPath === $newAvatarPath) {
            return;
        }

        DB::transaction(function () use ($user, $newAvatarPath, $oldAvatarPath) {
            // Step 1: Update record dengan path baru
            $user->update(['avatar' => $newAvatarPath]);

            // Step 2: Verifikasi file baru sudah ada di S3
            if ($newAvatarPath && ! FileHelper::existsOnS3($newAvatarPath)) {
                throw new \RuntimeException(
                    "File baru tidak ditemukan di S3: {$newAvatarPath}"
                );
            }

            // Step 3: Hapus file lama SETELAH verifikasi sukses
            if ($oldAvatarPath && $oldAvatarPath !== $newAvatarPath) {
                FileHelper::deleteFromS3($oldAvatarPath);
            }
        });
    }

    /**
     * Pola untuk multiple files.
     */
    public function updateAttachments(Document $document, array $newPaths): void
    {
        $oldPaths = $document->attachments ?? [];

        DB::transaction(function () use ($document, $newPaths, $oldPaths) {
            // Update record
            $document->update(['attachments' => $newPaths]);

            // Verifikasi semua file baru ada di S3
            foreach ($newPaths as $path) {
                if (! FileHelper::existsOnS3($path)) {
                    throw new \RuntimeException("File tidak ditemukan di S3: {$path}");
                }
            }

            // Hapus file lama yang tidak ada di list baru
            $filesToDelete = array_diff($oldPaths, $newPaths);
            foreach ($filesToDelete as $oldPath) {
                FileHelper::deleteFromS3($oldPath);
            }
        });
    }
}
```

---

## Hook Filament untuk File Update

```php
// Di Resource Page (CreateUser, EditUser)
class EditUser extends EditRecord
{
    protected static string $resource = UserResource::class;

    // ✅ Hook sebelum save — simpan path lama
    protected function mutateFormDataBeforeSave(array $data): array
    {
        // Simpan path lama di property sementara
        $this->oldAvatar = $this->record->avatar;
        return $data;
    }

    // ✅ Hook setelah save — hapus file lama jika berhasil
    protected function afterSave(): void
    {
        $newAvatar = $this->record->avatar;

        if (isset($this->oldAvatar) && $this->oldAvatar !== $newAvatar) {
            // File baru sudah tersimpan di DB → aman hapus yang lama
            FileHelper::deleteFromS3($this->oldAvatar);
        }
    }
}

class CreateUser extends CreateRecord
{
    protected static string $resource = UserResource::class;

    // Tidak ada file lama saat create — tidak perlu hook khusus
}
```

---

## Observer untuk Auto-Cleanup

```php
// app/Observers/UserObserver.php
namespace App\Observers;

use App\Models\User;
use App\Helpers\FileHelper;

class UserObserver
{
    // Saat record di-force delete (atau delete permanent)
    public function forceDeleted(User $user): void
    {
        if ($user->avatar) {
            FileHelper::deleteFromS3($user->avatar);
        }

        // Hapus multiple files
        foreach ($user->attachments ?? [] as $path) {
            FileHelper::deleteFromS3($path);
        }
    }

    // Jika tidak pakai soft delete
    public function deleted(User $user): void
    {
        FileHelper::deleteFromS3($user->avatar);
    }
}

// Daftar di AppServiceProvider
public function boot(): void
{
    User::observe(UserObserver::class);
}
```

---

## ImageColumn — Tampilkan dari S3

```php
// Di Table column
Tables\Columns\ImageColumn::make('avatar')
    ->disk('s3')
    ->circular()
    ->width(40)
    ->height(40)
    ->defaultImageUrl(asset('images/default-avatar.png')),

// Untuk file private — gunakan temporary URL via accessor di model
// Di Model:
protected function avatarUrl(): Attribute
{
    return Attribute::make(
        get: fn () => $this->avatar
            ? FileHelper::temporaryUrl($this->avatar, 60)
            : null,
    );
}

// Di column — gunakan accessor
Tables\Columns\ImageColumn::make('avatar_url')
    ->label('Foto')
    ->circular(),
```

---

## Konfigurasi S3 — Wajib Ada

```php
// config/filesystems.php
'disks' => [
    's3' => [
        'driver'                  => 's3',
        'key'                     => env('AWS_ACCESS_KEY_ID'),
        'secret'                  => env('AWS_SECRET_ACCESS_KEY'),
        'region'                  => env('AWS_DEFAULT_REGION'),
        'bucket'                  => env('AWS_BUCKET'),
        'url'                     => env('AWS_URL'),
        'endpoint'                => env('AWS_ENDPOINT'),       // untuk S3-compatible (MinIO, R2)
        'use_path_style_endpoint' => env('AWS_USE_PATH_STYLE_ENDPOINT', false),
        'throw'                   => false,                     // jangan throw exception
    ],
],

// .env
AWS_ACCESS_KEY_ID=your-key
AWS_SECRET_ACCESS_KEY=your-secret
AWS_DEFAULT_REGION=ap-southeast-1
AWS_BUCKET=your-bucket
AWS_URL=https://your-bucket.s3.ap-southeast-1.amazonaws.com
```

---

## Yang DILARANG

```php
// ❌ Disk local atau public
FileUpload::make('file')->disk('public')
FileUpload::make('file')->disk('local')
Storage::disk('local')->put(...)
Storage::put(...)  // menggunakan default disk

// ❌ Nama file dari user langsung
->preserveFilenames()
// ❌ Tanpa FileHelper
fn (TemporaryUploadedFile $file) => $file->getClientOriginalName()
fn (TemporaryUploadedFile $file) => time() . '_' . $file->getClientOriginalName()  // masih mengekspos nama asli

// ❌ Hapus file lama sebelum verifikasi file baru
Storage::disk('s3')->delete($user->avatar);  // dulu
$user->update(['avatar' => $newPath]);        // baru — SALAH URUTAN

// ❌ Biarkan Filament auto-delete (tidak terkontrol)
// tanpa ->deleteUploadedFileUsing(fn() => null)
// Filament default akan hapus file saat replace — bypass dengan closure kosong
```
