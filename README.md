# A STEP TO INSTALL ARCH LINUX ON YOUR MAC
### Persiapan Sebelum Instalasi

Sebelum memulai instalasi Arch Linux pada MacBook Pro 2015 (yang biasanya menggunakan prosesor Intel), pastikan Anda memiliki:

- **USB flash drive** minimal 8GB untuk bootable installer.
- **Backup data**: Instalasi ini akan menghapus semua data di drive target. Backup file penting dari macOS.
- **Koneksi internet**: Diperlukan untuk mengunduh file ISO dan selama instalasi (kecuali untuk WiFi, yang akan kita atasi).
- **MacBook Pro 2015 spesifikasi**: Model ini biasanya memiliki Broadcom BCM4360 WiFi chip, yang memerlukan driver khusus (broadcom-wl). Instalasi akan fokus pada itu.

Catatan: Arch Linux adalah distribusi rolling-release, jadi pastikan Anda menggunakan ISO terbaru (versi 2025.08.01 atau lebih baru pada Agustus 2025). Proses ini melibatkan dual-boot atau penggantian macOS sepenuhnya; saya asumsikan Anda ingin menggantikan macOS untuk kesederhanaan, tapi langkahnya bisa disesuaikan untuk dual-boot.

### Langkah 1: Unduh dan Buat Bootable USB Arch Linux

