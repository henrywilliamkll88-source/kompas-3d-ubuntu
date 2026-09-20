# KOMPAS-3D di Ubuntu: CAD profesional native — panduan instalasi v25 Home

[English](README.md) | [Русский](README.ru.md) | [简体中文](README.zh-CN.md) | [हिन्दी](README.hi.md) | [Español](README.es.md) | [العربية](README.ar.md) | [Français](README.fr.md) | [Português](README.pt.md) | [বাংলা](README.bn.md) | [Bahasa Indonesia](README.id.md)

Jalankan versi Linux KOMPAS-3D langsung di Ubuntu, tanpa Wine atau mesin virtual. Panduan komunitas ini mendokumentasikan instalasi yang berhasil pada Ubuntu 26.04.1 LTS, amd64. Edisi yang dipasang adalah Home dari produk CAD profesional ini.

> Ubuntu belum didukung secara resmi oleh ASCON. Home hanya untuk penggunaan pribadi nonkomersial; judul ini tidak menyiratkan lisensi komersial. ASCON menawarkan uji coba Home selama 60 hari, yang tidak tersedia di mesin virtual atau server terminal. Lihat ketentuan resmi di bawah.

Dicatat pada 2026-09-20: Ubuntu 26.04.1 LTS (resolute), amd64, paket KOMPAS 25.0.1.2738. Simulasi awal dengan dua paket menambahkan 47 paket tanpa menghapus atau memperbarui paket yang ada; dependensi sistem berasal dari Ubuntu. Utilitas aktivasi ditambahkan secara terpisah. Pengguna kemudian menyatakan semuanya berfungsi. Rakitan besar, kinerja, dan stabilitas jangka panjang belum diuji secara terpisah.

## 1. Periksa sistem dan siapkan alat

Gunakan Bash dan jalankan blok secara berurutan. Arsitektur harus amd64. Berhenti jika perintah gagal. Prosedur ini untuk Ubuntu 26.04; versi lain perlu divalidasi tersendiri.

```bash
. /etc/os-release
printf '%s (%s)\n' "$PRETTY_NAME" "$VERSION_CODENAME"
dpkg --print-architecture
sudo apt update
sudo apt install curl gnupg ca-certificates
mkdir -p ~/Downloads/kompas25
cd ~/Downloads/kompas25
```

## 2. Unduh kunci repositori

Lanjutkan di terminal yang sama, dalam ~/Downloads/kompas25. Kunci diunduh dari ASCON melalui HTTPS dan dibatasi ke repositori masing-masing dengan signed-by.

```bash
curl -fSL https://repo.ascon.ru/personal/deb/ascon.gpg -o ascon-personal.gpg &&
curl -fSL https://repo.ascon.ru/stable/deb/ascon.gpg -o ascon-stable.gpg &&
gpg --dearmor --yes --output ascon-personal-keyring.gpg ascon-personal.gpg &&
gpg --dearmor --yes --output ascon-stable-keyring.gpg ascon-stable.gpg &&
sudo install -d -m 0755 /etc/apt/keyrings &&
sudo install -m 0644 ascon-personal-keyring.gpg /etc/apt/keyrings/ &&
sudo install -m 0644 ascon-stable-keyring.gpg /etc/apt/keyrings/
```

## 3. Tambahkan repositori ASCON

Saat instalasi, kedua repositori hanya menyediakan 1.8_x86-64. Skrip vendor akan memasukkan resolute dan menghasilkan HTTP 404. Karena itu, kita memilih cabang paket ASCON untuk Astra Linux secara eksplisit. Jangan tambahkan repositori sistem operasi Astra Linux. Perintah ini menimpa kedua berkas .list yang disebutkan; periksa dan cadangkan berkas lama jika sudah memakai repositori ASCON. Berhenti jika ada kesalahan tanda tangan atau repositori; jangan nonaktifkan verifikasi.

