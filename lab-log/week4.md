# Week 4 – Install, configure and troubleshoot bootloaders

This week I focused on deepening my understanding of the Linux boot workflow, covering safe shutdown and reboot procedures, techniques for interrupting the boot sequence to perform critical recovery tasks such as resetting the root password, and methods for repairing and reconfiguring the GRUB bootloader.

## 1. Safe Shutdown and Reboot 

Shutting down or rebooting properly ensures data integrity and avoids filesystem corruption. So, it's important to understand how to do so!

### Examples

```bash
# Schedule a reboot at midnight, one day from now, with a wall message
sudo shutdown -r 00:00 +1 "Scheduled shutdown at 12 am"

# Shutdown in 15 minutes
sudo shutdown +15

# Immediate reboot/poweroff with systemd
sudo systemctl reboot
sudo systemctl poweroff

# Force reboot (skips graceful shutdown)
sudo systemctl reboot --force
sudo systemctl reboot --force --force

```


## 2. Interrupting Boot to Reset Root Password (RHEL 9)

I also learned that you can hijack the boot process at GRUB to drop into a shell.

This is an example of a case where you may need to do so to reset the root password when you can't log in normally.

### Steps

1. At the GRUB menu, press `e` to edit boot parameters.
2. Find the line starting with 'Linux'
- Change `ro` (aka read-only) to `rw` (read-write).
- Move to the end of the line `ctrl+e`
- Append: init=/bin/bash
3. Press Enter
4. At the shell prompt:

```bash
# Changes the root password
passwd root

# Ensures SELinux relabels files on next reboot
touch /.autorelabel

# Finish booting:
exec /sbin/init

```

## 3. Repairing the Boot Loader (GRUB)

If the bootloader is broken and the OS won’t start, use rescue mode.

### Steps

```bash

# Boot from installation media
# Choose: Troubleshooting → Rescue a CentOS system
# Root is mounted at /mnt/sysroot

# Enter chroot
chroot /mnt/sysroot

# Rebuild GRUB config
grub2-mkconfig -o /boot/grub2/grub.cfg
# For EFI systems:
grub2-mkconfig -o /boot/efi/EFI/centos/grub.cfg

# Identify boot disk
lsblk

# Reinstall GRUB to disk (usually /dev/sda)
grub2-install /dev/sda

# On EFI systems
dnf reinstall grub2-efi grub2-efi-modules shim

# Exit chroot and reboot

```

## 4. Configuring the GRUB Bootloader

GRUB controls boot menu and kernel parameters.

### Steps

```bash
# Edit defaults
sudo vi /etc/default/grub

# Common options:
# GRUB_TIMEOUT=1 → menu shows for 1 second
# GRUB_CMDLINE_LINUX="..." → kernel parameters (e.g., nomodeset, single, quiet)

# Example:
GRUB_CMDLINE_LINUX="rd.lvm.lv=centos/root crashkernel=auto rhgb quiet"

# Rebuild GRUB config
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
# EFI systems:
sudo grub2-mkconfig -o /boot/efi/EFI/centos/grub.cfg

# Reboot to apply changes
sudo systemctl reboot

```
