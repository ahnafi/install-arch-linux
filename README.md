# Dokumentasi: Instalasi Arch Linux (versi singkat dan mudah dibaca)

Panduan ini adalah catatan pengalaman saya saat menginstal Arch Linux di lingkungan virtual. Instalasi yang Anda lakukan bisa berbeda bergantung pada perangkat dan konfigurasi. Selalu utamakan membaca dokumentasi resmi Arch Linux di: https://wiki.archlinux.org/title/Installation_guide

Panduan ini ditulis agar:
- Lebih ringkas dan mudah dipahami.
- Menunjukkan langkah-langkah utama dari boot live environment sampai partisi disk.
- Menyertakan contoh perintah yang sering dipakai.

Jika Anda baru pada Arch: siapkan waktu, baca perlahan, dan jangan ragu cari referensi tambahan di wiki Arch.

---

## 1. Masuk ke live environment
Boot dari media instalasi (ISO) dan masuk ke live environment. Di sini Anda akan menjalankan semua perintah instalasi.

---

## 2. Atur layout keyboard dan font konsol

Periksa daftar `keymaps` yang tersedia:
install-arch/README.md#L201-210
```
localectl list-keymaps
```

Contoh: pakai layout `us`:
install-arch/README.md#L211-215
```
loadkeys us
```

Jika ingin ukuran font konsol lebih besar (opsional), lihat dokumentasi `Console fonts`. Contoh mengganti font:
install-arch/README.md#L216-220
```
setfont ter-132b
```

Catatan: mengganti font hanya memengaruhi tampilan di konsol, bukan lingkungan grafis.

---

## 3. Verifikasi mode boot (UEFI atau BIOS)

Periksa apakah sistem boot dalam mode UEFI atau BIOS:
install-arch/README.md#L221-225
```
cat /sys/firmware/efi/fw_platform_size
```

Interpretasi hasil:
- Mengembalikan `64` → sedang boot dalam mode UEFI 64-bit.
- Mengembalikan `32` → UEFI 32-bit (jarang dan bisa membatasi pilihan boot loader).
- Mengembalikan `No such file or directory` → kemungkinan boot lewat BIOS/CSM.

Jika mode boot bukan yang Anda inginkan, cek manual motherboard/VM untuk mengubah pengaturan boot.

---

## 4. Sambungkan ke internet

Langkah-langkah umum untuk membuat koneksi di live environment:

1. Pastikan interface jaringan terdeteksi:
install-arch/README.md#L226-230
```
ip link
```

2. Untuk perangkat nirkabel: pastikan tidak diblokir oleh `rfkill` (cek jika perlu).
3. Sambungkan:
   - Ethernet: cukup colok kabel.
   - Wi‑Fi: gunakan `iwctl` atau tool lain untuk autentikasi.
   - Modem seluler: gunakan `mmcli` jika diperlukan.
4. Konfigurasi IP:
   - DHCP umumnya bekerja otomatis (systemd-networkd / systemd-resolved).
   - Untuk IP statis, ikuti panduan konfigurasi jaringan di wiki Arch.
5. Verifikasi koneksi dengan `ping`:
install-arch/README.md#L231-235
```
ping -c 4 ping.archlinux.org
```

Catatan: gunakan `Ctrl+C` untuk menghentikan `ping` jika perlu.

---

## 5. Perbarui waktu sistem

Pastikan jam sistem akurat sebelum menginstal:
install-arch/README.md#L236-240
```
timedatectl set-ntp true
timedatectl status
```

Mengaktifkan NTP (jika tersedia) membantu menghindari masalah ketika membuat partisi terenkripsi atau mengatur sertifikat.

---

## 6. Partisi disk

Saat live system mengenali disk, perangkat block akan muncul seperti `/dev/sda`, `/dev/nvme0n1`, atau `/dev/mmcblk0`.

Lihat daftar disk dan partisi:
install-arch/README.md#L241-245
```
fdisk -l
```

Tampilkan pohon block devices:
install-arch/README.md#L246-250
```
lsblk
```

Untuk membuat atau mengubah partisi, gunakan alat partisi seperti `fdisk`, `cfdisk`, atau `parted`. Contoh menjalankan `fdisk`:
install-arch/README.md#L251-255
```
fdisk /dev/sdX
```
Ganti `/dev/sdX` dengan nama disk yang ingin Anda partisi (mis. `/dev/sda` atau `/dev/nvme0n1`).

Tips singkat:
- Jika menggunakan UEFI: buat partisi EFI (biasanya FAT32, ~300–512 MiB) dengan flag `boot`/`esp`.
- Untuk instalasi standar tanpa enkripsi: buat partisi root (`/`) dan (opsional) `swap` atau `home`.
- Pertimbangkan menggunakan LVM atau enkripsi (`LUKS`) kalau perlu keamanan atau fleksibilitas.

---

## 7. Langkah selanjutnya (ringkasan)

Setelah partisi selesai, langkah umum berikutnya biasanya meliputi:
1. Membuat filesystem (mis. `mkfs.fat -F32 /dev/sdX1`, `mkfs.ext4 /dev/sdX2`).
2. Mount partisi ke `/mnt`.
3. Instal paket dasar (`pacstrap /mnt base linux linux-firmware`).
4. Konfigurasi `fstab` (`genfstab -U /mnt >> /mnt/etc/fstab`).
5. Chroot ke sistem baru (`arch-chroot /mnt`) dan lanjutkan konfigurasi (timezone, locale, pengguna, boot loader).

Untuk detail lengkap masing-masing tahapan di atas, ikuti panduan resmi: https://wiki.archlinux.org/title/Installation_guide

---

## 8. Referensi dan sumber
- Panduan instalasi resmi Arch Linux: https://wiki.archlinux.org/title/Installation_guide
- Console fonts: https://wiki.archlinux.org/title/Console_fonts
- Dokumentasi networking di wiki Arch untuk pengaturan statis/lanjutan.

