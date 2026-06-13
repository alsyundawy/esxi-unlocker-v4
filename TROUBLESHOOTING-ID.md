# Catatan Pemecahan Masalah (Troubleshooting)

## Pengecek patch

Jalankan ulang `unlock` setiap kali ESXi di-upgrade atau masuk siklus patch. Update ESXi dapat mengganti `vmx`, `vmx-debug`, `libvmkctl.so`, metadata bootbank, atau tardisk yang sedang dimuat.

Cek status patch dari folder unlocker:

```sh
./check
```

Pada 4.0.7b, `check` keluar dengan status `0` jika pemeriksaan wajib lulus dan status `1` jika pemeriksaan wajib gagal. Komponen opsional seperti `vmx-stats` dan `libvmkctl.so` hanya menjadi peringatan jika tidak ada, karena memang tidak tersedia pada semua build ESXi.

Host yang sudah terpatch dengan sehat secara konsep akan terlihat seperti ini; build ESXi, nilai SHA-256, dan output komponen opsional pasti berbeda pada host Anda:

```text
VMware ESXi Unlocker 4.0.7b
===========================

Checking unlocker status...
Current ESXi version  : VMware ESXi 8.0.3 build-25429389
Patch built for ESXi  : VMware ESXi 8.0.3 build-25429389
ESXi version matches patch record.

Checking VMTAR loaded...
apple.v00 loaded.

Checking vmx vSMC patch status...
/bin/vmx contains AppleComputerInc marker.
---
PatchSMC 4.0.7b (ESXi 6.7 / 7.x / 8 U3)
...
Patch Status: True
/bin/vmx-debug contains AppleComputerInc marker.
---
Patch Status: True

Checking libvmkctl.so patch status...
Patch Status: True

Checking smcPresent status...
smcPresent = true

Check completed: required patch status looks OK.
```

Jika patch dibuat untuk versi ESXi yang berbeda, `check` akan melaporkan mismatch dan exit non-zero:

```text
>>> Version mismatch — please relock and unlock to update patches <<<
Check completed: one or more required checks failed.
```

Alur recovery yang disarankan:

```sh
./relock
reboot
# setelah reboot
./unlock
reboot
./check
```

## VM berjalan / Maintenance Mode

Versi 4.0.7b menolak menjalankan `unlock` jika `esxcli vm process list` mendeteksi masih ada VM berjalan. Matikan atau migrasikan semua VM terlebih dahulu.

Skrip juga memberi peringatan jika host tampak belum masuk Maintenance Mode. Maintenance Mode sangat disarankan karena `unlock` menyiapkan ulang dan membuat ulang modul bootbank ESXi.

## apple.v00 tidak loaded

Jika `check` menyatakan `apple.v00 not loaded`, penyebab paling umum adalah:

1. Host sudah dipatch tetapi belum reboot.
2. `BootModuleConfig.sh --add=apple.v00` gagal.
3. ESXi sudah di-upgrade dan status bootbank berubah.

Jalankan:

```sh
esxcli system visorfs tardisk list | grep apple.v00
ls -l /bootbank/apple.v00 /altbootbank/apple.v00 2>/dev/null
```

Jika modul hilang atau stale, jalankan alur relock/reboot/unlock/reboot.

## vmx atau vmx-debug tidak terpatch

`vmx` dan `vmx-debug` adalah komponen wajib. Jika salah satunya gagal, jangan anggap host sudah terpatch dengan benar.

Cek yang berguna:

```sh
./checksmc /bin/vmx
./checksmc /bin/vmx-debug
grep -a 'AppleComputerInc' /bin/vmx /bin/vmx-debug
```

Jika `patchsmc` melaporkan kondisi partial OSK patch, jangan lanjutkan patch secara paksa. Jalankan relock, reboot, pastikan biner ESXi kembali ke level patch host yang bersih, lalu jalankan `unlock` ulang.

## vmx-stats hilang atau tidak terpatch

Pada beberapa build ESXi 8.x, `/bin/vmx-stats` tidak ada atau hanya stub 0-byte. Versi 4.0.7b memperlakukan kondisi ini sebagai opsional dan non-fatal.

## libvmkctl.so hilang atau opsional

`/lib64/libvmkctl.so` tidak dijamin ada pada semua build ESXi. Jika tidak ada, patch vmkctl dilewati. Jika Anda memakai vCenter dan guest macOS gagal start dari vCenter setelah host dipatch, coba:

1. Disconnect dan reconnect host ESXi di vCenter.
2. Remove dan add ulang host ESXi di vCenter.
3. Jalankan ulang `./checkvmkctl /lib64/libvmkctl.so` jika file tersebut tersedia.

## Menetapkan resolusi guest macOS tertentu

Menginstal VMware Tools seharusnya memungkinkan pemilihan berbagai mode video. Jika resolusi masih tidak berubah, buka Terminal di dalam guest macOS dan jalankan:

```sh
sudo /Library/Application\ Support/VMware\ Tools/vmware-resolutionSet <width> <height>
```

Contoh:

```sh
sudo /Library/Application\ Support/VMware\ Tools/vmware-resolutionSet 1440 900
```

## CPU AMD

ESXi Unlocker tidak menambahkan dukungan resmi CPU AMD. Workaround AMD harus ditangani melalui konfigurasi guest, OpenCore, atau tooling macOS lain di luar proyek ini.

Jika Anda mengedit file VMX secara manual, pastikan tidak ada key duplikat. Key VMX duplikat dapat membuat VM gagal start dan menimbulkan error dictionary/configuration.
