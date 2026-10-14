# RekGo V4 — Laravel + MySQL

> **Status: development / belum siap untuk transaksi uang asli.** Repository ini merupakan titik awal dokumentasi. Source aplikasi lengkap **belum berhasil dipush ke GitHub** dari sesi ini. Jangan deploy repo ini sebagai aplikasi jadi.

## Gambaran sistem

RekGo adalah rancangan aplikasi rekening bersama (escrow) berbasis Laravel dengan UI mobile-first iOS-inspired Liquid Glass (mode terang/gelap), dashboard user dan admin, PWA, serta integrasi bertahap GoPay Merchant dan WhatsApp.

## Fitur dan fungsinya

| Modul | Fungsi | Status |
| --- | --- | --- |
| Akun | Login, registrasi, foto profil, rekening pencairan | Fondasi source lokal |
| Dashboard User | Saldo tersedia, tertunda, dalam pencairan; riwayat | Fondasi source lokal |
| Dashboard Admin | Saldo seluruh akun, transaksi, user, fee, ACC transfer | Fondasi source lokal |
| WD | Pending → Proses → Success, Rejected jika ditolak; audit log | Fondasi source lokal; perlu bank inquiry |
| Fee | Tier nominal/persentase, penanggung fee rekber dan QRIS terpisah | Fondasi source lokal |
| QRIS | Nominal sendiri, QR grafis, expiry, pembatalan, rekonsiliasi | Kode lokal; **belum diuji pembayaran nyata** |
| GoPay Merchant | Node bridge, OTP merchant, status transaksi | Kode lokal; **belum diuji merchant** |
| WhatsApp | Registrasi multi-nomor, rencana QR/pairing Neoxr + Baileys | **Belum terhubung** |
| PWA | Manifest, ikon, service worker aset statis | Fondasi source lokal |
| Maintenance | ON/OFF dan pesan perbaikan | Fondasi source lokal |
| Pengumuman | Popup, poster, jadwal, target user | Fondasi source lokal |
| Admin Roles | Super Admin / Finance / Support | Fondasi; perlu pengujian otorisasi |
| Keuangan | Ledger saldo dan pencegahan duplikasi | Fondasi; perlu audit & tes integrasi |

**Penting:** Fee platform tidak masuk saldo user. WD terkunci menjadi nol setelah semua WD selesai, tetapi saldo terkunci bisa tetap ada jika masih ada pengajuan lain.

## Arsitektur yang dituju

- Laravel 12 / PHP 8.2+ / MySQL 8+ atau MariaDB kompatibel.
- Nginx + PHP-FPM untuk web; HTTPS wajib.
- Node.js 20+ untuk service Neoxr/Baileys dan GoPay bridge, masing-masing dibatasi ke loopback/internal network.
- PM2/systemd untuk Node; cron Laravel scheduler setiap menit.
- Tidak melakukan polling GoPay jika tidak ada QRIS pending. Webhook/rekonsiliasi tetap harus aman.

## Instalasi VPS Debian 12 (setelah source lengkap tersedia)

```bash
sudo apt update
sudo apt install nginx mariadb-server git unzip curl composer php-fpm php-cli php-mysql php-mbstring php-xml php-curl php-zip php-bcmath
git clone https://github.com/scardbypass/Rekgo.git
cd Rekgo
composer install --no-dev --optimize-autoloader
cp .env.example .env
php artisan key:generate
```

**Jangan lanjut menjalankan perintah Laravel di repo ini sampai source lengkap benar-benar tersedia.** Sesuaikan PHP versi 8.2+ dan ekstensi sesuai `composer.json`. Buat database dan akun SQL dengan hak minimum; isi `APP_URL`, `DB_HOST`, `DB_DATABASE`, `DB_USERNAME`, `DB_PASSWORD`, `APP_ENV=production`, `APP_DEBUG=false` di `.env` (jangan commit rahasia).

Setelah source lengkap dan sudah diuji:

```bash
php artisan migrate --force
php artisan storage:link
php artisan optimize
php artisan route:list
```

Nginx harus mengarah ke `/path/to/Rekgo/public`, **bukan root repository**. Terapkan TLS dan izin tulis hanya untuk `storage/` serta `bootstrap/cache/`. Scheduler:

```cron
* * * * * cd /path/to/Rekgo && /usr/bin/php artisan schedule:run >> /dev/null 2>&1
```

Untuk Node bridge, jalankan hanya setelah SDK yang berlisensi dipasang secara sah dan variabel lingkungan privat telah dikonfigurasi. Jangan mempublikasikan port bridge, session Baileys, token, OTP, atau kredensial merchant. Simpan proses Node di PM2 dan pantau log.

## Pengaturan Admin yang direncanakan

- **Website:** nama, logo, mode maintenance, pesan, tema, PWA.
- **Pengumuman:** poster JPG/PNG/WebP, judul, isi, jadwal, aktif/nonaktif.
- **User:** profil, saldo, transaksi, status akun.
- **Admin:** tambah Finance/Support oleh Super Admin; hak akses minimum.
- **ACC Transfer:** rekening lengkap hanya untuk admin berwenang; verifikasi; proses; success; alasan penolakan; bukti opsional; audit.
- **Fee:** tier rekber dan QRIS; tidak membebankan MDR QRIS kepada konsumen.
- **Payment Gateway:** OTP merchant, outlet, enable/disable, QRIS On-Demand, interval pemeriksaan, rekonsiliasi.
- **WhatsApp:** daftar nomor, QR/pairing, status sesi, reconnect, logout; belum operasional.
- **Storage:** retensi bukti 45 hari **hanya jika** perlindungan sengketa/legal hold sudah tersedia; riwayat keuangan tidak dihapus.

## Pengujian wajib sebelum produksi

Verifikasi izin penyelenggaraan layanan, penyedia pembayaran resmi, dan kepatuhan; tes OTP/QRIS dengan merchant sah, jumlah yang dibayar, pembayaran terlambat, pembatalan, refund, idempotensi ledger, pencairan bank, rekonsiliasi, backup/restore, 2FA admin, dan otorisasi data rekening.

## Status upload

GitHub repository: https://github.com/scardbypass/Rekgo

Dokumentasi ini dipush terlebih dahulu. Paket kode lokal dari sesi pengembangan harus diunggah secara utuh sebelum instruksi instalasi di atas dapat digunakan.
