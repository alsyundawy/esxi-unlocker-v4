# Log Perubahan (Changelog)

Semua tanggal menggunakan format Inggris DD/MM/YY.

## 13/06/26 4.0.7b

_alsyundawy (Harry DS Alsyundawy - Alsyundawy IT Solution):_

> Tahap hardening final untuk ESXi 6.7 / 7.x / 8 U3 per 13/06/26. Wrapper ESXi
> tetap bersifat fork-specific; upstream DrDonk/unlocker 4.x hanya digunakan
> sebagai referensi logika patch yang relevan, bukan sebagai rilis ESXi siap pakai.

- `unlock`: Mendefinisikan variabel warna ANSI sebelum jalur error pertama; sebelumnya deteksi `python3` dapat mencetak format error kosong karena `${RED}` dan `${RST}` baru dideklarasikan setelahnya.
- `unlock`: Menambahkan preflight command wajib ESXi (`vmware`, `esxcli`, `vmtar`, `pigz`, `BootModuleConfig.sh`, dan lain-lain) agar host tool yang hilang gagal sejak awal, bukan menghasilkan `apple.v00` rusak.
- `unlock`: Menjadikan kegagalan `vmware -v` sebagai fatal; menulis `Unknown` ke `unlock.conf` membuat `check` tidak andal setelah update host.
- `unlock`: Memvalidasi output `df -m .` sebagai angka sebelum dibandingkan, mencegah error ekspresi integer POSIX shell saat output `df` tidak sesuai.
- `unlock`: Menolak patch saat masih ada VM berjalan berdasarkan `esxcli vm process list`; menampilkan peringatan jika host tampak belum masuk Maintenance Mode.
- `unlock`: Mengganti cleanup staging `find -delete` yang tidak portabel dengan `find ... -exec rm -f {} \;` agar lebih aman pada BusyBox/ESXi.
- `unlock`: Menjadikan kegagalan `patchsmc` pada `vmx` dan `vmx-debug` sebagai fatal; `vmx-stats` dan `libvmkctl.so` tetap opsional jika tidak berlaku pada build ESXi tertentu.
- `unlock`: Mendeteksi dan menghapus `/bootbank/apple.v00` lama sebelum menambahkan modul baru untuk mengurangi risiko modul duplikat/stale.
- `check`: Menambahkan exit status non-zero saat pemeriksaan wajib gagal, sementara absennya `vmx-stats`/`libvmkctl.so` tetap diperlakukan sebagai peringatan opsional.
- `check`: Menambahkan fallback resolusi direktori skrip jika `readlink -f` tidak tersedia dan memastikan path `python3` yang terdeteksi benar-benar executable.
- `checksmc`, `checkvmkctl`, `dumpsmc`: Menambahkan hardening PATH, fallback `SCRIPT_DIR`, validasi executable `python3`, dan pengecekan argumen `${1:-}` yang lebih aman.
- `patchsmc`: Mengembalikan exit code sukses/gagal yang bermakna ke pemanggil; kegagalan patch pada biner wajib kini menghentikan `unlock`.
- `patchsmc`: Memindahkan pembuatan `.bak` setelah pengecekan already-patched dan layout biner, sehingga file yang sudah patched/tidak kompatibel tidak menghasilkan backup menyesatkan.
- `patchsmc`: Mendeteksi kondisi partial OSK patch (`OSK0` ada tetapi `OSK1` tidak ada, atau sebaliknya) dan gagal aman alih-alih memaksa re-patch buta.
- `patchsmc`: Mencari `#KEY` tabel kedua relatif terhadap `SMC_HEADER_V1`, bukan memakai `rfind()` global, untuk mencegah marker akhir yang tidak relevan terpilih.
- `patchsmc`, `patchvmkctl`: Mengembalikan permission file memakai `stat.S_IMODE(os.stat(...).st_mode)` alih-alih memberikan bit `st_mode` penuh ke `os.chmod()`.
- `patchvmkctl`: Mengembalikan exit status bermakna dan tetap memperlakukan absennya `applesmc` sebagai non-fatal untuk file yang sudah patched atau build ESXi yang tidak memerlukan patch opsional ini.
- `relock`: Menambahkan validasi `BootModuleConfig.sh` dan pelaporan gagal yang lebih tegas saat removal gagal.
- `README.md`, `README-ID.md`: Memperbarui status proyek, kompatibilitas, panduan VMware Tools 2026, dan klarifikasi fork khusus ESXi.
- `TROUBLESHOOTING.md`, `TROUBLESHOOTING-ID.md`: Memperbarui contoh ke 4.0.7b dan mendokumentasikan exit non-zero baru, guard VM berjalan, serta komponen opsional.
- `LICENSE`: Menambahkan baris copyright modifikasi 2024-2026 tanpa mengubah teks lisensi MIT.

