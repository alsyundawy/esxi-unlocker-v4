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