1. Kunjungi situs resmi Arch Linux: [https://archlinux.org/download/](https://archlinux.org/download/).
2. Unduh file ISO terbaru (archlinux-2025.08.01-x86_64.iso atau yang terbaru).
3. Di macOS, gunakan Terminal untuk membuat bootable USB:
   - Masukkan USB drive.
   - Buka Terminal dan jalankan perintah berikut (ganti `/dev/diskX` dengan identifier USB Anda; cek dengan `diskutil list`):
     ```
     diskutil list
     diskutil unmountDisk /dev/diskX
     sudo dd if=/path/to/archlinux.iso of=/dev/rdiskX bs=1m
     ```
     - Ganti `/path/to/archlinux.iso` dengan lokasi file ISO.
     - Tunggu proses selesai (bisa memakan waktu 10-20 menit).
4. Eject USB: `diskutil eject /dev/diskX`.

### Langkah 2: Boot dari USB dan Mulai Archinstall

1. Matikan MacBook Pro, masukkan USB, lalu nyalakan sambil tahan tombol **Option (⌥)** untuk masuk ke Startup Manager.
2. Pilih USB drive (biasanya muncul sebagai "EFI Boot").
3. Setelah boot ke live environment Arch Linux (prompt root@archiso), koneksi internet sementara bisa menggunakan Ethernet jika tersedia. Jika tidak, lanjutkan offline dan konfigurasi WiFi nanti.
4. Jalankan Archinstall:
   ```
   archinstall
   ```
   - Ini akan membuka wizard interaktif berbasis teks.

### Langkah 3: Konfigurasi Archinstall

Ikuti wizard Archinstall langkah demi langkah. Berikut konfigurasi kunci untuk MacBook Pro 2015:

- **Keyboard layout**: Pilih `us` atau sesuai preferensi (untuk Indonesia: `id`).
- **Language**: Pilih `en_US.UTF-8` (atau tambahkan `id_ID.UTF-8` jika diperlukan).
- **Disk configuration**:
  - Pilih **Manual partitioning** jika ingin custom, atau **Wipe disk** untuk menghapus seluruh drive.
  - Untuk MacBook Pro 2015 (biasanya SSD 256GB+):
    - Buat partisi EFI: 512MB, type FAT32, mount ke `/boot/efi` (flag: boot, esp).
    - Partisi root (/): Sisa ruang, type ext4, mount ke `/`.
    - Opsional: Swap 4-8GB jika RAM <16GB.
  - Gunakan `cfdisk` atau `fdisk` jika manual. Pastikan GPT partition table (bukan MBR).
- **Network configuration**:
  - Pilih **NetworkManager** sebagai service (akan diaktifkan otomatis).
  - Saat ini, WiFi mungkin tidak terdeteksi di live env karena driver Broadcom. Skip dulu; kita atasi post-install.
- **Bootloader**: Pilih **systemd-boot** (direkomendasikan untuk Mac hardware; lebih kompatibel daripada GRUB).
- **Audio**: Pilih **pipewire** (modern dan kompatibel).
- **Profile**: Pilih **desktop** jika ingin GUI (misalnya GNOME atau KDE), atau **minimal** untuk CLI-only.
- **Hostname**: Masukkan nama seperti `arch-macbook`.
- **Root password**: Set password root yang kuat.
- **User account**: Buat user non-root (misalnya `yourusername`), set password, dan tambahkan ke grup `wheel` untuk sudo.
- **Additional packages**: Tambahkan `broadcom-wl-dkms` untuk WiFi (tapi install manual post-boot jika gagal di sini; lihat langkah selanjutnya).
- Review konfigurasi, lalu pilih **Install**.

Instalasi akan memakan waktu 10-30 menit. Setelah selesai, reboot dan lepaskan USB.

### Langkah 4: Post-Install - Konfigurasi WiFi Broadcom

MacBook Pro 2015 menggunakan chip WiFi Broadcom BCM4360, yang tidak didukung out-of-box di Linux. Driver `broadcom-wl` diperlukan, dan memerlukan AUR atau repo khusus. Asumsikan Anda boot ke sistem baru (jika WiFi tidak work, gunakan Ethernet atau tethering dari ponsel untuk internet).

1. Boot ke Arch Linux baru. Login sebagai root atau user dengan sudo.
2. Update sistem:
   ```
   sudo pacman -Syu
   ```
3. Install prasyarat untuk driver Broadcom:
   ```
   sudo pacman -S base-devel linux-headers dkms
   ```
4. Install driver Broadcom-WL:
   - Gunakan AUR helper seperti `yay` (install dulu jika belum):
     ```
     sudo pacman -S --needed git base-devel
     git clone https://aur.archlinux.org/yay.git
     cd yay
     makepkg -si
     cd ..
     ```
   - Kemudian install driver:
     ```
     yay -S broadcom-wl-dkms
     ```
     - Ini akan compile driver untuk kernel Anda. Jika error, pastikan kernel headers terinstall.
5. Reboot:
   ```
   sudo reboot
   ```
6. Setelah reboot, aktifkan NetworkManager:
   ```
   sudo systemctl enable --now NetworkManager
   ```
7. Scan dan connect WiFi:
   ```
   nmcli device wifi list
   nmcli device wifi connect "SSID-ANDA" password "PASSWORD"
   ```
   - Ganti `SSID-ANDA` dan `PASSWORD` sesuai jaringan Anda.
   - Jika GUI, install `nmtui` dengan `sudo pacman -S networkmanager` dan jalankan `nmtui`.

Jika driver gagal compile (jarang, tapi bisa karena kernel update), coba versi alternatif:
```
yay -S broadcom-wl
```
Atau blacklist modul konflik: Edit `/etc/modprobe.d/blacklist.conf` tambahkan `blacklist brcmfmac`, lalu reboot.

### Troubleshooting Umum untuk MacBook Pro 2015

- **Trackpad/Keyboard**: Install `libinput` (`sudo pacman -S xf86-input-libinput`) dan konfigurasi di Xorg/Wayland.
- **Brightness/Keyboard backlight**: Install `apple-bce-dkms` dari AUR untuk backlight keyboard.
- **Suspend/Hibernate**: Masalah umum di Mac; edit GRUB atau systemd untuk fix (cari di Arch Wiki).
- **No internet di live env**: Gunakan Ethernet adapter USB jika punya, atau download driver offline.
- **Boot issues**: Jika tidak boot, masuk ke live USB lagi, chroot ke instalasi (`arch-chroot /mnt`), dan install ulang bootloader: `bootctl --path=/boot install`.
- **Dual-boot dengan macOS**: Jangan wipe disk; resize partisi macOS dengan Disk Utility dulu, lalu buat partisi baru untuk Arch di wizard.

### Sumber dan Tips Tambahan

- Ikuti Arch Wiki resmi untuk detail lebih lanjut: [https://wiki.archlinux.org/title/Installation_guide](https://wiki.archlinux.org/title/Installation_guide) dan [https://wiki.archlinux.org/title/Broadcom_wireless](https://wiki.archlinux.org/title/Broadcom_wireless).
- Untuk Mac-specific: [https://wiki.archlinux.org/title/Mac](https://wiki.archlinux.org/title/Mac).
- Archinstall guide: [https://wiki.archlinux.org/title/archinstall](https://wiki.archlinux.org/title/archinstall).
- Jika stuck, forum Arch atau Reddit r/archlinux bisa membantu. Pastikan kernel LTS jika stabilitas penting (`sudo pacman -S linux-lts`).

Proses ini memerlukan pengetahuan dasar CLI; jika pemula, pertimbangkan distribusi lain seperti Ubuntu. Selamat mencoba! Jika ada error spesifik, berikan detail untuk bantuan lebih lanjut.