## 10/06/26 4.0.7a

_alsyundawy (Harry DS Alsyundawy - Alsyundawy IT Solution):_

> Tinjauan kode yang komprehensif terhadap ESXi 6.7 / 7.x / 8 U3 — semua perbaikan
> berdasarkan analisis kode sumber Go dari DrDonk/unlocker 4.2.8 dan kebutuhan khusus
> ESXi.

- `unlock`: **Perbaikan kritis** — penggunaan wildcard `./tmp/bin/*` pada langkah
  `chmod` dan `tar` menyebabkan file audit `.bak` dan `.sha256` (yang dibuat oleh patchsmc/
  patchvmkctl di tahap persiapan) ikut dikemas ke dalam bootbank VMTAR `apple.v00` dan
  diterapkan bit setuid. Perbaikan: hapus file audit dengan `find -delete` sebelum
  chmod/tar; gunakan target file eksplisit untuk `chmod 4555`.
- `unlock`: Menambahkan penjaga (guard) apabila `/lib64/libvmkctl.so` tidak ada (tidak terdapat di semua
  build ESXi); melewati patch vmkctl dengan anggun alih-alih gagal.
- `unlock`: Menambahkan loop deteksi biner `python3` yang mencakup ESXi 6.7 (`venv-65`),
  ESXi 7.x (`venv-vsphere`), dan ESXi 8.x (`/usr/bin/python3`). Skrip sekarang
  mengekspor `$PYTHON3` dan menggunakannya untuk semua pemanggilan Python.
- `patchsmc`: Memperbaiki variabel loop yang tidak digunakan `i` → `_` di `dumpkeys()`.
- `patchsmc`: Secara eksplisit menggunakan `os.chmod(name, orig_mode)` setelah mem-patch biner
  vmx — mencerminkan pemanggilan `CopyFile os.Chmod()` di hulu (upstream) Go pada `vmw.go`.
- `patchvmkctl`: Secara eksplisit menggunakan `os.chmod(name, orig_mode)` setelah menulis konten yang
  di-patch (membuka `'wb'` mungkin tidak mempertahankan bit izin di semua driver sistem file ESXi).
- `patchvmkctl`: Memindahkan pemanggilan `_backup_file()` ke setelah pencarian `APPLESMC` sehingga
  file `.bak` tidak dibuat jika file tersebut sudah di-patch atau tidak kompatibel.
- `checksmc`, `checkvmkctl`, `dumpsmc`: Menambahkan resolusi `SCRIPT_DIR` melalui
  `readlink -f` sehingga wrapper dapat bekerja dari direktori kerja mana pun (bukan hanya ketika
  dipanggil dari direktori unlocker). Menambahkan validasi argumen `$1`. Menggunakan
  jalur `$PYTHON3` yang terdeteksi.
- `check`: Menambahkan deteksi `SCRIPT_DIR` + `$PYTHON3` (sama seperti wrapper di atas).
- `check`: Memperbaiki `grep -qil` pada grep busybox ESXi — opsi `-l` (daftar nama file) dan
  `-q` (diam/quiet) saling bertentangan; diganti dengan loop `grep -q` per biner.
- `check`: Menjaga kegagalan `vmware -v` dengan `|| ESXI=''` dan pesan peringatan.
- `check`: Menambahkan pemeriksaan keberadaan `/lib64/libvmkctl.so` sebelum memanggil patchvmkctl.
- `check`: Memperbaiki peringatan POSIX sh (ShellCheck) terkait dengan `printf '---\n'` dengan
  memformat string secara eksplisit dan aman yang dimulai dengan tanda hubung: `printf '%s\n' '---'`.

