# docker-compose-laravel
Alur kerja Docker Compose yang cukup disederhanakan yang menyiapkan jaringan wadah LEMP untuk pengembangan Laravel lokal. Anda dapat melihat artikel lengkap yang menginspirasi repo ini [here](https://dev.to/aschmelyun/the-beauty-of-docker-for-local-laravel-development-13c0).

[![GitNFT](https://img.shields.io/badge/%F0%9F%94%AE-Open%20in%20GitNFT-darkviolet?style=flat)](https://gitnft.quine.sh/app/commits/list/repo/docker-compose-laravel)

## Penggunaan

Untuk memulai, pastikan Anda memiliki [Docker installed](https://docs.docker.com/docker-for-mac/install/) di sistem Anda, dan kemudian mengkloning repositori ini.

Selanjutnya, navigasikan di terminal Anda ke direktori yang Anda kloning ini, dan putar wadah untuk server web dengan menjalankan `docker-compose up -d --build site`.

Setelah itu selesai, ikuti langkah-langkah dari [src/README.md](src/README.md) file untuk menambahkan proyek Laravel Anda (atau buat yang kosong baru).

Memunculkan jaringan Docker Compose dengan `site` alih-alih hanya menggunakan `up`, memastikan bahwa hanya container situs kami yang ditampilkan di awal, bukan semua container perintah juga. Berikut ini dibuat untuk server web kami, dengan detail port yang terbuka:

- **nginx** - `:80`
- **mysql** - `:3306`
- **php** - `:9000`
- **redis** - `:6379`
- **mailhog** - `:8025` 

Tiga kontainer tambahan disertakan yang menangani perintah Composer, NPM, dan Artisan *tanpa* harus menginstal platform ini di komputer lokal Anda. Gunakan contoh perintah berikut dari root proyek Anda, modifikasi agar sesuai dengan kasus penggunaan khusus Anda.

- `docker-compose run --rm composer update`
- `docker-compose run --rm npm run dev`
- `docker-compose run --rm artisan migrate`

## Permissions Issues

Jika Anda mengalami masalah dengan izin sistem file saat mengunjungi aplikasi Anda atau menjalankan perintah container, coba selesaikan salah satu rangkaian langkah di bawah ini.

**Jika Anda menggunakan server atau lingkungan lokal Anda sebagai pengguna root:**
- Turunkan container apa pun dengan `docker-compose down`
- Ganti nama file `docker-compose.root.yml` menjadi `docker-compose.root.yml`, menggantikan yang sebelumnya
- Bangun kembali wadah dengan menjalankan `docker-compose build --no-cache`

**Jika Anda menggunakan server atau lingkungan lokal Anda sebagai pengguna yang bukan root:**

- Turunkan container apa pun dengan `docker-compose down`
- Di terminal Anda, jalankan `export UID=$(id -u)` lalu `export GID=$(id -g)`
- Jika Anda melihat kesalahan tentang variabel readonly dari langkah di atas, Anda dapat mengabaikannya dan melanjutkan
- Bangun kembali wadah dengan menjalankan `docker-compose build --no-cache`

Kemudian, bawa kembali jaringan penampung Anda atau jalankan kembali perintah yang Anda coba sebelumnya, dan lihat apakah itu memperbaikinya.

## Persistent MySQL Storage

Secara default, setiap kali Anda mematikan jaringan Docker, data MySQL Anda akan dihapus setelah wadah dihancurkan. Jika Anda ingin memiliki data persisten yang tersisa setelah menurunkan dan mencadangkan container, lakukan hal berikut:

1. Buat folder `mysql` di root proyek, di samping folder `nginx` dan `src`.
2. Di bawah layanan mysql di file `docker-compose.yml` Anda, tambahkan baris berikut:

```
volumes:
  - ./mysql:/var/lib/mysql
```

## Menggunakan BrowserSync dengan Laravel Mix

Jika Anda ingin mengaktifkan hot-reload yang disertakan dengan opsi BrowserSync Laravel Mix, Anda harus mengikuti beberapa langkah kecil. Pertama, pastikan Anda menggunakan `docker-compose.yml` yang diperbarui dengan port `:3000` dan `:3001` terbuka pada layanan npm. Kemudian, tambahkan berikut ini di akhir file `webpack.mix.js` proyek Laravel Anda:

```javascript
.browserSync({
    proxy: 'site',
    open: false,
    port: 3000,
});
```

Dari jendela terminal Anda di root proyek, jalankan perintah berikut untuk mulai melihat perubahan dengan wadah npm dan port yang dipetakan:

```bash
docker-compose run --rm --service-ports npm run watch
```

Itu akan membuat panel info kecil tetap terbuka di terminal Anda (yang dapat Anda keluarkan dengan Ctrl + C). Mengunjungi [localhost:3000](http://localhost:3000) di browser Anda kemudian akan memuat aplikasi Laravel Anda dengan BrowserSync diaktifkan dan hot-reloading aktif.

## MailHog

Versi Laravel saat ini (8 hari ini) menggunakan MailHog sebagai aplikasi default untuk menguji pengiriman email dan pekerjaan SMTP umum selama pengembangan lokal. Menggunakan image Docker Hub yang disediakan, menyiapkan dan menyiapkan instans menjadi mudah dan mudah. Layanan ini disertakan dalam file `docker-compose.yml`, dan berputar bersama server web dan layanan database.

Untuk melihat dasbor dan melihat email apa pun yang masuk melalui sistem, kunjungi [localhost:8025](http://localhost:8025) setelah menjalankan `docker-compose up -d site`.
