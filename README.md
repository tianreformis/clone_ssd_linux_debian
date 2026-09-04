# clone_ssd_linux_debian

Script Bash untuk mem-*clone* sistem Debian yang berjalan di HDD ke SSD baru, lengkap dengan partisi terpisah untuk `/`, `/var`, `/tmp`, `/home`, `swap`, dan `/boot/efi`, serta instalasi ulang GRUB agar SSD bisa langsung di-boot.

Script ini awalnya dibuat untuk migrasi **Intel NUC6CAYH** dari HDD 500GB ke SSD 2.5" SATA 256GB, tapi bisa disesuaikan untuk perangkat lain dengan mengubah ukuran partisi di dalam script.

## ⚠️ Peringatan

- Script ini akan **menghapus total seluruh data** di disk tujuan (`$SSD`, default `/dev/sdb`).
- Pastikan device tujuan **benar-benar SSD kosong**, bukan HDD sumber atau disk lain yang masih berisi data penting.
- Disk sumber (`$HDD`, default `/dev/sda`) tidak diubah — hanya dibaca melalui `rsync`.
- Selalu backup data penting sebelum menjalankan script ini.
- Jalankan di lingkungan yang tenang (idealnya dari live CD/USB atau saat sistem sumber tidak sedang banyak menulis data), karena proses ini melakukan clone sistem yang sedang live.

## Fitur

- Instalasi otomatis dependensi yang dibutuhkan (`parted`, `gdisk`, `dosfstools`, `rsync`, `grub-efi-amd64`, `efibootmgr`).
- Konfirmasi ganda sebelum menghapus disk tujuan (mencegah salah pilih device).
- Pembuatan tabel partisi GPT baru di SSD.
- Layout partisi terpisah mengikuti struktur HDD asal:

| Partisi | Ukuran   | Filesystem | Mount point |
|---------|----------|------------|-------------|
| sdb1    | ~976M    | vfat       | `/boot/efi` |
| sdb2    | ~31.5G   | ext4       | `/`         |
| sdb3    | ~11.8G   | ext4       | `/var`      |
| sdb4    | ~3.9G    | swap       | -           |
| sdb5    | ~2.8G    | ext4       | `/tmp`      |
| sdb6    | sisa     | ext4       | `/home`     |

- Copy data menggunakan `rsync -aHAXx` (menjaga permission, hardlink, ACL, xattr, dan tidak lintas filesystem).
- Update otomatis `/etc/fstab` di SSD dengan UUID partisi baru (fstab lama di-backup ke `/tmp/fstab.old.bak`).
- Instalasi ulang GRUB via `chroot` ke SSD, plus pendaftaran entri boot baru lewat `efibootmgr`.

## Kebutuhan

- Sistem Debian/Ubuntu (atau turunannya) dengan akses `sudo`/root.
- Boot mode **UEFI** (script ini menginstal GRUB dalam mode `x86_64-efi`).
- SSD target sudah terpasang dan terdeteksi sebagai block device (misalnya via USB enclosure atau slot SATA internal kedua).
- Koneksi internet aktif (untuk `apt-get install` dependensi).

## Cara Pakai

1. Clone repo ini ke sistem yang datanya ingin di-clone:
   ```bash
   git clone https://github.com/tianreformis/clone_ssd_linux_debian.git
   cd clone_ssd_linux_debian
   ```

2. Cek nama device HDD sumber dan SSD tujuan:
   ```bash
   lsblk -o NAME,SIZE,MODEL
   ```

3. Jika perlu, sesuaikan variabel `HDD` dan `SSD` serta ukuran partisi di bagian atas `clone_ssd.sh` agar sesuai dengan disk kamu.

4. Jalankan script dengan `sudo`:
   ```bash
   sudo bash clone_ssd.sh
   ```

5. Ikuti prompt konfirmasi:
   - Ketik `YES` untuk melanjutkan setelah membaca peringatan penghapusan data.
   - Konfirmasi bahwa device yang ditampilkan memang SSD tujuan yang benar.
   - Tekan Enter di setiap jeda proses (setelah partisi & mount siap, setelah copy data selesai) sebagai checkpoint manual sebelum lanjut ke tahap berikutnya (terutama sebelum instalasi GRUB).

6. Setelah script selesai:
   ```bash
   sudo umount -R /mnt/ssdclone
   sudo poweroff
   ```
   Lalu:
   - Lepas SSD dari enclosure USB (jika proses dilakukan via USB).
   - Pasang SSD ke slot 2.5" SATA internal, lepas HDD lama.
   - Nyalakan perangkat — GRUB seharusnya boot dari SSD secara otomatis.
   - Jika tidak boot, masuk BIOS (biasanya tombol `F2`) dan pilih SSD sebagai boot device.
   - Setelah berhasil boot, verifikasi hasil clone dengan `lsblk` dan `df -h`.

## Catatan

- HDD lama tidak diubah sama sekali selama proses (hanya dibaca), sehingga aman untuk dijadikan fallback jika ada masalah pasca-clone.
- Setelah dipastikan SSD berjalan normal dalam beberapa waktu, HDD lama bisa diformat ulang atau dipakai untuk keperluan lain (misal backup tambahan).
- File `clone_ssd.txt` berisi salinan identik dari `clone_ssd.sh`, disediakan sebagai referensi/backup teks biasa.

## Lisensi

Belum ditentukan. Tambahkan file `LICENSE` sesuai kebutuhan (misalnya MIT) jika project ini ingin dibagikan secara publik.
