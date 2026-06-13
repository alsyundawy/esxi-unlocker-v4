# Catatan Pemecahan Masalah (Troubleshooting)

## Pengecek Patch (Patch checker)
Sangat penting untuk mem-patch ulang ESXi setelah pembaruan atau perbaikan sistem telah diterapkan. Status dari patch unlocker
dapat diperiksa menggunakan perintah `check` yang ada di dalam folder unlocker.

Jika patch telah terinstal dan cocok dengan versi ESXi saat ini, outputnya akan terlihat seperti ini, tetapi nomor versi
mungkin berbeda pada host Anda bergantung pada patch apa saja yang telah diinstal.
```text
VMware ESXi Unlocker 4.0.3
==========================
(c) 2011-2022 David Parsons

Checking unlocker...
Current version of ESXi: VMware ESXi 7.0.3 build-20328353
Patch built for ESXi: VMware ESXi 7.0.3 build-20328353
Checking VMTAR loaded...
apple.v00 loaded
Checking vmx vSMC status...
/bin/vmx
/bin/vmx-debug
/bin/vmx-stats
vmx patched
Checking smcPresent status...
smcPresent = true
```
Jika patch telah terinstal tetapi _*TIDAK*_ cocok dengan versi ESXi saat ini, outputnya akan terlihat seperti ini dan sebuah pesan kesalahan (error)
akan ditampilkan. Anda harus mengunci kembali (relock), melakukan booting ulang, dan membuka kunci (unlock) host untuk memastikan patch diterapkan ke versi yang benar.

```text
VMware ESXi Unlocker 4.0.3
==========================
(c) 2011-2022 David Parsons

Checking unlocker...
Current version of ESXi: VMware ESXi 7.0.3 build-20328353
Patch built for ESXi: VMware ESXi 7.0.3 build-20036589
>>> ERROR: Mis-matched files please relock/unlock to update patches <<<
```

## VMware vCenter
Mulai dari versi 4.0.3, unlocker akan memungkinkan tamu macOS dijalankan melalui vCenter pada host yang telah di-patch. Jika Anda mendapatkan error
di vCenter, Anda mungkin perlu melakukan hal berikut:
1. Putuskan dan sambungkan kembali (disconnect & reconnect) host ESXi di vCenter
2. Hapus dan tambahkan kembali host ESXi di vCenter

## Menetapkan resolusi Tamu macOS tertentu

Menginstal VMWare Tools seharusnya memungkinkan Anda untuk memilih berbagai mode video. Jika Anda telah menginstalnya namun resolusi masih belum berubah, Anda dapat mencoba cara ini. Buka Terminal dan jalankan:

`sudo /Library/Application Support/VMware Tools/vmware-resolutionSet <width> <height>`

dengan `<width>` dan `<height>` adalah ukuran piksel yang Anda inginkan. Misalnya untuk mendapatkan 1440x900:

`sudo /Library/Application Support/VMware Tools/vmware-resolutionSet 1440 900`

## CPU AMD
ESXi Unlocker tidak menambahkan dukungan untuk CPU AMD, hal tersebut harus dilakukan melalui pengaturan untuk tamu macOS yang berjalan pada
CPU AMD modern. Pengaturan ini mungkin memungkinkan macOS untuk berjalan berdasarkan tes yang dilaporkan kembali ke proyek, tetapi tidak ada
dukungan resmi untuk CPU AMD pada unlocker.

Kernel AMD macOS yang telah dimodifikasi atau OpenCore harus digunakan untuk berjalan pada sistem AMD yang lebih lawas.

1. Baca artikel basis pengetahuan (KB) ini untuk mempelajari cara mengedit file VMX tamu dengan aman https://kb.vmware.com/s/article/2057902
2. Tambahkan baris-baris berikut ke dalam file VMX:
```text
cpuid.0.eax = "0000:0000:0000:0000:0000:0000:0000:1011"
cpuid.0.ebx = "0111:0101:0110:1110:0110:0101:0100:0111"
cpuid.0.ecx = "0110:1100:0110:0101:0111:0100:0110:1110"
cpuid.0.edx = "0100:1001:0110:0101:0110:1110:0110:1001"
cpuid.1.eax = "0000:0000:0000:0001:0000:0110:0111:0001"
cpuid.1.ebx = "0000:0010:0000:0001:0000:1000:0000:0000"
cpuid.1.ecx = "1000:0010:1001:1000:0010:0010:0000:0011"
cpuid.1.edx = "0000:0111:1000:1011:1111:1011:1111:1111"
vhv.enable = "FALSE"
vpmc.enable = "FALSE"
vvtd.enable = "FALSE"
```
3. Pastikan tidak ada baris duplikat (ganda) dalam file VMX, karena tamu tidak akan mulai dan kesalahan kamus akan
   ditampilkan oleh VMware.
4. Anda sekarang dapat menginstal dan menjalankan macOS sebagai sistem tamu.
