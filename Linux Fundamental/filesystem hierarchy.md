# Level 4: Filesystem Hierarchy

## Goal
Know where everything lives on a Linux system, how disks are mounted, and where attackers and defenders look first.

---

## 4.1 The Filesystem Hierarchy Standard (FHS)

**Concepts**
Linux has a single tree starting at `/` (root). Every disk, partition, and device appears somewhere in this tree. The FHS defines what each top-level directory is for.

| Directory | Purpose | Security note |
|---|---|---|
| `/bin`, `/sbin` | essential binaries (on modern systems often symlinks to `/usr/bin`, `/usr/sbin`) | verify integrity of system binaries |
| `/boot` | kernel, initramfs, bootloader files | tampering here = persistence |
| `/dev` | device files | raw disk access possible if permissions are weak |
| `/etc` | system-wide configuration | secrets, SSH config, sudoers, cron |
| `/home` | user home directories | user data, SSH keys, shell history |
| `/lib`, `/lib64` | shared libraries and kernel modules | library hijacking target |
| `/media`, `/mnt` | removable and temporary mount points | |
| `/opt` | optional third-party software | |
| `/proc` | process and kernel info (Level 3) | enumeration source |
| `/root` | root user's home | |
| `/run` | runtime data (PID files, sockets) | temporary, cleared on boot |
| `/srv` | data served by the system (web, FTP) | |
| `/sys` | device and kernel objects (Level 3) | |
| `/tmp` | temporary files, world-writable | common place for attacker payloads |
| `/usr` | most user programs and libraries | |
| `/var` | variable data: logs, spool, caches, databases | logs are key for incident response |

**Key commands**
```bash
ls -la /
man hier                     # the FHS description on your system
du -sh /var/* 2>/dev/null | sort -h | tail   # largest items in /var
```

---

## 4.2 Important files in /etc

**Key files**
| File | Contents |
|---|---|
| `/etc/passwd` | user accounts |
| `/etc/shadow` | password hashes (root only) |
| `/etc/group` | groups |
| `/etc/sudoers` (and `/etc/sudoers.d/`) | sudo rules |
| `/etc/ssh/sshd_config` | SSH server settings |
| `/etc/fstab` | filesystems mounted at boot |
| `/etc/hosts` | local name resolution |
| `/etc/crontab` | system cron jobs |
| `/etc/os-release` | distro identity |

**Security relevance:** Review these files on any system you are assessing. Configuration mistakes here are among the most common findings.

**Practice**
- Run `ls -l /etc/passwd /etc/shadow /etc/sudoers` and explain why the permissions differ.

---

## 4.3 Mounts

**Concepts**
- **Mounting** attaches a filesystem to a directory in the tree.
- `/etc/fstab` defines mounts at boot. `mount` and `umount` handle them at runtime.
- Mount **options** control behavior, for example `noexec` (no binary execution), `nosuid` (ignore SUID bits), and `nodev` (no device files).

**Key commands**
```bash
mount                        # list all mounts
findmnt                      # tree view of mounts (easier to read)
cat /etc/fstab
sudo mount -o remount,noexec /tmp    # example: make /tmp non-executable
```

**Security relevance:** Mounting `/tmp`, `/var/tmp`, and `/dev/shm` with `noexec,nosuid,nodev` blocks many simple attacks, because attackers often drop and run payloads there. Check whether these mounts have those options.

**Practice**
- Run `findmnt /tmp` and see which options are set.
- Read `/etc/fstab` and identify the root filesystem and any swap entry.

---

## 4.4 Block devices, partitions, and filesystems

**Concepts**
- A **block device** (such as `/dev/sda` or `/dev/nvme0n1`) is a disk.
- A **partition** is a slice of a disk (`/dev/sda1`).
- A **filesystem** (ext4, xfs, btrfs) organizes files within a partition.
- **LVM** lets you combine and resize disks into logical volumes.

**Key commands**
```bash
lsblk -f                     # devices, partitions, filesystems, UUIDs
sudo blkid                   # UUIDs and types
df -hT                       # space used per filesystem
sudo fdisk -l                # partition tables (read-only listing)
```

---

## 4.5 Inodes and file metadata

**Concepts**
- Every file has an **inode** that stores its metadata: owner, permissions, size, timestamps, and data block locations.
- The file name is just a pointer to an inode.
- Hard links share an inode. Symbolic links point to a path.

**Key commands**
```bash
ls -li file.txt              # show inode number
stat file.txt                # full metadata
df -i                        # inode usage per filesystem
find / -xdev -type f -links +1 2>/dev/null | head   # files with hard links
```

**Security relevance:** Attackers sometimes use hard links or symlinks to trick privileged programs into writing to sensitive files. This is a classic symlink attack.

---

## 4.6 Special and temporary locations

**Key concepts**
- `/tmp` and `/var/tmp` are world-writable. The **sticky bit** (`t`) prevents users from deleting each other's files (`ls -ld /tmp` shows `drwxrwxrwt`).
- `/dev/shm` is memory-backed temporary storage.
- `/proc` and `/sys` are virtual and use no disk space.

---

## Checklist
- [ ] I can name the purpose of at least 12 top-level directories.
- [ ] I know the location of the main configuration files in `/etc`.
- [ ] I can read `/etc/fstab` and `findmnt` output.
- [ ] I can explain why `noexec` on `/tmp` is a security control.
- [ ] I can inspect inodes with `ls -li` and `stat`.
- [ ] I can explain what the sticky bit does on `/tmp`.

## Further reading
- `man hier`
- Linux Journey: "Filesystem Hierarchy" lesson
