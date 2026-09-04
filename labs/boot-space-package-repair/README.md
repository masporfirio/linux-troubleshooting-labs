# Troubleshooting Note: apt and dpkg Recovery After `/boot` Filled

**Type:** Troubleshooting note from my lab environment.

## What happened

An upgrade in my Kali ARM64 virtual machine failed while creating a new initramfs. The useful error was:

```text
No space left on device
```

The package operation was left incomplete. This was a personal VM lab, not a company incident.

## Objective

Confirm which filesystem was full, reduce the initramfs size for this VM, finish the pending package configuration and record what was and was not verified.

## Investigation

### 1. Check the affected filesystem

```bash
df -h /boot
du -h /boot
ls -lh /boot
```

`df` showed the filesystem limit. `du` and `ls` helped explain which files used the space. The `/boot` partition was about 456 MB.

### 2. Check the running kernel and package state

```bash
uname -r
dpkg -l 'linux-image*'
sudo dpkg --audit
```

The running kernel check mattered because removing a kernel without confirming the active version could make recovery harder.

## Root cause

The new initramfs could not be written to the small `/boot` partition. The failure was storage-related; it was not fixed by simply running the upgrade again.

## Repair used in this VM

I kept the previous kernel and changed the initramfs module policy in `/etc/initramfs-tools/initramfs.conf` from the broad default to:

```text
MODULES=dep
```

For this VM, that reduced the generated initramfs to about 21 MB while keeping the modules needed by the guest. This setting is not a general fix for every physical machine. Hardware and recovery requirements should be checked first.

The pending package configuration was then completed:

```bash
sudo dpkg --configure -a
```

## Verification

```bash
sudo dpkg --audit
sudo apt-get check
df -h /boot
ls -lh /boot
```

`dpkg --audit` and `apt-get check` both completed without errors after the repair.

## Verification limit

The package and initramfs repair was verified in the running system. The newly prepared kernel was not boot-tested at that point, so this note does not claim that reboot verification was complete.

## What I learned

The last package name was not the root cause by itself. Reading the initramfs error and checking `/boot` connected the package failure to the storage limit. I also learned to report package-state verification separately from a successful reboot.
