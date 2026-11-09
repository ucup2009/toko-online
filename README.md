# yStore — Toko Online Pakaian

Toko online sederhana berbasis **CodeIgniter 3.1.13** + **PHP 8.2** + **MySQL/MariaDB**. Katalog produk dengan kategori, pencarian & pagination (server-side), keranjang belanja (localStorage), checkout lewat **WhatsApp**, serta panel admin mandiri (produk, kategori, profil toko, user admin, dashboard trafik).

Tanpa proses build, tanpa `composer install` — framework CI3 sudah dibundel di folder `system/`. Cukup extract, import database, dan jalankan.

## Fitur

- **Katalog publik** — beranda (maks. 6 produk terbaru), halaman `/produk` (9/halaman, pagination, filter kategori, pencarian, urutan harga/nama), detail produk + galeri + produk terkait.
- **Keranjang & checkout** — keranjang disimpan di browser (localStorage `ystore-cart`), tombol checkout menyusun pesan lalu mengirim ke WhatsApp toko (tanpa tabel order / pembayaran online).
- **Panel admin** (`/admin`) — login dengan rate limiting (5 gagal / 15 menit per IP), CRUD produk & kategori (cover + galeri), kelola profil toko (logo, judul beranda, alamat, WhatsApp, Instagram/Facebook), kelola akun admin, dashboard trafik pengunjung (`visitor_logs`).
- **SEO** — `sitemap.xml` & `robots.txt` dinamis, meta description + Open Graph per halaman, halaman 404 bertemplate situs, clean URLs.
- **Halaman statis** — `/tentang`, `/syarat-ketentuan`, `/kebijakan-privasi`.

## Struktur Folder

```
toko-online/
├── index.php              # front controller CI3
├── .htaccess              # clean URLs, keamanan, blok Force HTTPS (komentar)
├── database.sql           # schema + seed (18 produk, 3 kategori, 1 admin)
├── application/           # kode aplikasi (controllers, models, views, config)
├── system/                # framework CodeIgniter 3.1.13 — JANGAN diedit
└── assets/
    ├── template/          # halaman publik (CSS, JS, img)
    ├── admin/             # aset panel admin (admin.css, admin.js) + Tailwind CDN
    └── uploads/           # foto produk, logo, judul beranda (ikut di zip)
        ├── products/
        ├── logos/
        └── hero/
```

## Persyaratan

- PHP **8.2**
- MySQL atau MariaDB
- Apache dengan `mod_rewrite` (XAMPP untuk lokal; shared hosting cPanel biasanya sudah aktif)
- Tidak perlu Composer, Node, atau proses build apa pun

---

## Setup Lokal (XAMPP)

1. **Extract zip** ke `C:\xampp\htdocs\toko-online`.
   Pastikan hasil extract tidak jadi subfolder ganda (`htdocs/toko-online/toko-online`) — kalau yang terbuka `toko-online.zip` di dalam `toko-online/`, pindahkan isinya satu level ke atas.
2. **Buka XAMPP Control Panel** → Start **Apache** dan **MySQL**.
3. **Buat database bernama ystore lalu Import database.** Buka `http://localhost/phpmyadmin` → pilih ystore → tab **Import** → pilih `database.sql` (di root repo) → Go.
4. **Konfigurasi tidak perlu diubah.** Bawaan `application/config/database.php` (`root` / tanpa password / `ystore`) sudah cocok dengan XAMPP, dan `base_url` bawaan `http://localhost/toko-online/` juga sudah benar.
5. **Jalankan.** Buka `http://localhost/toko-online/`.
   - Halaman publik: `/`, `/produk`, `/tentang`
   - Panel admin: `http://localhost/toko-online/admin` → login **`admin` / `admin123`**
6. **Ganti password admin** begitu login pertama (menu admin → Users).

> **Catatan:** kalau zip yang kamu terima tidak berisi `assets/uploads/`, semua foto produk otomatis tampil sebagai gambar cadangan "Foto belum tersedia" (`noimage.jpg`). Foto bisa diunggah ulang lewat panel admin (menu Produk → form tambah/edit).

---

## Setup Shared Hosting (cPanel)

Urutan yang disarankan. Langkah 1–2 wajib sebelum langkah lainnya berarti.

