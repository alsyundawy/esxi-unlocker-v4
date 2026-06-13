# macOS Unlocker V4 untuk VMware ESXi

## PENTING: Catatan keamanan dan lisensi

Gunakan proyek ini hanya pada sistem yang Anda miliki atau kelola sendiri dan hanya jika lisensi VMware serta Apple/macOS yang Anda gunakan mengizinkannya. Proyek ini mem-patch biner ESXi dan dapat membuat host berada di luar dukungan resmi VMware/Broadcom. Jangan gunakan pada host produksi.

VMware/Broadcom dan proyek open-vm-tools masih menerbitkan pembaruan VMware Tools. Pada tahap pembaruan 2026 untuk fork ini, VMware Tools/open-vm-tools 13.1.0 build 25218885 menjadi generasi tools terbaru yang dijadikan referensi, dan guest tools sebaiknya selalu diperbarui untuk keamanan serta kompatibilitas.

## Unlocker 2007-2026

Proyek asli DrDonk Unlocker sekarang berstatus complete/archived untuk VMware Workstation/Player. Repositori ini adalah fork/port khusus ESXi yang mempertahankan workflow shell/Python ESXi secara terpisah.

Perbedaan penting:

- Upstream DrDonk/unlocker 4.x berbasis Go dan menargetkan VMware Workstation/Player.
- Fork ini menargetkan VMware ESXi dan memakai mekanisme bootbank/VMTAR khusus ESXi.
- Logika upstream hanya dijadikan referensi pada bagian konsep patch yang memang relevan.

## 1. Pendahuluan

Unlocker 4 untuk ESXi ditujukan untuk host VMware ESXi 6.7, 7.x, dan 8.x yang kompatibel, termasuk era build ESXi 8.0 U3.

Unlocker mengaktifkan flag dan tabel data tertentu yang diperlukan agar tipe guest OS macOS terlihat, serta memodifikasi implementasi virtual SMC controller. Kemampuan ini normalnya diekspos saat VMware berjalan pada hardware Apple yang didukung.

Kode patch melakukan modifikasi khusus ESXi berikut:

- Mem-patch `vmx` dan `vmx-debug` agar guest macOS dapat boot.
- Mem-patch `vmx-stats` hanya jika file tersebut ada dan tidak kosong; pada beberapa build ESXi 8.x file ini tidak ada atau hanya stub 0-byte.
- Mem-patch `libvmkctl.so` hanya jika tersedia, untuk membantu perilaku `smcPresent` pada sisi vCenter/hostd yang diperlukan oleh sebagian workflow guest macOS.
- Membuat dan mendaftarkan `apple.v00` sebagai modul bootbank ESXi.

Unlocker tidak dapat menambahkan kapabilitas yang memang tidak ada dalam kode VMware yang sudah dikompilasi. Unlocker tidak dapat:

- Menambahkan dukungan untuk versi macOS yang tidak didukung oleh build VMware dasar.
- Menambahkan dukungan GPU Apple paravirtualized.
- Menambahkan fitur CPU yang tidak tersedia pada host fisik.
- Memberikan dukungan resmi CPU AMD.

## 2. Menginstal patcher

### Catatan penting

Jalankan ulang ESXi unlocker setiap kali ESXi di-upgrade atau dipatch, karena file biner ESXi dan modul bootbank dapat berubah.

Sebelum menjalankan `unlock`:

- Masukkan host ESXi ke Maintenance Mode jika memungkinkan.
- Matikan atau migrasikan semua VM yang berjalan.
- Pastikan tersedia minimal 400 MB ruang kosong di folder/datastore tempat unlocker diekstrak.
- Siapkan jalur rollback: jalankan `relock`, reboot, lalu jalankan ulang `unlock` setelah update host jika `check` melaporkan mismatch.

Versi 4.0.7b sekarang menolak patch jika masih ada VM berjalan yang terdeteksi melalui `esxcli vm process list`. Skrip juga memberi peringatan jika host tampak belum masuk Maintenance Mode.

### Langkah-langkah

1. Unduh arsip rilis dari halaman rilis fork ESXi Unlocker Anda.
2. Ekstrak arsip ke folder datastore pada host ESXi.
3. Buka ESXi shell atau sesi SSH sebagai root.
4. Masuk ke folder hasil ekstrak.
5. Jalankan salah satu perintah berikut.

```sh
./unlock     # menerapkan patch unlock ESXi
./relock     # menghapus modul bootbank unlock ESXi
./check      # memverifikasi status patch saat ini
```

Untuk tracing shell verbose:

```sh
./unlock -v
./relock -v
```

## 3. File

| File | Fungsi |
|---|---|
| `unlock` | Menyiapkan biner ESXi, menerapkan patch, membuat `apple.v00`, dan mendaftarkan modul bootbank. |
| `relock` | Menghapus `apple.v00` dari konfigurasi bootbank ESXi. |
| `check` | Mengecek kecocokan versi ESXi, status load `apple.v00`, status patch vmx, dan status opsional `libvmkctl.so`. |
| `patchsmc` | Tool Python untuk patch/check/dump biner `vmx` ESXi. |
| `patchvmkctl` | Tool Python untuk patch/check opsional `libvmkctl.so`. |
| `checksmc` | Wrapper untuk `patchsmc check`. |
| `checkvmkctl` | Wrapper untuk `patchvmkctl check`. |
| `dumpsmc` | Wrapper untuk `patchsmc dump`. |

## 4. Hak cipta

Hak Cipta (c) 2007-2026 David Parsons  
Hak Cipta (c) 2024-2026 Harry DS Alsyundawy - Alsyundawy IT Solution

Kredit harus diberikan kepada penulis asli saat mendistribusikan ulang atau memodifikasi proyek ini.

## 5. Ucapan terima kasih

Terima kasih kepada Zenith432 karena awalnya membangun C++ Unlocker dan Mac Son of Knife (MSoK) untuk seluruh pengujian dan dukungan.

Terima kasih juga kepada Sam B karena menemukan solusi untuk ESXi 6 dan membantu proses debugging. Sam juga menulis kode untuk mem-patch file ELF ESXi dan memodifikasi kode Unlocker agar berjalan dengan Python 3 di lingkungan ESXi 6.5.

Terima kasih kepada lucaskamp dan para penguji lain yang membantu validasi perilaku Unlocker versi 4.

## 6. Log perubahan

Lihat [CHANGELOG-ID.md](CHANGELOG-ID.md) untuk daftar lengkap perubahan setiap versi.
