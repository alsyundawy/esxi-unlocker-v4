# macOS Unlocker V4 untuk VMware ESXi

## PENTING: Pembaruan Keamanan

VMware telah [mengumumkan](https://www.vmware.com/security/advisories/VMSA-2023-0024.html) dan memperbaiki kerentanan di
VMware Tools pada sistem tamu macOS, Linux, dan Windows. Harap pastikan bahwa Anda memperbarui alat tamu (guest tools) yang dapat
ditemukan di <https://vmware.com/go/tools>.

## Unlocker 2007-2023

Proyek ini sekarang diarsipkan.

Unlocker akan terus berjalan karena hanya ada sedikit perubahan pada kode VMware dalam beberapa tahun terakhir.
Saya telah menghentikan pengembangan karena saya tidak lagi menggunakan VMware tetapi akan dengan senang hati merujuk ke fork jika seseorang
mengirimkan saya email dengan detail yang relevan.

Ada juga [Auto Unlocker](https://github.com/paolo-projects/auto-unlocker) yang masih aktif.

## 1. Pendahuluan

Unlocker 4 dirancang untuk VMware ESXi 7 dan ESXi 8.

Unlocker mengaktifkan tanda (flags) tertentu dan tabel data yang diperlukan untuk melihat jenis macOS saat mengatur
jenis OS tamu, dan memodifikasi implementasi perangkat pengontrol SMC virtual. Kemampuan ini biasanya
diekspos di Fusion dan ESXi ketika berjalan di perangkat keras Apple.

Kode patch melakukan modifikasi berikut tergantung pada produk yang di-patch:

- Mem-patch vmx dan turunannya untuk memungkinkan macOS melakukan boot
- Mem-patch libvmkctl.so untuk memungkinkan vCenter melakukan boot tamu macOS

Penting untuk dipahami bahwa Unlocker tidak dapat menambahkan kemampuan baru apa pun ke VMware ESXi
tetapi mengaktifkan dukungan untuk macOS yang dinonaktifkan di produk VMware saat dijalankan pada perangkat keras non-Apple.

Unlocker tidak dapat:

- menambahkan dukungan untuk versi macOS yang baru
- menambahkan dukungan GPU Apple yang diparavirtualisasi
- menambahkan dukungan CPU AMD

atau fitur lain apa pun yang belum ada dalam kode yang dikompilasi VMware.

## 2. Menginstal Patcher

### Catatan Penting

ESXi unlocker perlu dijalankan setiap kali Server ESXi ditingkatkan (di-upgrade).
Sebaiknya alihkan juga ESXi ke mode Pemeliharaan (Maintenance mode) dan pastikan tidak ada VM yang berjalan.

Kode ini ditulis dengan Python dan tidak memiliki prasyarat khusus dan dapat berjalan langsung dari unduhan rilis zip.

### Langkah-langkah

- Unduh rilis biner dari <https://github.com/alsyundawy/esxi-unlocker-v4/releases>
- Ekstrak arsip ke sebuah folder di datastore server ESXi
- Arahkan ke folder dengan file yang diekstrak

Anda kemudian perlu menjalankan salah satu perintah berikut untuk mem-patch atau membatalkan patch perangkat lunak ESXi.

- `unlock` - menerapkan patch ke VMware ESXi
- `relock` - menghapus patch dari VMware ESXi
- `check`  - memeriksa status patch dari instalasi VMware Anda

## 3. Hak Cipta

Hak Cipta (c) 2007-2026 David Parsons
Hak Cipta (c) 2024-2026 Harry DS Alsyundawy - Alsyundawy IT Solution

Kredit harus diberikan kepada penulis asli saat mendistribusikan ulang atau memodifikasi proyek ini.

## 4. Ucapan Terima Kasih

Terima kasih kepada Zenith432 karena awalnya membangun C++ Unlocker dan Mac Son of Knife
(MSoK) untuk semua pengujian dan dukungan.

Terima kasih juga kepada Sam B karena menemukan solusi untuk ESXi 6 dan membantu saya dengan
keahlian debugging. Sam juga menulis kode untuk mem-patch file ESXi ELF dan
memodifikasi kode Unlocker untuk berjalan di Python 3 di lingkungan ESXi 6.5.

Terima kasih kepada lucaskamp karena menguji Unlocker ESXi versi 4 yang baru.

## 5. Log Perubahan (Changelog)

Silakan lihat file [CHANGELOG-ID.md](CHANGELOG-ID.md) untuk melihat daftar lengkap perubahan dari setiap versi.