```bash
echo 'deb [arch=amd64 signed-by=/etc/apt/keyrings/ascon-personal-keyring.gpg] https://repo.ascon.ru/personal/deb 1.8_x86-64 main' | sudo tee /etc/apt/sources.list.d/ascon-personal.list
echo 'deb [arch=amd64 signed-by=/etc/apt/keyrings/ascon-stable-keyring.gpg] https://repo.ascon.ru/stable/deb 1.8_x86-64 main' | sudo tee /etc/apt/sources.list.d/ascon.list
sudo apt update
```

## 4. Simulasikan instalasi

Tidak ada paket yang diubah. Tinjau seluruh rencana: tidak boleh ada penghapusan, penurunan versi, atau penggantian pustaka Ubuntu dengan versi distribusi lain. Jumlah paket dapat berbeda. Jika dependensi tidak terpenuhi, selidiki penyebabnya tanpa memaksa instalasi.

```bash
apt-get --simulate install ascon-kompas3d-v25 ascon-kompas-home-v25 ascon-kompas-kactivation-v25
```

## 5. Pasang KOMPAS dan utilitas aktivasi

Utilitas aktivasi yang tidak ada dalam instalasi minimal pertama disertakan. Periksa rencana APT sebelum menyetujui. --no-remove menghentikan APT jika perlu menghapus paket.

```bash
sudo apt-get --no-remove install ascon-kompas3d-v25 ascon-kompas-home-v25 ascon-kompas-kactivation-v25
```

## 6. Jalankan sebagai pengguna biasa

Jangan jalankan aplikasi dengan sudo. Keluaran terminal disimpan di first-launch.log; periksa informasi pribadi sebelum membagikannya.

```bash
kompas-home-v25 2>&1 | tee ~/Downloads/kompas25/first-launch.log
```

## 7. Aktifkan uji coba

Buka Bantuan → Utilitas kunci proteksi (Справка → Утилита ключа защиты) → Lisensi uji coba (Ознакомительные лицензии). Pilih mode uji coba, masukkan email, baca pemberitahuan privasi dan centang persetujuan jika setuju, lalu klik Activate. Nama menu bergantung pada bahasa antarmuka; terjemahan panduan ini tidak mengubah bahasa aplikasi.

## 8. Pemecahan masalah

**Utilitas kunci proteksi tidak ditemukan:** tutup KOMPAS, pasang paket berikut, lalu buka kembali aplikasinya.

```bash
sudo apt-get --no-remove install ascon-kompas-kactivation-v25
```

**Kontrol bertumpuk atau kotak persetujuan tidak dapat diklik:** gunakan Tab / Shift+Tab untuk memfokuskan kotak, lalu Spasi untuk mengubahnya. Jika perlu, ubah sementara Pengaturan Ubuntu → Layar → Skala menjadi 100%, tutup utilitas dan KOMPAS, lalu jalankan kembali. Ini adalah solusi yang disarankan; pengguna mengonfirmasi keberhasilan tanpa menyebutkan solusi yang membantu. Jika masalah berlanjut, kumpulkan diagnostik berikut; belum diketahui pustaka grafis mana yang menjadi penyebabnya.

```bash
echo "Session: $XDG_SESSION_TYPE"
printenv | sort | grep -E '^(QT_|GDK_|GTK_|.*SCALE|.*DPI)'
dpkg -L ascon-kompas-kactivation-v25 | grep -E '/bin/|\.desktop$|\.sh$'
```

Setelah aktivasi, buat sebuah komponen, lakukan ekstrusi sederhana, simpan dan buka kembali berkas untuk memeriksa instalasi Anda. Jika gagal dijalankan, periksa first-launch.log. Repositori ini hanya menyediakan petunjuk, bukan berkas program ASCON atau kunci lisensi.

## Referensi resmi

- [ASCON — Linux installation / Astra Linux (PDF)](https://kompas.ru/source/documents/2026/Home/Install_astralinux.pdf)

- [ASCON — Home trial activation on Linux (PDF)](https://kompas.ru/source/documents/2026/Home/activ_trial_Linux_25.pdf)

- [ASCON — Home trial terms](https://kompas.ru/kompas-3d-home/download/)

- [ASCON — KOMPAS-3D v25](https://kompas.ru/kompas-3d/v25/)
