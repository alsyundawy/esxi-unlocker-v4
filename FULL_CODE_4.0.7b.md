# ESXi Unlocker V4 4.0.7b - Full Code

Generated from the reviewed 4.0.7b package.

## README.md

```text
# macOS Unlocker V4 for VMware ESXi

## IMPORTANT: Security and licensing notice

Use this project only on systems you own or administer and only where your VMware and Apple/macOS licensing permits it. This project patches ESXi binaries and can make a host unsupported by VMware/Broadcom support. Do not use it on production hosts.

VMware/Broadcom and the open-vm-tools project continue to publish VMware Tools updates. As of the 2026 update pass for this fork, VMware Tools/open-vm-tools 13.1.0 build 25218885 is the current referenced tools generation, and guest tools should be kept updated for security and compatibility.

## Unlocker 2007-2026

The original DrDonk Unlocker project is now complete and archived for VMware Workstation/Player. This repository is an ESXi-focused fork/port that keeps the ESXi shell/Python workflow maintained separately.

Important distinction:

- Upstream DrDonk/unlocker 4.x is Go-based and targets VMware Workstation/Player.
- This fork targets VMware ESXi and uses ESXi-specific bootbank/VMTAR handling.
- Upstream logic is used only as a reference where the patching concepts match.

## 1. Introduction

Unlocker 4 for ESXi is intended for compatible VMware ESXi 6.7, 7.x, and 8.x hosts, including ESXi 8.0 U3-era builds.

The Unlocker enables certain flags and data tables that are required to see the macOS guest OS type and modifies the virtual SMC controller implementation. These capabilities are normally exposed when VMware runs on supported Apple hardware.

The patch code carries out the following ESXi-specific modifications:

- Patches `vmx` and `vmx-debug` to allow macOS guests to boot.
- Patches `vmx-stats` only when it exists and is non-empty; on some ESXi 8.x builds it is absent or a zero-byte stub.
- Patches `libvmkctl.so` only when present, allowing vCenter/hostd-side `smcPresent` behaviour required by some macOS guest workflows.
- Builds and registers `apple.v00` as an ESXi bootbank module.

The Unlocker cannot add capabilities that are not already present in compiled VMware code. It cannot:

- Add support for macOS versions not supported by the underlying VMware build.
- Add paravirtualized Apple GPU support.
- Add CPU features missing from the physical host.
- Provide formal AMD CPU support.

## 2. Installing the patcher

### Important notes

Run the ESXi unlocker again after every ESXi upgrade or patch because ESXi binary files and bootbank modules can change.

Before running `unlock`:

- Put the ESXi host into Maintenance Mode where possible.
- Shut down or migrate all running VMs.
- Ensure at least 400 MB free space in the folder/datastore where the unlocker is extracted.
- Keep a rollback path: run `relock`, reboot, and re-run `unlock` after host updates if `check` reports a mismatch.

Version 4.0.7b now refuses to patch if running VMs are detected through `esxcli vm process list`. It also warns if the host does not appear to be in Maintenance Mode.

### Steps

1. Download the release archive from your ESXi Unlocker fork release page.
2. Extract the archive to a datastore folder on the ESXi host.
3. Open an ESXi shell or SSH session as root.
4. Navigate to the extracted folder.
5. Run one of the commands below.

```sh
./unlock     # apply ESXi unlock patches
./relock     # remove ESXi unlock bootbank module
./check      # verify current patch status
```

For verbose shell tracing:

```sh
./unlock -v
./relock -v
```

## 3. Files

| File | Purpose |
|---|---|
| `unlock` | Stages ESXi binaries, applies patches, builds `apple.v00`, and registers the bootbank module. |
| `relock` | Removes `apple.v00` from ESXi bootbank configuration. |
| `check` | Checks ESXi version match, `apple.v00` load status, vmx patch state, and optional `libvmkctl.so` status. |
| `patchsmc` | Python patch/check/dump tool for ESXi `vmx` binaries. |
| `patchvmkctl` | Python patch/check tool for optional `libvmkctl.so`. |
| `checksmc` | Wrapper around `patchsmc check`. |
| `checkvmkctl` | Wrapper around `patchvmkctl check`. |
| `dumpsmc` | Wrapper around `patchsmc dump`. |

## 4. Copyright

Copyright (c) 2007-2026 David Parsons  
Copyright (c) 2024-2026 Harry DS Alsyundawy - Alsyundawy IT Solution

Credit must be given to the original authors when redistributing or modifying this project.

## 5. Thanks

Thanks to Zenith432 for originally building the C++ Unlocker and Mac Son of Knife (MSoK) for all testing and support.

Thanks also to Sam B for finding the solution for ESXi 6 and helping with debugging expertise. Sam also wrote code for patching ESXi ELF files and modified Unlocker code to run on Python 3 in the ESXi 6.5 environment.

Thanks to lucaskamp and other testers who helped validate Unlocker version 4 behaviour.

## 6. Changelog

See [CHANGELOG.md](CHANGELOG.md) for the complete version history.

```

## README-ID.md

```text
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

```

## CHANGELOG.md

```text
# Changes

All dates are UK DD/MM/YY format.

## 13/06/26 4.0.7b

_alsyundawy (Harry DS Alsyundawy - Alsyundawy IT Solution):_

> Final hardening pass for ESXi 6.7 / 7.x / 8 U3 as of 13/06/26. The ESXi
> wrapper remains fork-specific; upstream DrDonk/unlocker 4.x is used only as
> a patch-logic reference for matching concepts, not as a drop-in ESXi release.

- `unlock`: Define ANSI color variables before first possible error path; previous `python3` detection could print uncolored/empty error formatting because `${RED}` and `${RST}` were declared later.
- `unlock`: Add preflight checks for required ESXi commands (`vmware`, `esxcli`, `vmtar`, `pigz`, `BootModuleConfig.sh`, etc.) so missing host tools fail early instead of producing a broken `apple.v00`.
- `unlock`: Make `vmware -v` failure fatal; writing `Unknown` to `unlock.conf` would make later `check` unreliable after host updates.
- `unlock`: Validate `df -m .` output is numeric before comparing free space, preventing POSIX shell integer-expression errors on unexpected `df` output.
- `unlock`: Refuse to patch while running VMs are detected via `esxcli vm process list`; warn when the host does not appear to be in Maintenance Mode.
- `unlock`: Replace non-portable `find -delete` staging cleanup with POSIX-compatible `find ... -exec rm -f {} \;`, improving BusyBox/ESXi compatibility.
- `unlock`: Treat failed `patchsmc` on required `vmx` and `vmx-debug` as fatal; optional `vmx-stats` and `libvmkctl.so` remain non-fatal where not applicable.
- `unlock`: Detect and remove an existing `/bootbank/apple.v00` before adding the new boot module, reducing duplicate/stale module risk.
- `check`: Add final non-zero exit status when required checks fail, while keeping optional `vmx-stats`/`libvmkctl.so` absence as warnings.
- `check`: Add robust script-directory fallback if `readlink -f` is unavailable and verify the detected `python3` path is executable.
- `checksmc`, `checkvmkctl`, `dumpsmc`: Add PATH hardening, robust `SCRIPT_DIR` fallback, executable `python3` validation, and safer `${1:-}` argument checks.
- `patchsmc`: Return meaningful success/failure exit codes to the caller; failed required binary patching now stops `unlock`.
- `patchsmc`: Move `.bak` creation until after already-patched and binary-layout checks, avoiding misleading backups for already-patched or incompatible files.
- `patchsmc`: Detect partial OSK patch state (`OSK0` present but `OSK1` missing, or vice versa) and fail instead of attempting a blind re-patch.
- `patchsmc`: Locate table-1 `#KEY` relative to `SMC_HEADER_V1` instead of using global `rfind()`, preventing accidental selection of an unrelated late marker.
- `patchsmc`, `patchvmkctl`: Restore file permissions with `stat.S_IMODE(os.stat(...).st_mode)` instead of passing full `st_mode` bits to `os.chmod()`.
- `patchvmkctl`: Return meaningful exit status and keep absent `applesmc` as non-fatal for already-patched or ESXi builds where the optional vmkctl patch does not apply.
- `relock`: Add `BootModuleConfig.sh` validation and stricter failure reporting when removal fails.
- `README.md`, `README-ID.md`: Update project status, compatibility wording, 2026 VMware Tools guidance, and ESXi-specific fork clarification.
- `TROUBLESHOOTING.md`, `TROUBLESHOOTING-ID.md`: Update examples to 4.0.7b and document the new non-zero check behaviour, running-VM guard, and optional component handling.
- `LICENSE`: Add explicit 2024-2026 modification copyright line while preserving the MIT license text.

## 10/06/26 4.0.7a

_alsyundawy (Harry DS Alsyundawy - Alsyundawy IT Solution):_

> Comprehensive code review pass against ESXi 6.7 / 7.x / 8 U3 — all fixes
> based on analysis of DrDonk/unlocker 4.2.8 Go source and ESXi-specific
> requirements.

- `unlock`: **Critical fix** — wildcard glob `./tmp/bin/*` in `chmod` and
  `tar` steps caused `.bak` and `.sha256` audit files (created by patchsmc/
  patchvmkctl in staging) to be packed into `apple.v00` bootbank VMTAR and
  have setuid bit applied. Fix: purge audit files with `find -delete` before
  chmod/tar; use explicit file targets for `chmod 4555`.
- `unlock`: Add guard for `/lib64/libvmkctl.so` absence (not present on all
  ESXi builds); skip vmkctl patch gracefully instead of failing.
- `unlock`: Add `python3` binary detection loop covering ESXi 6.7 (`venv-65`),
  ESXi 7.x (`venv-vsphere`), and ESXi 8.x (`/usr/bin/python3`). Script now
  exports `$PYTHON3` and uses it for all Python invocations.
- `patchsmc`: Fix unused loop variable `i` → `_` in `dumpkeys()`.
- `patchsmc`: Explicitly `os.chmod(name, orig_mode)` after patching vmx
  binaries — mirrors upstream Go `CopyFile os.Chmod()` call in `vmw.go`.
