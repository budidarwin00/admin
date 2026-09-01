# Kas App

Aplikasi web untuk mencatat kas kelompok. Data tersimpan di Firebase (Firestore)
sehingga **otomatis muncul di semua HP/laptop** yang membuka aplikasi, secara
real-time. Anggota hanya bisa melihat; admin login untuk menambah, mengubah,
dan menghapus data transaksi, anggota, dan bank.

## Fitur

- Ringkasan saldo, pemasukan, dan pengeluaran
- Daftar transaksi (bisa tambah/ubah/hapus, khusus admin)
- Kategori "Iuran Bulanan" otomatis mengisi nominal Rp100.000
- Daftar anggota, otomatis terurut alfabet
- Daftar rekening bank dengan tombol salin nomor rekening
- Judul & subjudul halaman bisa diubah admin
- Login admin dengan Firebase Authentication (email + password)
- Sinkron real-time antar perangkat via Firestore

---

## 1. Buat project Firebase

1. Buka https://console.firebase.google.com dan buat project baru (gratis, paket **Spark** sudah cukup).
2. Di sidebar, buka **Build > Authentication > Get started**, lalu aktifkan
   metode **Email/Password**.
3. Masih di Authentication, buka tab **Users > Add user**, buat satu akun
   dengan email dan password yang akan dipakai untuk login admin.
4. Di sidebar, buka **Build > Firestore Database > Create database**, pilih
   mode **Production**, dan lokasi server terdekat (misalnya `asia-southeast2`).
5. Buka **Project settings** (ikon gerigi) > scroll ke **Your apps** > klik ikon
   web `</>` > daftarkan app (nama bebas) > salin objek `firebaseConfig` yang muncul.
6. Tempel isi `firebaseConfig` tadi ke file `src/firebase.js`, menggantikan
   nilai placeholder `GANTI_DENGAN_...`.
7. Buka `firestore.rules`, ganti `admin@kasapp.com` dengan email admin yang
   Anda buat di langkah 3 (bisa lebih dari satu email, pisahkan dengan koma).

## 2. Jalankan di komputer (opsional, untuk coba-coba dulu)

Perlu Node.js versi 18 ke atas (unduh di https://nodejs.org).

```bash
npm install
npm run dev
```

Buka alamat yang muncul di terminal (biasanya `http://localhost:5173`).

Saat pertama kali dibuka, tab **Anggota** dan **Bank** akan kosong. Login
sebagai admin (tombol "Masuk Admin" di kanan atas), lalu klik **"Isi daftar
anggota awal"** dan **"Isi rekening awal"** untuk mengisi 15 nama anggota dan
2 rekening bank secara otomatis. Setelah itu data bisa diubah kapan saja lewat
tombol Ubah/Hapus.

## 3. Terapkan aturan keamanan Firestore

Supaya hanya admin yang bisa mengubah data, terapkan `firestore.rules` ke
project Firebase Anda. Cara termudah tanpa install apa pun: buka **Firestore
Database > Rules** di Firebase Console, tempel isi file `firestore.rules`,
lalu klik **Publish**.

Alternatif via terminal (perlu Firebase CLI, lihat langkah 5):

```bash
firebase deploy --only firestore:rules
```

## 4. Deploy ke Netlify (lewat GitHub) — direkomendasikan

1. Buat repository baru di GitHub, lalu unggah semua isi folder ini:
   ```bash
   git init
   git add .
   git commit -m "Kas app pertama"
   git branch -M main
   git remote add origin https://github.com/USERNAME/NAMA_REPO.git
   git push -u origin main
   ```
2. Buka https://app.netlify.com > **Add new site > Import an existing project**.
3. Pilih repository GitHub tadi. Netlify akan otomatis membaca `netlify.toml`
   (build command `npm run build`, folder publish `dist`) — biarkan default.
4. Klik **Deploy**. Setelah selesai, Netlify memberi alamat publik
   (`https://nama-acak.netlify.app`), bisa diganti nama di **Site settings > Domain management**.
5. Setiap kali Anda `git push` perubahan baru, Netlify build ulang otomatis.

### Alternatif: GitHub Pages

GitHub Pages hanya melayani file statis, jadi build dulu secara lokal:

```bash
npm run build
```

Lalu unggah isi folder `dist/` ke branch `gh-pages` (atau gunakan aksi
GitHub Actions siap pakai seperti `peaceiris/actions-gh-pages`), dan aktifkan
GitHub Pages di **Settings > Pages** repository Anda.

## 5. Deploy mandiri lewat Firebase Hosting (versi Firebase penuh)

Karena data sudah memakai Firebase, Anda juga bisa meng-host halamannya di
Firebase Hosting supaya semuanya dalam satu platform.

```bash
npm install -g firebase-tools
firebase login
firebase init hosting   # pilih project yang sudah dibuat, folder "dist", jawab "Yes" untuk single-page app
npm run build
firebase deploy
```

Firebase CLI akan memakai konfigurasi yang sudah ada di `firebase.json`.
Setelah `firebase deploy` selesai, Anda akan mendapat alamat
`https://NAMA_PROJECT.web.app`.

## Struktur data Firestore

| Koleksi | Field |
|---|---|
| `members` | `name` |
| `banks` | `bankName`, `accountNumber`, `ownerName` |
| `transactions` | `date`, `memberName`, `category`, `type` (`in`/`out`), `amount`, `note` |
| `settings/app` | `title`, `subtitle` |

## Catatan keamanan

- Data bisa **dibaca siapa saja** yang membuka aplikasi (sesuai permintaan:
  anggota tidak perlu login untuk melihat). Konfigurasi Firebase di
  `src/firebase.js` memang selalu terlihat di sisi browser — ini normal dan
  aman selama aturan **write** hanya mengizinkan email admin (lihat
  `firestore.rules`).
- Jangan bagikan password admin ke anggota biasa.
- Untuk menambah admin lain, buat user baru di Firebase Authentication lalu
  tambahkan emailnya ke daftar di `firestore.rules`, kemudian publish ulang.
