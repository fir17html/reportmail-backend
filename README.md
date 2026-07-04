# ReportMail Ekik — Backend (Laravel 11)

Project Laravel 11 yang sudah LENGKAP (composer.json, public/index.php, config/,
migrations) — siap langsung di-push ke GitHub & dideploy ke Railway TANPA perlu
install PHP/Composer di laptop. Cocok untuk workflow HP-only.

## Cara Deploy (HP Only, via Railway.app)

Lihat panduan lengkap step-by-step di chat. Ringkasan:

1. Push folder ini ke GitHub (pakai Termux + git di HP).
2. Railway: New Project -> Deploy from GitHub repo.
3. Tambah service Database -> MySQL.
4. Atur Variables (isi dari `.env.example`, sambungkan DB_* ke service MySQL
   pakai Reference Variable `${{MySQL.MYSQLHOST}}` dst).
5. Atur **Pre-Deploy Command**: `php artisan migrate --force && php artisan db:seed --force && php artisan storage:link`
6. Generate Domain di tab Networking.
7. Tambah service baru (dari repo yang sama) khusus untuk queue worker, dengan
   **Custom Start Command**: `php artisan queue:work --tries=3 --timeout=120`

## Login Default (setelah db:seed)

```
username: admin
password: admin123
```

GANTI password ini setelah login pertama kali!

## Catatan Konfigurasi (sudah di-set agar hemat resource trial Railway)

- `QUEUE_CONNECTION=database` -> TIDAK perlu Redis (hemat 1 service)
- `CACHE_STORE=file`, `SESSION_DRIVER=file` -> TIDAK perlu tabel cache/session tambahan
- `LOG_CHANNEL=stack` dengan channel `stderr` -> log muncul di Railway dashboard

## Endpoint Test

```
GET  /              -> health check ("API is running")
POST /api/login     -> {"username":"admin","password":"admin123"}
```

Setelah login, gunakan `token` dari response sebagai Bearer Token untuk endpoint lain.

## Microservice Telegram (Opsional, Lanjutan)

Backend ini TIDAK mengelola koneksi MTProto langsung. Untuk fitur kirim Telegram,
perlu microservice Python (Telethon) terpisah — lihat `app/Services/TelegramAuthService.php`
untuk kontrak API yang diharapkan. Fitur Email bisa langsung dipakai tanpa ini.

## Struktur Folder

```
app/Models/                 -> 8 Eloquent model
app/Http/Controllers/Api/   -> Controller per fitur (+ Admin/)
app/Http/Requests/          -> FormRequest validasi
app/Http/Middleware/        -> CheckSenderLimit (limit 200) & EnsureUserIsAdmin
app/Jobs/SendBlastReportJob.php -> Job blast email/telegram
app/Services/               -> ReportTextService (prefix+signature), TelegramAuthService
database/migrations/        -> 12 migration (8 fitur + 4 standar Laravel/Sanctum)
database/seeders/           -> DatabaseSeeder (akun admin awal)
routes/api.php              -> semua endpoint REST
config/                     -> konfigurasi Laravel standar (sudah disesuaikan)
public/index.php            -> entry point web server
```