## 10/06/26 4.0.7

_alsyundawy (Harry DS Alsyundawy - Alsyundawy IT Solution):_

> Referensi: DrDonk/unlocker **4.2.8** (14/05/26) — menganalisis biner Linux
> dan sumber Go untuk menyelaraskan implementasi ESXi Python/Shell dengan logika hulu (upstream).

- `patchsmc`: Mengadopsi literal byte OSK mentah (`osk0Data`/`osk1Data`) dari
  `smc.go` versi Go alih-alih pengkodean rot13 — lebih transparan dan dapat diaudit.
- `patchsmc`: Menambahkan checksum SHA-256 sebelum/sesudah patch (mencerminkan fitur
  `WriteHashes()` dari upstream yang diperkenalkan pada Go 4.1.0).
- `patchsmc`: Menambahkan pencadangan `.bak` dari setiap file sebelum mem-patch (mencerminkan
  `Backup()` dari upstream di `vmw.go`).
- `patchsmc`: Memperbaiki pemeriksaan "sudah di-patch" menggunakan pencarian byte data OSK mentah
  (lebih cepat, tidak memerlukan proses "seek") — mencerminkan `checkPatch()` dari upstream di `smc.go`.
- `patchsmc`: Meningkatkan penjaga (guard) loop ELF RELA (`find()` mengembalikan -1 → break).
- `patchvmkctl`: Menambahkan SHA-256 sebelum/sesudah patch (`WriteHashes` hulu).
- `patchvmkctl`: Menambahkan pencadangan `.bak` sebelum mem-patch (`Backup()` hulu).
- `patchvmkctl`: Mencetak string `before -> after` (mencerminkan `PatchVMKCTL()` hulu
  di `vmkctl.go` versi Go).
- `patchvmkctl`: Menggunakan `bytearray` untuk patch di memori (lebih aman dari "seek" `r+b`).
- `unlock`: Menambahkan deteksi versi ESXi dan menampilkannya saat memulai.
- `unlock`: Mengadopsi pemeriksaan keberadaan `vmx-stats` upstream 4.2.4+ di shell:
  hanya salin dan patch jika file tersebut tidak kosong dan merupakan file biasa (ESXi 8.x
  menyediakan file stub berukuran 0-byte untuk `vmx-stats`).
- `check`: Menambahkan pemanggilan `checksmc` per-biner (SHA-256 + status patch).
- `check`: Menambahkan status patch `libvmkctl.so` via `patchvmkctl check`.
- Semua skrip: Memperbarui string versi menjadi `4.0.7`.
- Semua skrip: Menambahkan warna ANSI `YLW` (kuning) untuk peringatan non-fatal.

## 10/06/26 4.0.6

_alsyundawy (Harry DS Alsyundawy - Alsyundawy IT Solution):_

- `unlock`: Memperbaiki **bug kritis** — argumen posisi `$1` yang salah ditambahkan
  ke perintah `cp`, `rm`, `vmtar`, `pigz`, dan `mv`, yang menyebabkan semua operasi penyiapan
  file dan pembersihan gagal secara diam-diam atau menggunakan nama file yang salah.
