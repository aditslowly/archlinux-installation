Error yang kamu alami saat instalasi Arch Linux di Mac Intel menggunakan iwd (iNet Wireless Daemon) dan muncul pesan seperti:

ieee80211 phy0: brcmf_inetaddr_changed: fail to get arp ip table err:-52


berkaitan dengan driver Wi-Fi Broadcom (brcmfmac), yang seringkali bermasalah di Linux, terutama di Mac karena Broadcom banyak digunakan di hardware Apple.

Penjelasan Singkat

brcmfmac adalah kernel module untuk chipset Wi-Fi Broadcom (bcm43xx).

Error -52 biasanya berarti -ENOTSUPP (operation not supported).

brcmf_inetaddr_changed adalah bagian dari proses saat Wi-Fi mencoba mengupdate konfigurasi IP.

iwd mungkin mengalami konflik atau tidak mendukung dengan baik chipset Broadcom tertentu.

Kenapa Bisa Terjadi?

Driver tidak stabil atau tidak cocok dengan firmware/hardware Broadcom milik Mac.

iwd tidak sepenuhnya kompatibel dengan brcmfmac — iwd dirancang lebih baik untuk Intel dan beberapa chipset Qualcomm/Atheros.

Firmware Broadcom (brcmfmac*-pcie.txt) mungkin tidak lengkap atau korup.

Konflik dengan DHCP/ARP saat IP address berubah (saat koneksi baru tersambung).

Solusi dan Workaround
✅ 1. Gunakan wpa_supplicant sebagai pengganti iwd

iwd kadang tidak bekerja baik dengan Broadcom. Ganti ke wpa_supplicant:

sudo pacman -S wpa_supplicant dialog dhcpcd


Konfigurasi dengan wifi-menu (dari paket netctl) atau wpa_passphrase.

✅ 2. Pastikan firmware Broadcom terinstal

Install firmware yang sesuai:

sudo pacman -S linux-firmware


Kalau masih gagal, dan kamu tahu model chipset Broadcom-nya, coba:

yay -S broadcom-wl-dkms


Untuk MacBook Broadcom lama, broadcom-wl-dkms sering lebih stabil daripada brcmfmac.

⚠️ Catatan: broadcom-wl-dkms tidak kompatibel dengan brcmfmac. Kamu harus blacklist dulu modul default:

Edit file blacklist:

sudo nano /etc/modprobe.d/blacklist-brcmfmac.conf


Isi:

blacklist brcmfmac
blacklist brcmsmac
blacklist bcma


Reboot, lalu modul baru akan digunakan.

✅ 3. Coba jalankan iwd dengan debug log

Untuk tahu lebih lanjut:

sudo iwd -d


Lihat apakah ada error lain saat koneksi terjadi.

✅ 4. Tambahkan opsi kernel untuk brcmfmac

Kadang menambahkan parameter boot dapat membantu:

Di grub (/etc/default/grub):

GRUB_CMDLINE_LINUX="... brcmfmac.sdio_wakeup=0"


Update grub:

sudo grub-mkconfig -o /boot/grub/grub.cfg

Rekomendasi Final

Untuk instalasi Arch di Mac dengan Broadcom Wi-Fi, hindari iwd, dan pakai:

broadcom-wl-dkms (driver)

wpa_supplicant (koneksi Wi-Fi)

networkmanager atau netctl (opsional)

Kalau kamu ingin bantuan untuk identifikasi model Wi-Fi yang digunakan, jalankan:

lspci -k | grep -A 3 -i "network"


dan kirim hasilnya ke aku — nanti aku bantu carikan driver yang cocok.
