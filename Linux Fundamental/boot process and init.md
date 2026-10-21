# Level 2: Boot Process and Init

## Goal
Trace what happens from pressing the power button to a login prompt, and know where attackers can interfere.

---

## 2.1 Boot stages overview

**Concepts**
The boot process runs in this order:

1. **Firmware** (BIOS or UEFI) initializes hardware and finds a boot device.
2. **Bootloader** (usually GRUB) loads the kernel.
3. **Kernel** starts, detects hardware, and mounts the root filesystem.
4. **initramfs** provides a temporary root filesystem with the drivers needed to mount the real one.
5. **init system** (systemd on most modern distros) starts services and targets.
6. **Login** through a console or display manager.

**Key commands**
```bash
systemd-analyze            # total boot time
systemd-analyze blame      # time taken by each unit
journalctl -b              # logs from the current boot
journalctl -b -1           # logs from the previous boot
```

---

## 2.2 BIOS vs. UEFI

**Concepts**
- **BIOS:** legacy firmware. Reads the first sector (MBR) of the disk.
- **UEFI:** modern firmware. Reads an EFI System Partition (ESP) and supports Secure Boot.
- **Secure Boot:** only signed bootloaders and kernels can run. It blocks many boot-level attacks.

**Key commands**
```bash
[ -d /sys/firmware/efi ] && echo "UEFI" || echo "BIOS"
sudo bootctl status        # systemd-boot status, if used
mokutil --sb-state         # Secure Boot state (needs mokutil package)
```

**Security relevance:** Bootkits and firmware implants run before the OS starts, so Secure Boot and firmware passwords matter.

---

## 2.3 GRUB bootloader

**Concepts**
- GRUB shows the boot menu and loads the kernel and initramfs.
- Its config is generated from `/etc/default/grub` and `/etc/grub.d/`.
- Boot parameters on the kernel command line control behavior (for example, `single` or `init=/bin/bash` for recovery).

**Key commands**
```bash
cat /etc/default/grub
sudo update-grub           # Debian/Ubuntu: regenerate grub.cfg
cat /proc/cmdline          # kernel parameters used for this boot
```

**Security relevance:** Anyone with physical access who can edit GRUB parameters can often get a root shell. Protect GRUB with a password on sensitive machines.

**Practice**
- Run `cat /proc/cmdline` and identify each parameter.
- Read `/etc/default/grub` and note the timeout and default entry.

---

## 2.4 initramfs

**Concepts**
- A compressed image loaded into memory that contains drivers and tools needed to find and mount the real root filesystem (for example, disk or LVM drivers).
- Rebuilt when kernels or drivers change.

**Key commands**
```bash
ls /boot                    # kernel and initramfs images
sudo update-initramfs -u    # Debian/Ubuntu: rebuild initramfs
lsinitramfs /boot/initrd.img-$(uname -r) | head   # list contents (Debian/Ubuntu)
```

---

## 2.5 init and systemd

**Concepts**
- **PID 1** is the first user-space process. On most modern systems it is `systemd`.
- systemd starts services in parallel and manages **targets** (groups of services, similar to old runlevels).
- Common targets: `multi-user.target` (text mode) and `graphical.target` (desktop).

**Key commands**
```bash
ps -p 1 -o pid,comm        # confirm PID 1 is systemd
systemctl get-default      # current default target
systemctl list-units --type=target
```

**Security relevance:** PID 1 runs as root. Malicious services and unit files can add persistence at boot.

---

## 2.6 Recovery and single-user mode

**Concepts**
- Recovery mode boots into a root shell for repairs.
- Use it to reset passwords or fix broken configs.

**Security relevance:** An attacker with boot access can use the same method to reset passwords. Hard-to-reach recovery shells are a hardening point.

**Practice (in your VM only)**
- Reboot with `systemctl rescue` and then `systemctl default` to return.
- Look at `/var/log/boot.log` or `journalctl -b` for errors.

---

## Checklist
- [ ] I can list the boot stages in order.
- [ ] I know whether my system uses BIOS or UEFI, and whether Secure Boot is on.
- [ ] I can read `/etc/default/grub` and explain the kernel parameters in `/proc/cmdline`.
- [ ] I can explain what initramfs does.
- [ ] I can confirm PID 1 and explain its role.
- [ ] I can use `systemd-analyze` and `journalctl -b`.

## Further reading
- ArchWiki: "Boot process" (very clear even if you use another distro)
- `man systemd.special` for targets