- `unlock`: Mengganti `read -p` dengan idiom POSIX yang aman `printf … ; read -r`.
- `unlock`: Menggunakan `( subshell )` untuk langkah `tar` untuk menghindari perlunya kembali dengan `cd`.
- `unlock`: Menggunakan glob `./*` dalam `tar` agar nama file yang dimulai dengan `-` tetap aman.
- `unlock`: Menambahkan penjaga (guard) `cd … || exit` sesuai praktik terbaik POSIX sh.
- `unlock`: Menggunakan pola `printf '%s'` untuk variabel dalam format string.
- `relock`: Menambahkan `exit 0` yang hilang. Mengganti `read -p` dengan idiom yang aman pada POSIX.
- `relock`: Menambahkan pesan informasi ketika pengguna memilih untuk tidak melakukan boot ulang (reboot).
- `check`: Mengganti semua konstruksi bash `[[ ]]` dengan POSIX `[ ]`.
- `check`: Mengganti `==` dengan `=` di dalam `[ ]` untuk portabilitas POSIX sh.
- `check`: Menggunakan tes perintah langsung dalam `if` alih-alih memeriksa `$?`.
- `check`: Menggunakan pola `printf '%s'` untuk variabel dalam format string.
- `checksmc`, `checkvmkctl`, `dumpsmc`: Menggunakan tanda kutip pada `$1` untuk mencegah pemisahan kata (word splitting).
- `patchvmkctl`: Mengubah shebang dari `python` menjadi `python3`.
- `patchvmkctl`: Mengganti pemanggilan `open()` secara langsung dengan manajer konteks `with`.
- `patchsmc`: Menghapus pemanggilan `vmx.close()` yang berlebihan di dalam/setelah blok `with`.
- `patchsmc`, `patchvmkctl`: Menambahkan docstring modul yang berisi hak cipta dan log perubahan.
- Semua skrip: Menambahkan header blok dokumentasi dengan instruksi penggunaan, hak cipta, dan log perubahan.
- Semua skrip: Memperbarui string versi menjadi `4.0.6`.
- `README.md`: Memperbaiki semua peringatan markdownlint (MD001, MD009, MD012, MD022,
  MD026, MD034). Memperbaiki salah ketik: `developemnt`, `implmentation`, `capabiltiites`,
  `Maintanence`. Menambahkan bagian 3 yang hilang (Hak Cipta). Membungkus URL telanjang.

## 03/03/23 4.0.6

### Ini adalah rilis terakhir dari ESXi Unlocker.

_drdonk:_
* Merapikan kode
* Menambahkan penjelasan bahwa unlocker tidak dapat menambahkan dukungan AMD tetapi beberapa pengaturan di file VMX mungkin berfungsi

## 09/01/23 4.0.5
_drdonk:_
* Memperbaiki kesalahan izin dengan benar kali ini!

## 03/01/23 4.0.4
_drdonk:_
* Perintah `check` dengan benar melaporkan jika sistem belum di-patch
* Perintah `unlock` memeriksa ruang disk kosong sebelum mem-patch
* Perintah `unlock` memeriksa instalasi V3 sebelumnya sebelum mem-patch
* Memperbaiki kesalahan jika libvmkctl.so sudah di-patch
* Menghapus bendera (flag) unlocker yang tidak digunakan (KPPW/KPST) agar cocok dengan versi Go
* Memperbarui semua tanggal hak cipta menjadi 2023
* Memastikan bahwa file yang akan di-patch memiliki izin tulis sebelum mem-patch, karena datastore NFS memberlakukan 
izin R/W dengan benar tetapi VMFS3 tidak, dan dapat mem-patch dengan bendera read only (hanya baca) yang disetel.

## 22/10/22 4.0.3
_drdonk:_
* Mengembalikan patch libvmkctl untuk memungkinkan vCenter mem-booting VM macOS di host ESXi
* Mengembalikan muatan (load) pada saat boot sehingga patch libvmkctl dimuat ketika hostd dimulai
* Menyimpan versi ESXi dengan file yang di-patch
* Perintah `check` membandingkan versi ESXi saat ini dan yang tersimpan

## 22/09/22 4.0.2
Terima kasih kepada _lucaskamp_ dan penguji _anonim_ atas pengujiannya.
 
_drdonk:_
* Kembali ke proses baca/tulis file biner di patchsmc alih-alih menggunakan mmap, yang menyebabkan kesalahan diam-diam selama proses pem-patch-an
yang berujung pada kegagalan patch.
* Memodifikasi format VMTAR dari PSIGNED-XZ menjadi GZIP
* Perintah sekarang diharuskan berjalan setiap boot ESXi untuk menghindari kemungkinan "Purple Screens of Death" (PSOD).
* Memastikan file vmx memiliki izin yang benar, 4555/-r-sr-xr-x, pada arsip apple.v00

## 26/01/22 4.0.1
_drdonk:_
* Memperbaiki bit +x yang hilang pada checksmc, dumpsmc dan relock

## 03/08/22 4.0.0
* Rilis perdana