- `patchvmkctl`: Explicitly `os.chmod(name, orig_mode)` after writing patched
  content (open `'wb'` may not preserve permission bits on all ESXi FS drivers).
- `patchvmkctl`: Move `_backup_file()` call to after `APPLESMC` search so
  `.bak` is not created when file is already patched or incompatible.
- `checksmc`, `checkvmkctl`, `dumpsmc`: Add `SCRIPT_DIR` resolution via
  `readlink -f` so wrappers work from any working directory (not just when
  called from the unlocker directory). Add `$1` argument validation. Use
  detected `$PYTHON3` path.
- `check`: Add `SCRIPT_DIR` + `$PYTHON3` detection (same as wrappers above).
- `check`: Fix `grep -qil` on busybox ESXi grep — `-l` (list filenames) and
  `-q` (quiet) conflict; replaced with per-binary `grep -q` loop.
- `check`: Guard `vmware -v` failure with `|| ESXI=''` and warning message.
- `check`: Add `/lib64/libvmkctl.so` existence check before patchvmkctl call.
- `check`: Fix POSIX sh warnings (ShellCheck) related to `printf '---\n'` by
  safely explicitly formatting strings starting with dashes: `printf '%s\n' '---'`.

## 10/06/26 4.0.7

_alsyundawy (Harry DS Alsyundawy - Alsyundawy IT Solution):_

> Reference: DrDonk/unlocker **4.2.8** (14/05/26) — analysed Linux binaries
> and Go source to align ESXi Python/Shell implementation with upstream logic.

- `patchsmc`: Adopt raw OSK byte literals (`osk0Data`/`osk1Data`) from Go
  `smc.go` instead of rot13 encoding — more transparent and auditable.
- `patchsmc`: Add SHA-256 checksum before/after patch (mirrors upstream
  `WriteHashes()` feature introduced in Go 4.1.0).
- `patchsmc`: Add `.bak` backup of each file before patching (mirrors
  upstream `Backup()` in `vmw.go`).
- `patchsmc`: Improve already-patched check using raw OSK data bytes search
  (faster, no seek required) — mirrors upstream `checkPatch()` in `smc.go`.
- `patchsmc`: Improve ELF RELA loop guard (`find()` returns -1 → break).
- `patchvmkctl`: Add SHA-256 before/after patch (upstream `WriteHashes`).
- `patchvmkctl`: Add `.bak` backup before patching (upstream `Backup()`).
- `patchvmkctl`: Print `before -> after` string (mirrors upstream
  `PatchVMKCTL()` in Go `vmkctl.go`).
- `patchvmkctl`: Use `bytearray` for in-memory patch (safer than `r+b` seek).
- `unlock`: Add ESXi version detection and display at startup.
- `unlock`: Adopt upstream 4.2.4+ `vmx-stats` existence check in shell:
  only copy and patch if file is non-empty and a regular file (ESXi 8.x
  ships a 0-byte stub for `vmx-stats`).
- `check`: Add per-binary `checksmc` invocation (SHA-256 + patch status).
- `check`: Add `libvmkctl.so` patch status via `patchvmkctl check`.
- All scripts: Update version string to `4.0.7`.
- All scripts: Add `YLW` (yellow) ANSI color for non-fatal warnings.

## 10/06/26 4.0.6

_alsyundawy (Harry DS Alsyundawy - Alsyundawy IT Solution):_

- `unlock`: Fix **critical bug** — erroneous `$1` positional argument was appended
  to `cp`, `rm`, `vmtar`, `pigz`, and `mv` commands, causing all file staging
  and cleanup operations to fail silently or with wrong filenames.
- `unlock`: Replace `read -p` with POSIX-safe `printf … ; read -r` idiom.
- `unlock`: Use `( subshell )` for `tar` step to avoid needing `cd` back.
- `unlock`: Use `./*` glob in `tar` so filenames starting with `-` are safe.
- `unlock`: Add `cd … || exit` guard as required by POSIX sh best practices.
- `unlock`: Use `printf '%s'` pattern for variables in format strings.
- `relock`: Add missing `exit 0`. Replace `read -p` with POSIX-safe idiom.
- `relock`: Add informational message when user chooses not to reboot.
- `check`: Replace all bashism `[[ ]]` constructs with POSIX `[ ]`.
- `check`: Replace `==` with `=` inside `[ ]` for POSIX sh portability.
- `check`: Use direct command tests in `if` instead of checking `$?`.
- `check`: Use `printf '%s'` pattern for variables in format strings.
- `checksmc`, `checkvmkctl`, `dumpsmc`: Quote `$1` to prevent word splitting.
- `patchvmkctl`: Change shebang from `python` to `python3`.
- `patchvmkctl`: Replace bare `open()` calls with `with` context managers.
- `patchsmc`: Remove redundant `vmx.close()` calls inside/after `with` blocks.
- `patchsmc`, `patchvmkctl`: Add module docstrings with copyright and changelog.
- All scripts: Add docblock headers with usage, copyright, and changelog.
- All scripts: Update version string to `4.0.6`.
- `README.md`: Fix all markdownlint warnings (MD001, MD009, MD012, MD022,
  MD026, MD034). Fix typos: `developemnt`, `implmentation`, `capabiltiites`,
  `Maintanence`. Add missing section 3 (Copyright). Wrap bare URLs.

## 03/03/23 4.0.6

### This is the last release of ESXi Unlocker.

_drdonk:_
* Tidy up of code
* Add explanantion that the unlocker cannot add AMD support but some settings in VMX file may work

## 09/01/23 4.0.5
_drdonk:_
* Fixed permissions error correctly this time!

## 03/01/23 4.0.4
_drdonk:_
* `check` command correctly reports if system not patched
* `unlock` command checks for free disk space before patching
* `unlock`  command checks for previous V3 installation before patching
* Fixed error if libvmkctl.so already patched
* Removed an unused unlocker flag (KPPW/KPST) to match Go version
* Update all copyright dates to 2023
* Make sure the files to be patched have write permissions before patching as NFS datastores correctly enforce 
R/W permissions but VMFS3 does not and can patch with read only flag set.

## 22/10/22 4.0.3
_drdonk:_
* Reinstate libvmkctl patch to allow vCenter to boot macOS VMs on ESXi host
* Reinstate boot time load so the libvmkctl patch is loaded when hostd starts
* Store ESXi version with patched files
* `check` command compares current and stored ESXi versions

## 22/09/22 4.0.2
Thanks to _lucaskamp_ and an _anonymous_ tester for testing.
 
_drdonk:_
* Revert to binary file read/writes in patchsmc instead of using mmap, which caused silent errors during the patching 
process leading to failed patches.
* Modified VMTAR format from PSIGNED-XZ to GZIP
* Commands are now required to be run each boot of ESXi to avoid possible "Purple Screens of Death" (PSOD).
* Ensure vmx files have correct permissions, 4555/-r-sr-xr-x,  in apple.v00 archive

## 26/01/22 4.0.1
_drdonk:_
* Fix missing +x bit on checksmc, dumpsmc and relock

## 03/08/22 4.0.0
* Initial release

```

## CHANGELOG-ID.md

```text
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

```

## TROUBLESHOOTING.md

```text
# Troubleshooting Notes

## Patch checker

Re-run `unlock` after every ESXi upgrade or patch cycle. ESXi updates can replace `vmx`, `vmx-debug`, `libvmkctl.so`, bootbank metadata, or the loaded tardisk set.

Check patch status from the unlocker folder:

```sh
./check
```

In 4.0.7b, `check` exits with status `0` when required checks pass and status `1` when a required check fails. Optional components such as `vmx-stats` and `libvmkctl.so` are warnings when absent because they are not present on all ESXi builds.

A healthy patched host should look conceptually like this; exact ESXi build, SHA-256 values, and optional component output will differ:

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

If the patch was built for a different ESXi version, `check` reports a version mismatch and exits non-zero:

```text
>>> Version mismatch — please relock and unlock to update patches <<<
Check completed: one or more required checks failed.
```

Recommended recovery flow:

```sh
./relock
reboot
# after reboot
./unlock
reboot
./check
```

## Running VMs / Maintenance Mode

Version 4.0.7b refuses to run `unlock` when `esxcli vm process list` detects running VMs. Shut down or migrate all VMs first.

The script also warns if the host does not appear to be in Maintenance Mode. Maintenance Mode is strongly recommended because `unlock` stages and repacks ESXi bootbank modules.

## apple.v00 not loaded

If `check` says `apple.v00 not loaded`, the most common causes are:

1. The host was patched but not rebooted.
2. `BootModuleConfig.sh --add=apple.v00` failed.
3. ESXi was upgraded and the bootbank state changed.

Run:

```sh
esxcli system visorfs tardisk list | grep apple.v00
ls -l /bootbank/apple.v00 /altbootbank/apple.v00 2>/dev/null
```

If the module is missing or stale, run the relock/reboot/unlock/reboot flow.

## vmx or vmx-debug not patched

`vmx` and `vmx-debug` are required. If either fails, do not treat the host as correctly patched.

Useful checks:

```sh
./checksmc /bin/vmx
./checksmc /bin/vmx-debug
grep -a 'AppleComputerInc' /bin/vmx /bin/vmx-debug
```

If `patchsmc` reports a partial OSK patch state, do not continue patching blindly. Relock, reboot, refresh the ESXi binaries through a known-good host patch level, and run `unlock` again.

## vmx-stats missing or not patched

On some ESXi 8.x builds, `/bin/vmx-stats` is absent or a zero-byte stub. Version 4.0.7b treats this as optional and non-fatal.

## libvmkctl.so missing or optional

`/lib64/libvmkctl.so` is not guaranteed to exist on every ESXi build. If it is missing, the vmkctl patch is skipped. If you use vCenter and macOS guests fail to start from vCenter after host patching, try:

1. Disconnect and reconnect the ESXi host in vCenter.
2. Remove and re-add the ESXi host in vCenter.
3. Re-run `./checkvmkctl /lib64/libvmkctl.so` if the file exists.

## Set a specific macOS guest resolution

Installing VMware Tools should allow different video modes to be selected. If resolution still does not change, open Terminal inside the macOS guest and run:

```sh
sudo /Library/Application\ Support/VMware\ Tools/vmware-resolutionSet <width> <height>
```

Example:

```sh
sudo /Library/Application\ Support/VMware\ Tools/vmware-resolutionSet 1440 900
```

## AMD CPUs

The ESXi Unlocker does not add formal AMD CPU support. Any AMD workaround must be handled by guest configuration, OpenCore, or other macOS-specific tooling outside this project.

If you edit a VMX file manually, make sure there are no duplicate keys. Duplicate VMX keys can prevent the VM from starting and can produce dictionary/configuration errors.

```

## TROUBLESHOOTING-ID.md

```text
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

```

## LICENSE

```text
MIT License

Copyright (c) 2011-2023 David Parsons
Copyright (c) 2024-2026 Harry DS Alsyundawy - Alsyundawy IT Solution

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

```

## check

```sh
#!/bin/sh
# SPDX-License-Identifier: MIT
#
# check - Verify the patch status of the VMware ESXi installation
#
# Description : Compares the ESXi version at patch time with the current
#               running version, verifies that apple.v00 is loaded in the
#               visorfs tardisk list, and confirms vmx binaries and the
#               smcPresent flag have been patched.
#
# Usage       : check
#
# Author      : drdonk / David Parsons
# Maintainer  : Harry DS Alsyundawy <alsyundawy@gmail.com>
# Copyright   : Copyright (c) 2007-2026 David Parsons
#               Copyright (c) 2024-2026 Harry DS Alsyundawy - Alsyundawy IT Solution
# License     : MIT
#
# Changelog   :
#   2026-06-13 4.0.7b alsyundawy  Add reusable helpers, robust SCRIPT_DIR
#                                 fallback, mandatory python3 executable check,
#                                 clearer exit status for automation, missing
#                                 required-binary guards, and final OK/FAILED
#                                 summary without treating optional vmx-stats or
#                                 libvmkctl absence as fatal.
#   2026-06-10 4.0.7a alsyundawy  Add SCRIPT_DIR so python3 calls work from any
#                                 CWD. Add python3 detection for ESXi 6.7 through
#                                 8 U3. Fix busybox grep -q/-l conflict. Guard
#                                 'vmware -v' failure.
#   2026-06-10 4.0.7  alsyundawy  Update banner. SHA-256 via checksmc.
#                                 libvmkctl.so check via patchvmkctl.
#   2026-06-10 4.0.6  alsyundawy  Replace [[ ]] with POSIX [ ]. Replace == with
#                                 =. Use direct command in if. Quote variables.
#   2023-01-03 4.0.4  drdonk     check correctly reports if system not patched.
#   2022-10-22 4.0.3  drdonk     Store ESXi version; compare with stored value.
#   2022-08-03 4.0.0  drdonk     Initial release.

export PATH=/bin:/sbin:/usr/bin:/usr/sbin

RED='\033[41m\033[37m'
GRN='\033[32m'
YLW='\033[33m'
RST='\033[0m'
CHECK_FAILED=0

fail_check() {
    CHECK_FAILED=1
    printf '%s\n' "${RED}>>> $* <<<${RST}"
}

warn() {
    printf '%s\n' "${YLW}WARNING: $*${RST}"
}