1. **Domain & SSL.** Arahkan domain ke hosting, aktifkan SSL (AutoSSL / Let's Encrypt dari cPanel). Pastikan `https://domainanda.com` sudah hijau sebelum lanjut.
2. **Upload & extract.** Zip-kan isi repo (bukan foldernya), lalu upload ke `public_html/` dan extract lewat **File Manager** cPanel — hasilnya harus langsung `public_html/index.php`, bukan `public_html/toko-online/index.php`.
   - Pastikan file `.htaccess` root ikut terunggah. Kalau pakai FileZilla, aktifkan dulu *Server → Force showing hidden files* — dotfile sering tidak ikut terkirim dan akibatnya semua halaman 404.
   - Pastikan folder `assets/uploads/` (±8 MB, berisi foto produk/logo/hero) ikut terunggah — tanpa ini semua foto 404.
3. **Buat database.** cPanel → **MySQL Databases** → buat database + user baru, beri user **ALL PRIVILEGES**. Catat nama `cpaneluser_ystore` (biasanya berprefix username cPanel).
4. **Import database.** cPanel → **phpMyAdmin** → pilih database hasil langkah 3 → tab **Import** → pilih `database.sql` → Go.
5. **Isi kredensial database.** Buka `application/config/database.php`, cari array `$db['default']` lalu ubah 4 nilai ini sesuai kredensial cPanel:

   | Key | Nilai |
   |---|---|
   | `hostname` | `localhost` (umumnya tetap `localhost` di shared hosting) |
   | `username` | username database, mis. `u1234567_ystore` |
   | `password` | password database |
   | `database` | nama database, mis. `u1234567_ystore` |

   (Bawaan file ini gaya XAMPP: `root` / kosong / `ystore` — jadi di lokal tidak perlu disentuh.)
6. **Isi alamat situs.** Buka `application/config/config.php`, cari `$config['base_url']` (sekitar baris 35) lalu ganti ke domain sendiri, diakhiri garis miring:
   ```php
   $config['base_url'] = 'https://domainanda.com/';
   ```
7. **Aktifkan paksa HTTPS.** Di `.htaccess` root, hapus tanda `#` pada 3 baris blok "Force HTTPS" (`RewriteCond %{HTTPS} off` …) supaya semua kunjungan HTTP dialihkan ke HTTPS.
8. **Amankan akun & isi data toko.** Login `admin` / `admin123` → **langsung ganti password** (Users). Lalu menu **Kelola Toko** — semua data di bawah ini masih placeholder:
   - Alamat & telepon asli toko
   - **Nomor WhatsApp** (bawaan `6281234567890` masih palsu — semua tombol checkout mengarah ke nomor ini)
   - Instagram / Facebook (kosong = kolom "Sosial" di footer disembunyikan)
   - Logo toko & judul beranda
9. **Uji setelah live:** `/`, `/produk` (pagination & filter), detail produk, `/tentang`, `/syarat-ketentuan`, `/kebijakan-privasi`, `/robots.txt`, `/sitemap.xml`, buka halaman ngawur (harus 404 rapi, bukan error server), login admin, simpan produk, upload gambar, tombol checkout WhatsApp.
10. **Pasca-live:** daftarkan `https://domainanda.com/sitemap.xml` ke Google Search Console; kosongkan isi `application/logs/`; pastikan tampilan error produksi tidak menampilkan stack trace (`ENVIRONMENT` otomatis `production` untuk host non-localhost).

---

## Konfigurasi Tambahan

| Opsi | Cara | Kapan dipakai |
|---|---|---|
| `ENVIRONMENT` otomatis | `index.php`: host `localhost`/`127.0.0.1`/CLI → `development`, lainnya → `production` | Selalu aktif; `display_errors` otomatis mati di hosting |
| `CI_ENV` | `SetEnv CI_ENV development` di `.htaccess` | Memaksa environment tertentu (mis. matikan display_errors di lokal) |
| `APP_BASE_URL` | `SetEnv APP_BASE_URL https://domainanda.com/` di `.htaccess` | Alternatif mengisi `base_url` tanpa menyentuh `config.php` |
| `database.local.php` | Buat `application/config/database.local.php` berisi override `$db['default']` (file ini gitignored) | Opsional — bila repo dipakai lewat git, supaya kredensial produksi tidak ikut ter-commit |

Catatan kecil untuk pengembang: karena distribusi utama berupa zip, langkah wajib di atas cukup mengedit `database.php` & `config.php` langsung. `database.local.php` dan `APP_BASE_URL` hanya relevan bila proyek ini suatu saat dikelola dengan git.

---

## Troubleshooting

- **Semua halaman 404** — `.htaccess` tidak terkirim / `mod_rewrite` mati / `AllowOverride None`. Pastikan file `.htaccess` ada di docroot dan hosting mengizinkan rewrite.
- **Halaman 500** — lihat `application/logs/log-*.php`. Penyebab umum: folder `application/logs/` atau `assets/uploads/` tidak bisa ditulis web server.
- **Login admin ditolak terus** — rate limiting (5 gagal / 15 menit per IP). Reset dengan menjalankan `TRUNCATE login_attempts;` di phpMyAdmin.
- **Semua foto produk hilang/ganti gambar abu-abu** — folder `assets/uploads/` tidak ikut terunggah atau tidak bisa ditulis.
- **Checkout WhatsApp salah nomor** — nomor diambil dari pengaturan Kelola Toko, bukan dari file kode.
- **Jangan jalankan `composer install`** — post-install script-nya untuk Linux dan `composer_autoload` memang sengaja `FALSE`; framework sudah dibundel di `system/`.
- **Jangan edit `system/`** — inti framework dibiarkan vanilla agar mudah di-upgrade.
- **File `.md` / `.sql` tidak bisa dibuka via browser** — sengaja diblokir `.htaccess` ( proteksi dokumen & dump database).
- **Tailwind hanya untuk halaman admin** — jangan pasang Tailwind CDN di `templates/header.php`; utilitas `.container`-nya menimpa CSS template front-end.

## Akun Default

| Peran | Login |
|---|---|
| Admin | `admin` / `admin123` — **ganti segera setelah dipakai** |
#   t o k o - o n l i n e  
 