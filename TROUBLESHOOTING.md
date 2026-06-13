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