resolve_script() {
    if command -v readlink >/dev/null 2>&1; then
        _resolved=$(readlink -f "$0" 2>/dev/null || true)
        if [ -n "$_resolved" ]; then
            printf '%s\n' "$_resolved"
            return 0
        fi
    fi

    case "$0" in
        /*) printf '%s\n' "$0" ;;
        *)  printf '%s/%s\n' "$(pwd -P)" "$0" ;;
    esac
}

detect_python3() {
    for _py in /usr/bin/python3 \
               /usr/lib/vmware/venv-vsphere/bin/python3 \
               /usr/lib/vmware/venv-65/bin/python3; do
        if [ -x "$_py" ]; then
            printf '%s\n' "$_py"
            return 0
        fi
    done
    return 1
}

SCRIPT=$(resolve_script)
SCRIPT_DIR=$(dirname "$SCRIPT")
PYTHON3=$(detect_python3) || {
    fail_check 'python3 not found on this ESXi host'
    PYTHON3=''
}

printf 'VMware ESXi Unlocker 4.0.7b\n'
printf '===========================\n'
printf '\nChecking unlocker status...\n'

ESXI=$(vmware -v 2>/dev/null) || ESXI=''
if [ -z "$ESXI" ]; then
    fail_check "Could not determine ESXi version via 'vmware -v'"
else
    printf 'Current ESXi version  : %s\n' "$ESXI"
fi

if [ ! -f '/etc/unlock.conf' ]; then
    fail_check 'System has not been patched (unlock.conf missing)'
else
    PATCH=$(cat /etc/unlock.conf)
    printf 'Patch built for ESXi  : %s\n' "$PATCH"

    if [ -n "$ESXI" ] && [ "$ESXI" != "$PATCH" ]; then
        fail_check 'Version mismatch — please relock and unlock to update patches'
    elif [ -n "$ESXI" ]; then
        printf '%s\n' "${GRN}ESXi version matches patch record.${RST}"
    fi
fi

printf '\nChecking VMTAR loaded...\n'
if esxcli system visorfs tardisk list 2>/dev/null | grep -q 'apple.v00'; then
    printf '%s\n' "${GRN}apple.v00 loaded.${RST}"
else
    warn 'apple.v00 not loaded — a reboot may be required, or bootbank module is not registered'
fi

printf '\nChecking vmx vSMC patch status...\n'
for _f in /bin/vmx /bin/vmx-debug; do
    if [ ! -f "$_f" ] || [ ! -s "$_f" ]; then
        fail_check "Required binary missing or empty: $_f"
        continue
    fi

    if grep -q 'AppleComputerInc' "$_f" 2>/dev/null; then
        printf '%s\n' "${GRN}$_f contains AppleComputerInc marker.${RST}"
    else
        fail_check "$_f is NOT patched"
    fi

    if [ -n "$PYTHON3" ]; then
        printf '%s\n' '---'
        "$PYTHON3" "$SCRIPT_DIR/patchsmc" check "$_f" 2>/dev/null || CHECK_FAILED=1
    fi
done

if [ -f '/bin/vmx-stats' ] && [ -s '/bin/vmx-stats' ]; then
    if grep -q 'AppleComputerInc' /bin/vmx-stats 2>/dev/null; then
        printf '%s\n' "${GRN}/bin/vmx-stats contains AppleComputerInc marker.${RST}"
    else
        warn '/bin/vmx-stats is NOT patched; this binary is optional on many ESXi builds'
    fi

    if [ -n "$PYTHON3" ]; then
        printf '%s\n' '---'
        "$PYTHON3" "$SCRIPT_DIR/patchsmc" check /bin/vmx-stats 2>/dev/null || \
            warn 'patchsmc check failed for optional vmx-stats'
    fi
else
    printf '%s\n' "${YLW}SKIP: /bin/vmx-stats missing or empty — optional on ESXi 8.x.${RST}"
fi

printf '\nChecking libvmkctl.so patch status...\n'
if [ -f '/lib64/libvmkctl.so' ] && [ -s '/lib64/libvmkctl.so' ]; then
    if [ -n "$PYTHON3" ]; then
        "$PYTHON3" "$SCRIPT_DIR/patchvmkctl" check /lib64/libvmkctl.so 2>/dev/null || \
            warn 'patchvmkctl check returned non-zero; vCenter smcPresent behaviour may need manual review'
    fi
else
    printf '%s\n' "${YLW}SKIP: /lib64/libvmkctl.so missing — optional vmkctl patch not applicable.${RST}"
fi

printf '\nChecking smcPresent status...\n'
if command -v vim-cmd >/dev/null 2>&1; then
    _smc=$(vim-cmd hostsvc/hosthardware 2>/dev/null | grep smcPresent | cut -d ',' -f 1 | sed 's/^[[:space:]]*//')
    if [ -n "$_smc" ]; then
        printf '%s\n' "$_smc"
    else
        warn 'Could not retrieve smcPresent from vim-cmd'
    fi
else
    warn 'vim-cmd not found; cannot query smcPresent'
fi

if [ "$CHECK_FAILED" -eq 0 ]; then
    printf '\n%s\n' "${GRN}Check completed: required patch status looks OK.${RST}"
    exit 0
fi

printf '\n%s\n' "${RED}Check completed: one or more required checks failed.${RST}"
exit 1

```

## checksmc

```sh
#!/bin/sh
# SPDX-License-Identifier: MIT
# checksmc - Check the vSMC patch status of a vmx binary
# Usage: checksmc <vmx-filename>
# Copyright (c) 2007-2026 David Parsons
# Copyright (c) 2024-2026 Harry DS Alsyundawy - Alsyundawy IT Solution
# License: MIT
# Changelog:
#   2026-06-13 4.0.7b alsyundawy  Add PATH hardening, robust SCRIPT_DIR
#                                 fallback when readlink -f is unavailable,
#                                 executable python3 validation, and safer
#                                 argument handling with ${1:-}.
#   2026-06-10 4.0.7a alsyundawy  Add SCRIPT_DIR resolution so script works
#                                 regardless of caller's CWD. Validate $1.
#   2026-06-10 4.0.7  alsyundawy  Update version reference.
#   2026-06-10 4.0.6  alsyundawy  Quote $1 to prevent word splitting.

export PATH=/bin:/sbin:/usr/bin:/usr/sbin

resolve_script() {
    if command -v readlink >/dev/null 2>&1; then
        _resolved=$(readlink -f "$0" 2>/dev/null || true)
        if [ -n "$_resolved" ]; then
            printf '%s\n' "$_resolved"
            return 0
        fi
    fi

    case "$0" in
        /*) printf '%s\n' "$0" ;;
        *)  printf '%s/%s\n' "$(pwd -P)" "$0" ;;
    esac
}

detect_python3() {
    for _py in /usr/bin/python3 \
               /usr/lib/vmware/venv-vsphere/bin/python3 \
               /usr/lib/vmware/venv-65/bin/python3; do
        if [ -x "$_py" ]; then
            printf '%s\n' "$_py"
            return 0
        fi
    done
    return 1
}

if [ -z "${1:-}" ]; then
    printf 'Usage: %s <vmx-filename>\n' "$0" >&2
    exit 1
fi

SCRIPT=$(resolve_script)
SCRIPT_DIR=$(dirname "$SCRIPT")
PYTHON3=$(detect_python3) || {
    printf 'ERROR: python3 not found on this ESXi host\n' >&2
    exit 1
}

"$PYTHON3" "$SCRIPT_DIR/patchsmc" check "$1"

```

## checkvmkctl

```sh
#!/bin/sh
# SPDX-License-Identifier: MIT
# checkvmkctl - Check the patch status of libvmkctl.so
# Usage: checkvmkctl <libvmkctl-filename>
# Copyright (c) 2007-2026 David Parsons
# Copyright (c) 2024-2026 Harry DS Alsyundawy - Alsyundawy IT Solution
# License: MIT
# Changelog:
#   2026-06-13 4.0.7b alsyundawy  Add PATH hardening, robust SCRIPT_DIR
#                                 fallback when readlink -f is unavailable,
#                                 executable python3 validation, and safer
#                                 argument handling with ${1:-}.
#   2026-06-10 4.0.7a alsyundawy  Add SCRIPT_DIR resolution so script works
#                                 regardless of caller's CWD. Validate $1.
#   2026-06-10 4.0.7  alsyundawy  Update version reference.
#   2026-06-10 4.0.6  alsyundawy  Quote $1 to prevent word splitting.

export PATH=/bin:/sbin:/usr/bin:/usr/sbin

resolve_script() {
    if command -v readlink >/dev/null 2>&1; then
        _resolved=$(readlink -f "$0" 2>/dev/null || true)
        if [ -n "$_resolved" ]; then
            printf '%s\n' "$_resolved"
            return 0
        fi
    fi

    case "$0" in
        /*) printf '%s\n' "$0" ;;
        *)  printf '%s/%s\n' "$(pwd -P)" "$0" ;;
    esac
}

detect_python3() {
    for _py in /usr/bin/python3 \
               /usr/lib/vmware/venv-vsphere/bin/python3 \
               /usr/lib/vmware/venv-65/bin/python3; do
        if [ -x "$_py" ]; then
            printf '%s\n' "$_py"
            return 0
        fi
    done
    return 1
}

if [ -z "${1:-}" ]; then
    printf 'Usage: %s <libvmkctl-filename>\n' "$0" >&2
    exit 1
fi

SCRIPT=$(resolve_script)
SCRIPT_DIR=$(dirname "$SCRIPT")
PYTHON3=$(detect_python3) || {
    printf 'ERROR: python3 not found on this ESXi host\n' >&2
    exit 1
}

"$PYTHON3" "$SCRIPT_DIR/patchvmkctl" check "$1"

```

## dumpsmc

```sh
#!/bin/sh
# SPDX-License-Identifier: MIT
# dumpsmc - Dump the vSMC key table from a vmx binary
# Usage: dumpsmc <vmx-filename>
# Copyright (c) 2007-2026 David Parsons
# Copyright (c) 2024-2026 Harry DS Alsyundawy - Alsyundawy IT Solution
# License: MIT
# Changelog:
#   2026-06-13 4.0.7b alsyundawy  Add PATH hardening, robust SCRIPT_DIR
#                                 fallback when readlink -f is unavailable,
#                                 executable python3 validation, and safer
#                                 argument handling with ${1:-}.
#   2026-06-10 4.0.7a alsyundawy  Add SCRIPT_DIR resolution so script works
#                                 regardless of caller's CWD. Validate $1.
#   2026-06-10 4.0.7  alsyundawy  Update version reference.
#   2026-06-10 4.0.6  alsyundawy  Quote $1 to prevent word splitting.

export PATH=/bin:/sbin:/usr/bin:/usr/sbin

resolve_script() {
    if command -v readlink >/dev/null 2>&1; then
        _resolved=$(readlink -f "$0" 2>/dev/null || true)
        if [ -n "$_resolved" ]; then
            printf '%s\n' "$_resolved"
            return 0
        fi
    fi

    case "$0" in
        /*) printf '%s\n' "$0" ;;
        *)  printf '%s/%s\n' "$(pwd -P)" "$0" ;;
    esac
}

detect_python3() {
    for _py in /usr/bin/python3 \
               /usr/lib/vmware/venv-vsphere/bin/python3 \
               /usr/lib/vmware/venv-65/bin/python3; do
        if [ -x "$_py" ]; then
            printf '%s\n' "$_py"
            return 0
        fi
    done
    return 1
}

if [ -z "${1:-}" ]; then
    printf 'Usage: %s <vmx-filename>\n' "$0" >&2
    exit 1
fi

SCRIPT=$(resolve_script)
SCRIPT_DIR=$(dirname "$SCRIPT")
PYTHON3=$(detect_python3) || {
    printf 'ERROR: python3 not found on this ESXi host\n' >&2
    exit 1
}

"$PYTHON3" "$SCRIPT_DIR/patchsmc" dump "$1"

```

## patchsmc

```python
#!/usr/bin/env python3
# coding=utf-8

# SPDX-License-Identifier: MIT

"""
patchsmc - Patch, check, or dump the vSMC key tables in VMware ESXi vmx binaries.

The vSMC (virtual System Management Controller) tables embedded in the vmx
executables contain OSK0 and OSK1 keys that macOS uses to verify it is running
on Apple hardware. This script replaces the handler pointers and key data so
that a patched ESXi host passes the Apple SMC validation.

Usage:
    patchsmc patch|check|dump <vmx-filename>

Compatibility:
    ESXi 6.7, 7.0, 7.0 U1/U2/U3, 8.0, 8.0 U1/U2/U3 (ESXi 8 U3 latest)

Copyright (c) 2007-2026 David Parsons
Copyright (c) 2024-2026 Harry DS Alsyundawy - Alsyundawy IT Solution
License: MIT

Changelog:
    2026-06-13 4.0.7b alsyundawy  Return non-zero status for failed patch/check
                                  paths so unlock/check automation can stop
                                  safely. Move .bak creation until after
                                  already-patched and binary-layout checks so
                                  unsupported or already-patched files do not
                                  generate misleading backups. Use S_IMODE()
                                  when restoring permissions. Locate table 1
                                  #KEY relative to SMC_HEADER_V1 instead of
                                  using global rfind().
    2026-06-10 4.0.7a alsyundawy  Fix unused loop variable 'i' in dumpkeys()
                                  (use '_' per Python convention). Explicitly
                                  preserve and restore original file permissions
                                  after patching vmx binaries (mirrors upstream
                                  Go CopyFile os.Chmod() call in vmw.go).
    2026-06-10 4.0.7  alsyundawy  Reference: DrDonk/unlocker 4.2.8 (14/05/26).
                                  Raw OSK bytes, SHA-256, backup, fast patched
                                  check, ELF RELA guard, ESXi 8.x guards.
    2026-06-10 4.0.6  alsyundawy  Fix critical bugs: erroneous $1 arg in shell
                                  scripts. POSIX sh compatibility. ESXi 8.x
                                  guards: skip 0-byte/symlink files, guard
                                  bytes.find()==-1 before seek. _find_required()
                                  helper. Remove redundant file.close() calls.
    2023-01-09 4.0.5  drdonk     Fixed permissions error correctly this time!
    2023-01-03 4.0.4  drdonk     Remove unused unlocker flag (KPPW/KPST).
    2022-10-22 4.0.3  drdonk     Reinstate libvmkctl patch and boot time load.
    2022-09-22 4.0.2  drdonk     Revert to binary file read/writes.
    2022-01-26 4.0.1  drdonk     Fix missing +x bit on checksmc, dumpsmc, relock.
    2022-08-03 4.0.0  drdonk     Initial release.

vSMC Header Structure
=====================
Offset  Length  Struct Type Description
----------------------------------------
0x00/00 0x08/08 Q      ptr  Offset to key table
0x08/08 0x04/4  I      int  Number of private keys
0x0C/12 0x04/4  I      int  Number of public keys

vSMC Key Data Structure
Offset  Length  Struct Type Description
----------------------------------------
0x00/00 0x04/04 4s     int  Key name (byte reversed e.g. #KEY is YEK#)
0x04/04 0x01/01 B      byte Length of returned data
0x05/05 0x04/04 4s     int  Data type (byte reversed e.g. ui32 is 23iu)
0x09/09 0x01/01 B      byte Flag R/W
0x0A/10 0x06/06 6x     byte Padding
0x10/16 0x08/08 Q      ptr  Internal VMware routine
0x18/24 0x30/48 48B    byte Data

The internal VMware routines point to 4 variants:
    AppleSMCHandleDefault
    AppleSMCHandleNTOK
    AppleSMCHandleNumKeys
    AppleSMCHandleOSK
"""

import hashlib
import os
import shutil
import stat
import struct
import sys

# ---------------------------------------------------------------------------
# Version check
# ---------------------------------------------------------------------------
if sys.version_info < (3, 6):
    sys.stderr.write('You need Python 3.6 or later\n')
    sys.exit(1)

# ---------------------------------------------------------------------------
# Struct constants for header and key access
# ---------------------------------------------------------------------------
HDR_PACK   = '=QII'
HDR_LENGTH = 16
KEY_PACK   = '=4sB4sB6xQ'
KEY_LENGTH = 24
DATA_LENGTH = 48
ROW_LENGTH  = KEY_LENGTH + DATA_LENGTH

# ---------------------------------------------------------------------------
# vSMC header magic bytes (private/public key count fields)
# From Go upstream: smcHeaderV0 / smcHeaderV1
# ---------------------------------------------------------------------------
SMC_HEADER_V0 = b'\xF2\x00\x00\x00\xF0\x00\x00\x00'
SMC_HEADER_V1 = b'\xB4\x01\x00\x00\xB0\x01\x00\x00'

# ---------------------------------------------------------------------------
# Key name constants (stored byte-reversed in binary)
# ---------------------------------------------------------------------------
KEY_KEY  = b'YEK#'   # #KEY reversed
LKS_KEY  = b'SKL+'   # +LKS reversed
OSK0_KEY = b'0KSO'   # OSK0 reversed
OSK1_KEY = b'1KSO'   # OSK1 reversed

# ---------------------------------------------------------------------------
# OSK data — raw bytes, matching osk0Data / osk1Data in DrDonk Go smc.go
# Decoded from rot13 for auditability but semantically identical to 4.0.5
# "ourhardworkbythesewordsguardedpl" (32 bytes)
# "easedontsteal(c)AppleComputerInc" (32 bytes)
# ---------------------------------------------------------------------------
OSK0 = b'ourhardworkbythesewordsguardedpl'
OSK1 = b'easedontsteal(c)AppleComputerInc'

# ---------------------------------------------------------------------------
# ELF magic for detecting ELF executables (ESXi vmx is ELF)
# ---------------------------------------------------------------------------
ELF_MAGIC = b'\x7fELF'


# ---------------------------------------------------------------------------
# Helper utilities
# ---------------------------------------------------------------------------

def _sha256(data: bytes) -> str:
    """Return hex SHA-256 digest of *data* (matches upstream WriteHashes)."""
    return hashlib.sha256(data).hexdigest()


def _find_required(data: bytes, pattern: bytes, start: int = 0,
                   label: str = '') -> int:
    """
    Locate *pattern* in *data* starting at *start*.

    Raises RuntimeError with a descriptive message instead of returning -1,
    preventing cryptic 'ValueError: seek out of range' on ESXi 8.x where
    some binary layouts may have changed.
    """
    pos = data.find(pattern, start)
    if pos == -1:
        desc = label or pattern.hex()
        raise RuntimeError(
            f'Pattern not found: {desc!r} — '
            f'binary may not be a patchable vmx build '
            f'(ESXi 8.x layout change or already different format).'
        )
    return pos


def _check_file_patchable(name: str) -> bool:
    """
    Return True if *name* is a non-empty regular file that can be patched.

    On ESXi 8.0+ vmx-stats is sometimes a 0-byte stub or a broken symlink.
    Upstream Go (DrDonk/unlocker 4.2.4+, vmw.go) checks os.Stat() before
    queuing vmx-stats for patching; we mirror that logic here.
    """
    # Resolve symlinks — os.path.isfile() follows symlinks
    if not os.path.isfile(name):
        print(f'SKIP: {name} — not a regular file (symlink, missing, or '
              f'directory). Skipping (expected on ESXi 8.x for vmx-stats).')
        return False
    size = os.path.getsize(name)
    if size == 0:
        print(f'SKIP: {name} — file is 0 bytes. '
              f'Skipping (ESXi 8.x stub file).')
        return False
    return True


def _backup_file(name: str) -> str:
    """
    Create a backup copy of *name* as *name*.bak before patching.

    Mirrors upstream Go Backup() in vmw.go. If a backup already exists it is
    left unchanged (idempotent), matching upstream behaviour.

    Returns the backup filename.
    """
    backup = name + '.bak'
    if not os.path.exists(backup):
        shutil.copy2(name, backup)
        print(f'Backup: {name} -> {backup}')
    else:
        print(f'Backup already exists: {backup} (skipped overwrite)')
    return backup


# ---------------------------------------------------------------------------
# Low-level binary I/O helpers
# ---------------------------------------------------------------------------

def bytetohex(data: bytes) -> str:
    return ''.join('{:02X}'.format(c) for c in data)


def printhdr(offset: int, hdr: tuple) -> None:
    print(f'File Offset  : 0x{offset:08x}')
    print(f'Keys Offset  : 0x{hdr[0]:08x}')
    print(f'Private Key #: 0x{hdr[1]:04x}/{hdr[1]:04d}')
    print(f'Public Keys #: 0x{hdr[2]:04x}/{hdr[2]:04d}')
    print()


def printkey(offset: int, smc_key: tuple, smc_data: bytes) -> None:
    smc_type = smc_key[2][::-1].replace(b'\x00', b' ').decode('UTF-8')
    print(f'0x{offset:08x} '
          f'{smc_key[0][::-1].decode("UTF-8")} '
          f'{smc_key[1]:03d} '
          f'{smc_type} '
          f'0x{smc_key[3]:02x} '
          f'0x{smc_key[4]:08x} '
          f'{bytetohex(smc_data)}')


def gethdr(vmx, offset: int) -> tuple:
    vmx.seek(offset)
    hdr = struct.unpack(HDR_PACK, vmx.read(HDR_LENGTH))
    vmx.seek(offset)
    return hdr


def getkey(vmx, offset: int) -> tuple:
    vmx.seek(offset)
    smc_key = struct.unpack(KEY_PACK, vmx.read(KEY_LENGTH))
    vmx.seek(offset)
    return smc_key


def setkey(vmx, offset: int, smc_key: tuple) -> None:
    vmx.seek(offset)
    vmx.write(struct.pack(KEY_PACK,
                          smc_key[0], smc_key[1],
                          smc_key[2], smc_key[3],
                          smc_key[4]))
    vmx.flush()
    vmx.seek(offset)


def getdata(vmx, offset: int, smc_key: tuple) -> bytes:
    vmx.seek(offset + KEY_LENGTH)
    smc_data = vmx.read(smc_key[1])
    vmx.seek(offset)
    return smc_data


def setdata(vmx, offset: int, smc_data: bytes) -> None:
    vmx.seek(offset + KEY_LENGTH)
    vmx.write(smc_data)
    vmx.flush()
    vmx.seek(offset)


# ---------------------------------------------------------------------------
# Patch helpers
# ---------------------------------------------------------------------------

def patchosk(vmx, offset: int, ptr: int, data: bytes) -> int:
    """
    Patch a single OSK key entry:
      1. Replace the function pointer (AppleSMCHandleOSK -> AppleSMCHandleDefault)
      2. Write the plain-text OSK data bytes

    Returns the original OSK handler pointer (needed for ELF RELA patching).
    """
    smc_key  = getkey(vmx, offset)
    smc_data = getdata(vmx, offset, smc_key)
    key      = smc_key[0][::-1].decode('UTF-8')
    smc_osk_ptr = smc_key[4]

    print(f'{key} Key Before:')
    printkey(offset, smc_key, smc_data)

    # Replace AppleSMCHandleOSK with AppleSMCHandleDefault
    temp    = list(smc_key)
    temp[4] = ptr
    smc_key = tuple(temp)
    setkey(vmx, offset, smc_key)

    # Write the OSK data value (32 bytes)
    setdata(vmx, offset, data)

    # Verify
    smc_key  = getkey(vmx, offset)
    smc_data = getdata(vmx, offset, smc_key)
    print(f'{key} Key After:')
    printkey(offset, smc_key, smc_data)

    return smc_osk_ptr


# ---------------------------------------------------------------------------
# Main public functions
# ---------------------------------------------------------------------------

def patchsmc(name: str) -> bool:
    """Patch the vSMC OSK0/OSK1 keys and ELF RELA records in *name*."""

    # ESXi 8.x guard: skip 0-byte stubs and non-regular files
    if not _check_file_patchable(name):
        return False

    # Capture original permissions before any write operation
    # Mirrors upstream Go CopyFile os.Chmod() in vmw.go
    orig_mode = stat.S_IMODE(os.stat(name).st_mode)

    with open(name, 'rb+') as vmx:
        vmx_bytes = vmx.read()

        # Compute SHA-256 of unpatched file (upstream WriteHashes feature)
        sha_before = _sha256(vmx_bytes)
        print(f'SHA256 (before): {sha_before}')

        # ------------------------------------------------------------------
        # Fast already-patched check using raw OSK data search
        # Mirrors upstream Go checkPatch() which searches for osk0Data/osk1Data
        # ------------------------------------------------------------------
        osk0_present = OSK0 in vmx_bytes
        osk1_present = OSK1 in vmx_bytes
        if osk0_present and osk1_present:
            print(f'File {name} is already patched')
            return True
        if osk0_present != osk1_present:
            print(f'ERROR: File {name} appears partially patched (OSK0/OSK1 mismatch).')
            return False

        # ------------------------------------------------------------------
        # Locate vSMC structures (guard against missing patterns on ESXi 8.x)
        # ------------------------------------------------------------------
        try:
            smc0_header = _find_required(
                vmx_bytes, SMC_HEADER_V0, label='SMC_HEADER_V0') - 8
            smc1_header = _find_required(
                vmx_bytes, SMC_HEADER_V1, label='SMC_HEADER_V1') - 8

            smc0_key = _find_required(
                vmx_bytes, KEY_KEY, smc0_header, label='#KEY (table 0)')
            smc1_key = _find_required(
                vmx_bytes, KEY_KEY, smc1_header, label='#KEY (table 1)')

            smc0_lks  = _find_required(
                vmx_bytes, LKS_KEY, smc0_key, label='+LKS (table 0)')
            smc1_lks  = _find_required(
                vmx_bytes, LKS_KEY, smc1_key, label='+LKS (table 1)')

            smc0_osk0 = _find_required(
                vmx_bytes, OSK0_KEY, smc0_key, label='OSK0 (table 0)')
            smc1_osk0 = _find_required(
                vmx_bytes, OSK0_KEY, smc1_key, label='OSK0 (table 1)')
            smc0_osk1 = _find_required(
                vmx_bytes, OSK1_KEY, smc0_key, label='OSK1 (table 0)')
            smc1_osk1 = _find_required(
                vmx_bytes, OSK1_KEY, smc1_key, label='OSK1 (table 1)')

        except RuntimeError as exc:
            print(f'ERROR: {exc}')
            print(f'File {name} cannot be patched — skipping.')
            return False

        # Backup only after confirming the file is patchable and not already patched.
        _backup_file(name)

        # ------------------------------------------------------------------
        # Patch first vSMC table (smc.version = "0")
        # ------------------------------------------------------------------
        print('\nappleSMCTableV0 (smc.version = "0")')
        hdr = gethdr(vmx, smc0_header)
        printhdr(smc0_header, hdr)

        smc_key         = getkey(vmx, smc0_lks)
        smc_default_ptr = smc_key[4]

        patchosk(vmx, smc0_osk0, smc_default_ptr, OSK0)
        patchosk(vmx, smc0_osk1, smc_default_ptr, OSK1)

        # ------------------------------------------------------------------
        # Patch second vSMC table (smc.version = "1")
        # ------------------------------------------------------------------
        print('\nappleSMCTableV1 (smc.version = "1")')
        hdr = gethdr(vmx, smc1_header)
        printhdr(smc1_header, hdr)

        smc_key         = getkey(vmx, smc1_lks)
        smc_default_ptr = smc_key[4]

        patchosk(vmx, smc1_osk0, smc_default_ptr, OSK0)
        smc_osk_ptr = patchosk(vmx, smc1_osk1, smc_default_ptr, OSK1)

        # ------------------------------------------------------------------
        # Patch ELF RELA records (ESXi vmx is an ELF binary)
        # Mirrors upstream patchELF() in Go smc.go
        # ------------------------------------------------------------------
        vmx.seek(0)
        magic = vmx.read(4)
        if magic == ELF_MAGIC:
            print(f'\nModifying ELF RELA records from '
                  f'0x{smc_osk_ptr:08x} -> 0x{smc_default_ptr:08x}')

            packed_old_ptr = struct.pack('=Q', smc_osk_ptr)
            packed_new_ptr = struct.pack('=Q', smc_default_ptr)

            offset = 0
            for _ in range(4):
                offset = vmx_bytes.find(packed_old_ptr, offset)
                if offset == -1:
                    break
                print(f'Relocation modified at: 0x{offset:08x}')
                vmx.seek(offset)
                vmx.write(packed_new_ptr)
                offset += 1

        print()

        # Flush and compute SHA-256 of patched file
        vmx.flush()
        vmx.seek(0)
        sha_after = _sha256(vmx.read())
        print(f'SHA256 (after) : {sha_after}')

        # Restore original file permissions after patching
        os.chmod(name, orig_mode)

        # Write checksum file (upstream WriteHashes feature)
        sha_file = name + '.sha256'
        with open(sha_file, 'w') as sf:
            sf.write(f'{sha_before}\n{sha_after}\n')
        print(f'Checksums written: {sha_file}')

        del vmx_bytes
        return True


def dumpkeys(vmx, offset: int, count: int) -> None:
    print(f'Table Offset : 0x{offset:08x}')
    print('Offset     Name Len Type Flag FuncPtr    Data')
    print('-------    ---- --- ---- ---- -------    ----')

    for _ in range(count):   # '_' — loop counter not used
        smc_key  = getkey(vmx, offset)
        smc_data = getdata(vmx, offset, smc_key)
        printkey(offset, smc_key, smc_data)
        offset += ROW_LENGTH


def dumpsmc(name: str) -> bool:
    """Dump both vSMC key tables from *name* to stdout."""

    if not _check_file_patchable(name):
        return False

    with open(name, 'rb') as vmx:
        vmx_bytes = vmx.read()

        try:
            smc0_header = _find_required(
                vmx_bytes, SMC_HEADER_V0, label='SMC_HEADER_V0') - 8
            smc1_header = _find_required(
                vmx_bytes, SMC_HEADER_V1, label='SMC_HEADER_V1') - 8
            smc0_key = _find_required(
                vmx_bytes, KEY_KEY, smc0_header, label='#KEY (table 0)')
            smc1_key = _find_required(
                vmx_bytes, KEY_KEY, smc1_header, label='#KEY (table 1)')
        except RuntimeError as exc:
            print(f'ERROR: {exc}')
            print(f'File {name} cannot be dumped — skipping.')
            return False

        del vmx_bytes

        print('\nappleSMCTableV0 (smc.version = "0")')
        hdr = gethdr(vmx, smc0_header)
        printhdr(smc0_header, hdr)
        dumpkeys(vmx, smc0_key, hdr[1])

        print('\nappleSMCTableV1 (smc.version = "1")')
        hdr = gethdr(vmx, smc1_header)
        printhdr(smc1_header, hdr)
        dumpkeys(vmx, smc1_key, hdr[1])
        return True


def checksmc(name: str) -> bool:
    """Print the patch status of *name*."""

    if not _check_file_patchable(name):
        print('Patch Status: Unknown (file not patchable)')
        return False

    with open(name, 'rb') as vmx:
        vmx_bytes = vmx.read()

        sha = _sha256(vmx_bytes)
        print(f'SHA256: {sha}')

        # Fast check using raw OSK data bytes (mirrors upstream checkPatch)
        osk0_present = OSK0 in vmx_bytes
        osk1_present = OSK1 in vmx_bytes

        if osk0_present and osk1_present:
            flag = True
        elif not osk0_present and not osk1_present:
            flag = False
        else:
            # One present, one missing — partial/corrupt state
            print('Patch Status: Unknown (partial patch detected)')
            return False

        print(f'Patch Status: {flag}')
        return flag


def print_usage() -> None:
    print('Usage:')
    print(f'  {sys.argv[0]} patch|dump|check <vmx-filename>')
    print()
    print('Commands:')
    print('  patch - Patch the vSMC table in vmx executable')
    print('  check - Check the vSMC table patch status')
    print('  dump  - Dump the vSMC key table to stdout')
    print()
    print('Argument:')
    print('  <vmx-filename>  ESXi vmx binary: vmx / vmx-debug / vmx-stats')


def main() -> int:
    if len(sys.argv) == 2 and sys.argv[1] in ('-h', '--help'):
        print_usage()
        return 0

    if len(sys.argv) != 3:
        print_usage()
        return 1

    cmd = sys.argv[1]
    filename = sys.argv[2]

    if cmd == 'patch':
        fn = patchsmc
    elif cmd == 'check':
        fn = checksmc
    elif cmd == 'dump':
        fn = dumpsmc
    else:
        print(f'Error: Unknown command "{cmd}".')
        print_usage()
        return 1

    if not os.path.exists(filename):
        print(f'Error: File not found: {filename}')
        return 1

    print(f'Filename: {filename}')
    try:
        ok = fn(filename)
    except OSError as exc:
        print(f'ERROR: OS error while processing {filename}: {exc}')
        return 1
    return 0 if ok else 1


if __name__ == '__main__':
    print('PatchSMC 4.0.7b (ESXi 6.7 / 7.x / 8 U3)')
    print('=========================================')
    print()
    sys.exit(main())

```

## patchvmkctl

```python
#!/usr/bin/env python3
# coding=utf-8

# SPDX-License-Identifier: MIT

"""
patchvmkctl - Patch or check the libvmkctl.so shared library for ESXi.

The 'applesmc' string in libvmkctl.so gates smcPresent reporting to vCenter.
Replacing it with 'vmkernel' (same 8-byte length, no null-terminator issue)
allows vCenter to start macOS guests on a patched ESXi host.

Usage:
    patchvmkctl patch|check <libvmkctl-filename>

Compatibility:
    ESXi 6.7, 7.0, 7.0 U1/U2/U3, 8.0, 8.0 U1/U2/U3 (ESXi 8 U3 latest)

Copyright (c) 2007-2026 David Parsons
Copyright (c) 2024-2026 Harry DS Alsyundawy - Alsyundawy IT Solution
License: MIT

Changelog:
    2026-06-13 4.0.7b alsyundawy  Return meaningful exit status for patch/check
                                  operations. Use S_IMODE() when restoring
                                  file permissions after writing. Keep already
                                  patched or unsupported libvmkctl.so as a
                                  non-fatal condition for optional ESXi builds.
    2026-06-10 4.0.7a alsyundawy  Preserve original file mode (chmod) after
                                  writing patched content back with open('wb')
                                  — open('wb') truncates then rewrites, which
                                  on some ESXi filesystem drivers may not
                                  honour the inode's existing permission bits
                                  correctly. Now explicitly calls os.chmod().
                                  Move backup() call to after patchability
                                  check so .bak is not created for files that
                                  cannot be patched (already patched or no
                                  applesmc string found).
    2026-06-10 4.0.7  alsyundawy  Reference: DrDonk/unlocker 4.2.8. Align with
                                  upstream PatchVMKCTL() in vmkctl.go: print
                                  before/after string when patching. Add
                                  SHA-256 before/after. Add backup. Add
                                  _check_file_patchable() guard. Update banner.
    2026-06-10 4.0.6  alsyundawy  Change shebang to python3. Replace bare
                                  open() with context managers. Fix shebang.
                                  Add module docstring and copyright.
    2023-01-09 4.0.5  drdonk     Original release.
"""

import hashlib
import os
import shutil
import stat
import sys

if sys.version_info < (3, 6):
    sys.stderr.write('You need Python 3.6 or later\n')
    sys.exit(1)


# ---------------------------------------------------------------------------
# Patch constants (matching upstream vmkctl.go)
# ---------------------------------------------------------------------------
APPLESMC = b'applesmc'   # 8 bytes — string to replace
VMKERNEL = b'vmkernel'   # 8 bytes — replacement string


# ---------------------------------------------------------------------------
# Helpers
# ---------------------------------------------------------------------------

def _sha256(data: bytes) -> str:
    """Return hex SHA-256 digest of *data*."""
    return hashlib.sha256(data).hexdigest()


def _check_file_patchable(name: str) -> bool:
    """Return True if *name* is a non-empty regular file."""
    if not os.path.isfile(name):
        print(f'SKIP: {name} — not a regular file. Skipping.')
        return False
    if os.path.getsize(name) == 0:
        print(f'SKIP: {name} — file is 0 bytes. Skipping.')
        return False
    return True


def _backup_file(name: str) -> str:
    """Create *name*.bak backup if it does not already exist."""
    backup = name + '.bak'
    if not os.path.exists(backup):
        shutil.copy2(name, backup)
        print(f'Backup: {name} -> {backup}')
    else:
        print(f'Backup already exists: {backup} (skipped overwrite)')
    return backup


# ---------------------------------------------------------------------------
# Main functions
# ---------------------------------------------------------------------------

def checkvmkctl(name: str) -> bool:
    """
    Return True if libvmkctl.so is already patched (applesmc string absent).

    Mirrors upstream IsVMKCTLPatched() semantics.
    """
    if not _check_file_patchable(name):
        return False

    with open(name, 'rb') as fh:
        vmkctl = fh.read()

    sha = _sha256(vmkctl)
    print(f'SHA256: {sha}')

    patched = APPLESMC not in vmkctl
    print(f'Patch Status: {patched}')
    if patched:
        print('Note: applesmc string is absent; file is already patched or this ESXi build does not require vmkctl patching.')
    return patched


def patchvmkctl(name: str) -> bool:
    """
    Patch libvmkctl.so by replacing 'applesmc' with 'vmkernel'.

    Mirrors upstream PatchVMKCTL() in vmkctl.go:
        offset := bytes.Index(contents, APPLESMC)
        before := string(contents[offset : offset+8])
        copy(contents[offset:offset+8], VMKERNEL)
        after  := string(contents[offset : offset+8])
        fmt.Printf("Patching %s -> %s\n", before, after)
    """
    if not _check_file_patchable(name):
        return False

    # Capture original permissions before any write operation
    orig_mode = stat.S_IMODE(os.stat(name).st_mode)

    with open(name, 'rb') as fh:
        vmkctl = fh.read()

    sha_before = _sha256(vmkctl)
    print(f'SHA256 (before): {sha_before}')

    offset = vmkctl.find(APPLESMC)
    if offset == -1:
        print('applesmc string not found — file may already be patched '
              'or is not a supported libvmkctl.so.')
        print(f'Patch Status: {APPLESMC not in vmkctl}')
        return True

    before = vmkctl[offset:offset + 8].decode('ascii', errors='replace')
    print(f'Patching {before} -> {VMKERNEL.decode()}')

    # Backup before writing (only if file is actually going to be patched)
    _backup_file(name)

    # Patch in-memory using bytearray (safe, no seek needed)
    vmkctl_patched = bytearray(vmkctl)
    vmkctl_patched[offset:offset + 8] = VMKERNEL

    with open(name, 'wb') as fh:
        fh.write(vmkctl_patched)

    # Restore original file permissions (open 'wb' may reset on some FS)
    os.chmod(name, orig_mode)

    sha_after = _sha256(bytes(vmkctl_patched))
    print(f'SHA256 (after) : {sha_after}')

    # Write checksum file
    sha_file = name + '.sha256'
    with open(sha_file, 'w') as sf:
        sf.write(f'{sha_before}\n{sha_after}\n')
    print(f'Checksums written: {sha_file}')

    print('smcPresent patched successfully.')
    return True


def print_usage() -> None:
    print('Usage:')
    print(f'  {sys.argv[0]} patch|check <libvmkctl-filename>')
    print()
    print('Commands:')
    print('  patch - Patch smcPresent string in libvmkctl.so')
    print('  check - Check patch status of libvmkctl.so')
    print()
    print('Argument:')
    print('  <libvmkctl-filename>  Path to libvmkctl.so')


def main() -> int:
    if len(sys.argv) == 2 and sys.argv[1] in ('-h', '--help'):
        print_usage()
        return 0

    if len(sys.argv) != 3:
        print_usage()
        return 1

    cmd = sys.argv[1]
    filename = sys.argv[2]

    if cmd == 'check':
        fn = checkvmkctl
    elif cmd == 'patch':
        fn = patchvmkctl
    else:
        print(f'Error: Unknown command "{cmd}".')
        print_usage()
        return 1

    if not os.path.exists(filename):
        print(f'Error: File not found: {filename}')
        return 1

    print(f'Filename: {filename}')
    try:
        ok = fn(filename)
    except OSError as exc:
        print(f'ERROR: OS error while processing {filename}: {exc}')
        return 1
    return 0 if ok else 1


if __name__ == '__main__':
    print('PatchVMKCTL 4.0.7b (ESXi 6.7 / 7.x / 8 U3)')
    print('============================================')
    print()
    sys.exit(main())

```

## relock

```sh
#!/bin/sh
# SPDX-License-Identifier: MIT
#
# relock - Remove macOS unlock patches from VMware ESXi
#
# Description : Removes the apple.v00 VMTAR module from the ESXi bootbank,
#               reverting the host to stock (locked) state after reboot.
#
# Usage       : relock [-v|-h]
#   -v        : Enable verbose/debug output (set -x)
#   -h|--help : Show this help message
#
# Author      : drdonk / David Parsons
# Maintainer  : Harry DS Alsyundawy <alsyundawy@gmail.com>
# Copyright   : Copyright (c) 2007-2026 David Parsons
#               Copyright (c) 2024-2026 Harry DS Alsyundawy - Alsyundawy IT Solution
# License     : MIT
#
# Changelog   :
#   2026-06-13 4.0.7b alsyundawy  Add BootModuleConfig.sh command validation,
#                                 clearer version banner, and stricter error
#                                 reporting if apple.v00 removal fails.
#   2026-06-10 4.0.7a alsyundawy  Clean up changelog comments.
#   2026-06-10 4.0.7  alsyundawy  Update banner to 4.0.7.
#   2026-06-10 4.0.6  alsyundawy  Replace read -p with POSIX printf+read.
#                                 Add exit 0. Add docblock. Color output.
#   2023-01-09 4.0.5  drdonk     Fixed permissions error correctly this time!
#   2022-08-03 4.0.0  drdonk     Initial release.

export PATH=/bin:/sbin:/usr/bin:/usr/sbin

RED='\033[41m\033[37m'
GRN='\033[32m'
YLW='\033[33m'
RST='\033[0m'

die() {
    printf '%s\n' "${RED}>>> ERROR: $* <<<${RST}" >&2
    exit 1
}

warn() {
    printf '%s\n' "${YLW}WARNING: $*${RST}"
}

usage() {
    printf 'Usage: %s [-v|-h]\n' "$0"
    printf '  -v        Enable verbose/debug output\n'
    printf '  -h|--help Show this help message\n'
}

if [ $# -gt 1 ]; then
    usage
    die 'Too many arguments'
fi

if [ $# -eq 1 ]; then
    case "$1" in
        -v)
            printf 'Verbose mode enabled\n'
            set -x
            ;;
        -h|--help)
            usage
            exit 0
            ;;
        *)
            usage
            die "Invalid option: $1"
            ;;
    esac
fi

command -v BootModuleConfig.sh >/dev/null 2>&1 || die 'BootModuleConfig.sh not found'

printf 'VMware ESXi Unlocker 4.0.7b\n'
printf '===========================\n'
printf '\nRemoving unlocker...\n'

if [ -f '/bootbank/apple.v00' ]; then
    printf 'Removing apple.v00 from bootbank...\n'
    BootModuleConfig.sh --verbose --remove=apple.v00 || die 'Failed to remove apple.v00 from bootbank'
    printf '%s\n' "${GRN}apple.v00 removed from bootbank.${RST}"
else
    warn 'apple.v00 not found in bootbank — host may already be in locked state'
fi

printf '\nReboot the ESXi server to complete the relock (y/N)? '
read -r response
case "$response" in
    [yY])
        printf 'Rebooting the ESXi server now...\n'
        reboot
        ;;
    *)
        printf '%s\n' "${YLW}NOTE: Patches will remain active until the ESXi server is rebooted.${RST}"
        ;;
esac

exit 0

```

## unlock

```sh
#!/bin/sh
# SPDX-License-Identifier: MIT
#
# unlock - Apply macOS unlock patches to VMware ESXi
#
# Description : Patches vmx, vmx-debug, vmx-stats (if non-empty), and
#               libvmkctl.so (if present) to enable macOS guest support on
#               compatible VMware ESXi hosts.
#
#               This ESXi shell/Python port is fork-specific. Upstream
#               DrDonk/unlocker 4.x is currently a Go-based Workstation/Player
#               project; only relevant patching logic is used as reference.
#
# Usage       : unlock [-v|-h]
#   -v        : Enable verbose/debug output (set -x)
#   -h|--help : Show this help message
#
# Requirements: ESXi host in Maintenance Mode, no running VMs,
#               at least 400 MB free disk space on the datastore.
#
# Author      : drdonk / David Parsons
# Maintainer  : Harry DS Alsyundawy <alsyundawy@gmail.com>
# Copyright   : Copyright (c) 2007-2026 David Parsons
#               Copyright (c) 2024-2026 Harry DS Alsyundawy - Alsyundawy IT Solution
# License     : MIT
#
# Changelog   :
#   2026-06-13 4.0.7b alsyundawy  Define ANSI colors before first use. Add
#                                 command preflight checks, fatal vmware -v
#                                 guard, numeric disk-space validation, running
#                                 VM guard, Maintenance Mode warning, portable
#                                 audit-file purge without find -delete, strict
#                                 error handling for required vmx/vmx-debug
#                                 patch steps, and stale apple.v00 replacement
#                                 guard before BootModuleConfig --add.
#   2026-06-10 4.0.7a alsyundawy  Fix critical bug: chmod/tar wildcard glob in
#                                 staging included .bak and .sha256 audit files.
#                                 Add libvmkctl.so guard and python3 detection.
#   2026-06-10 4.0.7  alsyundawy  Reference DrDonk/unlocker 4.2.8. vmx-stats
#                                 existence check, backup, SHA-256, ESXi version
#                                 detection, -h/--help flag.
#   2026-06-10 4.0.6  alsyundawy  Fix critical bug: $1 arg on cp/rm/vmtar/pigz.
#                                 POSIX sh: read -p, [[ ]], ==. Subshell tar.
#                                 ./* glob. cd || exit guard.
#   2023-01-09 4.0.5  drdonk     Fixed permissions error correctly this time!
#   2023-01-03 4.0.4  drdonk     check/unlock free space and v3 detection.
#   2022-10-22 4.0.3  drdonk     Reinstate libvmkctl patch and boot time load.
#   2022-09-22 4.0.2  drdonk     Revert to binary read/writes in patchsmc.
#   2022-01-26 4.0.1  drdonk     Fix missing +x bit on checksmc, dumpsmc, relock.
#   2022-08-03 4.0.0  drdonk     Initial release.

export PATH=/bin:/sbin:/usr/bin:/usr/sbin
umask 022

RED='\033[41m\033[37m'
GRN='\033[32m'
YLW='\033[33m'
RST='\033[0m'

die() {
    printf '%s\n' "${RED}>>> ERROR: $* <<<${RST}" >&2
    exit 1
}

warn() {
    printf '%s\n' "${YLW}WARNING: $*${RST}"
}

need_cmd() {
    command -v "$1" >/dev/null 2>&1 || die "Required command not found: $1"
}

resolve_script() {
    if command -v readlink >/dev/null 2>&1; then
        _resolved=$(readlink -f "$0" 2>/dev/null || true)
        if [ -n "$_resolved" ]; then
            printf '%s\n' "$_resolved"
            return 0
        fi
    fi

    case "$0" in
        /*) printf '%s\n' "$0" ;;
        *)  printf '%s/%s\n' "$(pwd -P)" "$0" ;;
    esac
}

detect_python3() {
    for _py in /usr/bin/python3 \
               /usr/lib/vmware/venv-vsphere/bin/python3 \
               /usr/lib/vmware/venv-65/bin/python3; do
        if [ -x "$_py" ]; then
            printf '%s\n' "$_py"
            return 0
        fi
    done
    return 1
}

usage() {
    printf 'Usage: %s [-v|-h]\n' "$0"
    printf '  -v        Enable verbose/debug output\n'
    printf '  -h|--help Show this help message\n'
}

if [ $# -gt 1 ]; then
    usage
    die 'Too many arguments'
fi

if [ $# -eq 1 ]; then
    case "$1" in
        -v)
            printf 'Verbose mode enabled\n'
            set -x
            ;;
        -h|--help)
            usage
            exit 0
            ;;
        *)
            usage
            die "Invalid option: $1"
            ;;
    esac
fi

printf 'VMware ESXi Unlocker 4.0.7b\n'
printf '===========================\n'
printf 'Compatibility: ESXi 6.7 / 7.x / 8 U3\n'
printf 'Reference    : DrDonk/unlocker 4.x patch logic; ESXi port is fork-specific\n'
printf '\nInstalling unlocker...\n'

# Required ESXi/base commands. vim-cmd/reboot are intentionally not mandatory
# here because they are only used for best-effort warnings or optional reboot.
for _cmd in vmware df awk tail cp chmod rm mkdir tar vmtar pigz mv \
            BootModuleConfig.sh find grep esxcli; do
    need_cmd "$_cmd"
done

PYTHON3=$(detect_python3) || die 'python3 not found on this ESXi host'
printf 'Python3 path : %s\n' "$PYTHON3"

SCRIPT=$(resolve_script)
SCRIPTPATH=$(dirname "$SCRIPT")
cd "$SCRIPTPATH" || die "Cannot change to script directory: $SCRIPTPATH"

ESXI_VERSION=$(vmware -v 2>/dev/null || true)
[ -n "$ESXI_VERSION" ] || die "Could not determine ESXi version via 'vmware -v'"
printf 'ESXi version : %s\n' "$ESXI_VERSION"

REQUIRED=400
FREE=$(df -m . 2>/dev/null | awk 'NR==2 {print $4}')
case "$FREE" in
    ''|*[!0-9]*) die "Could not parse free disk space from 'df -m .'" ;;
esac

if [ "$FREE" -lt "$REQUIRED" ]; then
    die "Not enough free space: ${FREE} MB free, ${REQUIRED} MB required"
fi
printf 'Disk space   : %s MB free (OK)\n' "$FREE"

if esxcli vm process list 2>/dev/null | grep -q '^World ID:'; then
    die 'Running VMs detected. Shut down/migrate all VMs before patching ESXi binaries'
fi

if command -v vim-cmd >/dev/null 2>&1; then
    _maintenance=$(vim-cmd hostsvc/hostsummary 2>/dev/null | \
        awk -F'= ' '/inMaintenanceMode/ {gsub(",", "", $2); print $2; exit}')
    if [ "$_maintenance" != 'true' ]; then
        warn 'Host does not appear to be in Maintenance Mode; continue only for controlled lab/maintenance use'
    fi
fi

if [ -f '/bootbank/unlocker.tgz' ]; then
    die 'ESXi Unlocker v3 is installed and must be removed first: BootModuleConfig.sh --verbose --remove=unlocker.tgz, then reboot'
fi

rm -rf ./tmp 2>/dev/null || true
rm -f ./apple.* 2>/dev/null || true

mkdir -p ./tmp/bin ./tmp/lib64 ./tmp/etc || die 'Could not create staging directories'

printf '\nCopying ESXi binaries to staging area...\n'
[ -f /bin/vmx ]       && [ -s /bin/vmx ]       || die '/bin/vmx is missing or empty'
[ -f /bin/vmx-debug ] && [ -s /bin/vmx-debug ] || die '/bin/vmx-debug is missing or empty'

cp /bin/vmx ./tmp/bin/ || die 'Failed to copy /bin/vmx'
cp /bin/vmx-debug ./tmp/bin/ || die 'Failed to copy /bin/vmx-debug'

if [ -f '/bin/vmx-stats' ] && [ -s '/bin/vmx-stats' ]; then
    cp /bin/vmx-stats ./tmp/bin/ || die 'Failed to copy /bin/vmx-stats'
    printf '%s\n' "${GRN}vmx-stats found and copied.${RST}"
else
    printf '%s\n' "${YLW}SKIP: /bin/vmx-stats is missing or empty — skipping optional vmx-stats patch.${RST}"
fi

if [ -f '/lib64/libvmkctl.so' ] && [ -s '/lib64/libvmkctl.so' ]; then
    cp /lib64/libvmkctl.so ./tmp/lib64/ || die 'Failed to copy /lib64/libvmkctl.so'
    printf '%s\n' "${GRN}libvmkctl.so found and copied.${RST}"
else
    printf '%s\n' "${YLW}SKIP: /lib64/libvmkctl.so missing or empty — optional vmkctl patch not applicable.${RST}"
fi

chmod 4755 ./tmp/bin/vmx ./tmp/bin/vmx-debug || die 'Failed to chmod staged vmx binaries writable/setuid'
[ -f ./tmp/bin/vmx-stats ] && chmod 4755 ./tmp/bin/vmx-stats || true
[ -f ./tmp/lib64/libvmkctl.so ] && chmod 0755 ./tmp/lib64/libvmkctl.so || true

printf '%s\n' "$ESXI_VERSION" > ./tmp/etc/unlock.conf || die 'Failed to write staged unlock.conf'

printf '\nPatching vmx binaries...\n'
"$PYTHON3" ./patchsmc patch ./tmp/bin/vmx || die 'patchsmc failed for vmx'
"$PYTHON3" ./patchsmc patch ./tmp/bin/vmx-debug || die 'patchsmc failed for vmx-debug'

if [ -f './tmp/bin/vmx-stats' ] && [ -s './tmp/bin/vmx-stats' ]; then
    "$PYTHON3" ./patchsmc patch ./tmp/bin/vmx-stats || warn 'patchsmc failed for optional vmx-stats; continuing without failing required vmx patch'
else
    printf '%s\n' "${YLW}SKIP: vmx-stats not in staging — skipping patch.${RST}"
fi

if [ -f './tmp/lib64/libvmkctl.so' ]; then
    printf '\nPatching libvmkctl.so...\n'
    "$PYTHON3" ./patchvmkctl patch ./tmp/lib64/libvmkctl.so || warn 'patchvmkctl failed; vCenter macOS guest start may require manual review'
fi

# Audit files are useful when patching binaries directly, but staged .bak and
# .sha256 files must not enter apple.v00. Use POSIX find -exec instead of
# non-portable find -delete for BusyBox/ESXi compatibility.
find ./tmp \( -name '*.bak' -o -name '*.sha256' \) -type f -exec rm -f {} \; 2>/dev/null || \
    die 'Failed to purge staged audit files before building apple.v00'

chmod 4555 ./tmp/bin/vmx ./tmp/bin/vmx-debug || die 'Failed to set final vmx permissions'
[ -f ./tmp/bin/vmx-stats ] && chmod 4555 ./tmp/bin/vmx-stats || true
[ -f ./tmp/lib64/libvmkctl.so ] && chmod 0555 ./tmp/lib64/libvmkctl.so || true

printf '\nBuilding apple.v00 VMTAR file...\n'
( cd ./tmp && tar cf ../apple.tar ./* ) || die 'Failed to create apple.tar'
vmtar -c apple.tar -o apple.vtar 2>/dev/null || die 'Failed to create apple.vtar'
pigz -f -9 -n -T -k -S .v00 apple.vtar || die 'Failed to compress apple.vtar to .v00'
[ -f apple.vtar.v00 ] || die 'Expected apple.vtar.v00 was not created'
mv -f apple.vtar.v00 apple.v00 || die 'Failed to create apple.v00'
[ -s apple.v00 ] || die 'apple.v00 is missing or empty'

printf 'Cleaning up temporary files...\n'
rm -rf ./tmp
rm -f apple.tar apple.vtar

printf '\nAdding apple.v00 to bootbank...\n'
if [ -f '/bootbank/apple.v00' ]; then
    warn 'Existing /bootbank/apple.v00 detected; removing old module before adding the new one'
    BootModuleConfig.sh --verbose --remove=apple.v00 || warn 'Old apple.v00 remove reported an error; attempting add anyway'
fi
BootModuleConfig.sh --verbose --add=apple.v00 || die 'BootModuleConfig add failed for apple.v00'

printf '\n%s\n' "${GRN}Unlocker 4.0.7b installed successfully.${RST}"
printf 'ESXi version patched: %s\n' "$ESXI_VERSION"

printf '\nReboot the ESXi server to activate the unlocker (y/N)? '
read -r response
case "$response" in
    [yY])
        printf 'Rebooting the ESXi server now...\n'
        reboot
        ;;
    *)
        printf '%s\n' "${YLW}NOTE: SMC patches will NOT be active until the ESXi server is rebooted.${RST}"
        ;;
esac

exit 0

```

