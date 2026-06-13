# Panduan Deployment Laravel - ShareMeal (Railway)

Dokumen ini berisi panduan langkah demi langkah untuk men-deploy aplikasi Laravel **ShareMeal** ke **Railway**. Platform ini mendukung PHP secara native, menyediakan basis data terintegrasi, dan mendeteksi konfigurasi Laravel secara otomatis.

---

## Langkah-langkah Deployment:

1. **Daftar & Hubungkan GitHub**:
   * Buka [Railway.app](https://railway.app) dan masuk menggunakan akun GitHub Anda.

2. **Buat Database Baru**:
   * Di dashboard Railway, klik **New Project** -> **Provision PostgreSQL** atau **Provision MySQL** (pilih salah satu sesuai kebutuhan database Anda).

3. **Tambahkan Layanan Aplikasi**:
   * Klik **+ Add** -> **GitHub Repo** -> pilih repositori `ShareMeal_Deployment`.

4. **Konfigurasi Variabel Lingkungan (Variables)**:
   * Klik pada layanan aplikasi `ShareMeal_Deployment` Anda, masuk ke tab **Variables**, lalu klik **New Variable** untuk menambahkan variabel berikut:
     * `APP_ENV`: `production`
     * `APP_DEBUG`: `false`
     * `APP_KEY`: *(Dapatkan kunci ini dengan menjalankan `php artisan key:generate --show` di terminal lokal Anda)*
     * `APP_URL`: `${{ RAILWAY_PUBLIC_DOMAIN }}` (Railway akan mengisi ini secara otomatis)

5. **Hubungkan Database ke Aplikasi**:
   * Karena database MySQL dan aplikasi Laravel Anda berada di dua service terpisah di Railway, Anda **wajib** merujuk ke service database menggunakan prefix nama service tersebut (misalnya, jika service database Anda bernama `mysql`):
     * **Jika nama service database Anda di canvas Railway adalah `mysql`**:
       * `DB_CONNECTION`: `mysql`
       * `DB_HOST`: `${{mysql.MYSQLHOST}}`
       * `DB_PORT`: `${{mysql.MYSQLPORT}}`
       * `DB_DATABASE`: `${{mysql.MYSQLDATABASE}}`
       * `DB_USERNAME`: `${{mysql.MYSQLUSER}}`
       * `DB_PASSWORD`: `${{mysql.MYSQLPASSWORD}}`
     * **Catatan Penting**: 
       * Jangan menggunakan tanda kutip `"` atau `'` pada nilai variabel di tab Variables Railway.
       * Jika service database Anda di canvas Railway bernama `MySQL` (dengan huruf besar), gunakan `${{MySQL.MYSQLHOST}}` dst. (karena penamaan service bersifat case-sensitive).

6. **Command Jalankan Migrasi Otomatis & Clear Cache**:
   * Selama proses build, Nixpacks mungkin menjalankan caching konfigurasi secara otomatis menggunakan nilai default (yang membuat koneksi database gagal).
   * Di tab **Settings** layanan aplikasi Anda, cari bagian **Deploy** -> **Custom Start Command**, lalu isi dengan perintah berikut untuk menghapus cache build-time dan menjalankan migrasi saat startup:
     ```bash
     php artisan config:clear && php artisan migrate --force
     ```
   * Railway secara otomatis menggunakan Nixpacks untuk mendeteksi bahwa ini adalah proyek Laravel, mengompilasi aset front-end (Vite), menginstal dependencies Composer, dan mengarahkan web server ke direktori `public/`.
