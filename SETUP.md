# MODVAULT + Supabase + Vercel (setup dari HP)

## Yang perlu diketahui
Website ini sudah memakai Supabase Auth dan database dengan Row Level Security (RLS). Namun kamu harus membuat project Supabase milikmu dan mengisi konfigurasi sendiri. Aku tidak bisa membuat project atau mengambil API key atas namamu dari sini.

## 1. Buat project Supabase
1. Buka https://supabase.com/ lalu login/daftar.
2. Buat project baru, simpan password database di tempat aman.
3. Buka **Project Settings → API** (atau **Connect**) dan salin **Project URL** serta **anon/publishable key**.
4. Buka file `config.js`, ganti nilai `SUPABASE_URL` dan `SUPABASE_ANON_KEY`.
   - Key anon/publishable boleh berada di frontend dengan RLS yang benar.
   - JANGAN pernah masukkan `service_role` / secret key ke `config.js` atau GitHub.

## 2. Buat tabel dan kebijakan keamanan
1. Di Supabase buka **SQL Editor → New query**.
2. Salin seluruh isi `schema.sql`, lalu jalankan.
3. Ini membuat tabel profil, daftar versi, trigger profil otomatis, serta RLS. Hanya role admin yang boleh mengubah daftar versi.

## 3. Buat akun login
Gunakan **Authentication → Users → Add user / Create new user**. Buat dua akun dengan email berikut (email ini hanya alias login website; jangan pakai untuk email sungguhan):
- `admin@modvault.local` dengan password awal `ADMIN123`
- `user@modvault.local` dengan password awal `USER`

Jika dashboard meminta konfirmasi email, nonaktifkan email confirmation sementara di **Authentication → Providers → Email** untuk demo ini, atau ikuti opsi konfirmasi yang tersedia. Jangan gunakan password demo ini untuk akun pribadi lain.

## 4. Jadikan akun pertama sebagai admin
Setelah kedua akun dibuat, buka SQL Editor dan jalankan query ini:
```sql
update public.profiles set username='ADMIN', role='admin'
where id = (select id from auth.users where email='admin@modvault.local');

update public.profiles set username='USER', role='user'
where id = (select id from auth.users where email='user@modvault.local');
```
Pastikan hasilnya satu baris terubah untuk masing-masing akun. Jika profile tidak ditemukan, pastikan `schema.sql` sudah dijalankan sebelum membuat akun.

## 5. Deploy ke Vercel
1. Unggah semua file dalam folder ini ke repository GitHub (index.html, styles.css, app.js, config.js, schema.sql, SETUP.md).
2. Buka Vercel dan impor repository tersebut.
3. Karena website statis, framework preset **Other**, build command kosong, output directory root/default.
4. Setelah deploy, buka domain Vercel dan uji login admin dan user.

## 6. Tes akses
- Login ADMIN / ADMIN123: panel admin tampil; tambah/edit/hapus versi berfungsi.
- Login USER / USER: bisa membaca dan mengunduh versi, tidak bisa mengelola.
- Ganti password di panel admin mengubah password Supabase Auth admin. Setelah diganti, login berikutnya pakai password baru.

## Catatan keamanan penting
- Akun USER adalah akun bersama sesuai permintaanmu. Semua orang yang tahu password-nya dapat masuk sebagai user; jangan gunakan akun bersama untuk akses pribadi.
- Jangan pernah beri role admin ke akun user.
- RLS melindungi perubahan database, walaupun seseorang mencoba memanggil API langsung.
- Ini portal tautan download, bukan tempat menyimpan file APK secara langsung. Upload APK ke penyimpanan tepercaya (misalnya Supabase Storage) jika ingin hosting file di website.
- Hanya distribusikan aplikasi yang memang berhak kamu bagikan dan pastikan file aman.
