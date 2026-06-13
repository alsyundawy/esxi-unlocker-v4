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
