# Uploadin-Gdrive

Selamat datang di repositori Aplikasi Upload File! Ini adalah sebuah aplikasi web yang dibangun sepenuhnya menggunakan Google Apps Script, yang memungkinkan pengguna untuk mengunggah file ke folder Google Drive yang terorganisir secara otomatis dan mencatat setiap aktivitas unggahan ke dalam sebuah Google Sheet yang berfungsi sebagai database.
Proyek ini dirancang untuk menjadi solusi praktis bagi tim atau individu yang membutuhkan sistem pengumpulan file yang terpusat dan terdokumentasi dengan baik.

Fitur Utama
- Antarmuka Modern: UI yang bersih dan responsif dengan dukungan drag-and-drop.
- Akses Kamera: Ambil foto atau rekam video langsung dari kamera perangkat seluler.
- Organisasi Otomatis: Membuat folder dan sub-folder secara dinamis berdasarkan input pengguna.
- Pencatatan Metadata: Setiap file yang diunggah akan tercatat di Google Sheet, lengkap dengan timestamp, ukuran, tipe file, dan URL.
- Riwayat & Statistik: Tampilan riwayat unggahan dan data sheet dengan fitur pagination dan statistik total file.
- Bantuan Input: Dropdown saran otomatis untuk nama folder yang sudah pernah digunakan untuk konsistensi data.
- Tanpa Server Eksternal: Berjalan sepenuhnya di ekosistem Google, tidak memerlukan hosting atau database tambahan.

## Cara Menggunakan
1. Buat Prasyarat:
- Buat sebuah Google Sheet baru. Salin ID-nya dari URL.
- Buat sebuah Folder baru di Google Drive. Salin ID-nya dari URL.
2. Setup Proyek Apps Script:
- Buka script.google.com dan buat Proyek Baru.
- Salin seluruh isi Code.gs dari repositori ini ke dalam file Code.gs di proyek Anda.
- Buat file HTML baru (File > New > HTML file) dan beri nama index. Salin seluruh isi index.html dari repositori ini ke dalamnya.
- Tempelkan ID Sheet dan Folder Anda ke dalam variabel konfigurasi di Code.gs.
3. Deploy Aplikasi:
- Klik Deploy > New deployment.
- Klik ikon gerigi dan pilih Web app.
- Pada Who has access, pilih Anyone.
- Klik Deploy. Berikan izin akses saat diminta.
- Salin Web app URL yang diberikan. Aplikasi Anda sudah online!
