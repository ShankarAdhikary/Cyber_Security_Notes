# Level 1: Linux Fundamentals and Architecture

## Goal
Understand what Linux is, how its layers fit together, and how to identify your system.

---

## 1.1 What Linux is

**Concepts**
- Linux is a **kernel**, not a complete operating system. A full OS is the kernel plus system libraries, tools, and utilities (usually from GNU).
- A **distribution** (distro) packages the kernel with software, a package manager, and defaults. Examples: Ubuntu, Debian, Fedora, Arch, Kali.
- Linux is open source, so its code can be audited. This matters for security research.

**Key commands**
```bash
uname -a              # kernel name, version, architecture
cat /etc/os-release   # distribution name and version
hostnamectl           # hostname, OS, kernel (systemd systems)
```

**Security relevance:** Knowing the exact kernel and distro version tells you which known vulnerabilities may apply.

**Practice**
- Run the three commands above and write down your kernel version and distro.
- Find out your distro's release date and support end date.

---

## 1.2 The architecture layers

Linux is organized in layers. Each layer talks only to the one next to it.

```
┌──────────────────────────────┐
│   Applications (user space)  │  browser, ssh, nginx, your scripts
├──────────────────────────────┤
│   Shell and utilities        │  bash, zsh, ls, grep, systemctl
├──────────────────────────────┤
│   System libraries           │  glibc, libssl, and similar
├──────────────────────────────┤  ← system call boundary
│   Linux kernel (kernel space)│  process, memory, file, network, device drivers
├──────────────────────────────┤
│   Hardware                   │  CPU, RAM, disk, network card
└──────────────────────────────┘
```

**Concepts**
- **Hardware:** physical components. The kernel controls them through drivers.
- **Kernel:** manages CPU scheduling, memory, filesystems, networking, and devices. It is the only layer with full hardware access.
- **System libraries:** shared code (such as the C library) that programs use to request kernel services.
- **Shell:** the program that reads your commands and runs them.
- **Applications:** everything else you run.

**Key commands**
```bash
lsb_release -a       # distro info (may need the lsb-release package)
lscpu                # CPU information
free -h              # memory usage
lsblk                # block devices (disks and partitions)
```

---

## 1.3 Kernel space vs. user space

**Concepts**
- **Kernel space:** where the kernel runs. Code here has full hardware access. A bug here can crash the whole system.
- **User space:** where normal programs run. They have limited access and must ask the kernel for anything privileged.
- The **system call interface** is the controlled gate between the two (covered in Level 3).

**Security relevance:** Most exploits aim to escape user space and gain kernel-level control. Privilege escalation is the attacker moving from user space toward root.

**Practice**
- Run `ps aux | head` and note which processes run as root and which run as normal users.

---

## 1.4 Multi-user and multitasking design

**Concepts**
- Linux is multi-user: many accounts can use one system, each isolated by permissions.
- Linux is multitasking: many processes run at the same time, scheduled by the kernel.
- The root account (UID 0) has unrestricted access. Most of the security work in Linux is about limiting root.

**Key commands**
```bash
whoami
id                   # your UID, GID, and groups
who                  # users logged in right now
```

---

## 1.5 Virtual machine lab setup

**Concepts**
- Practice in a VM so mistakes do not affect your main system.
- Options: VirtualBox, VMware Workstation Player, or UTM (Apple Silicon).
- Use **Ubuntu or Debian** for learning the architecture, and **Kali Linux** for security tools.

**Practice**
- Install a VM with Ubuntu Server or Desktop.
- Take a snapshot after a clean install. You can restore it when you break something.

---

## Checklist
- [ ] I can explain the difference between the kernel and a distribution.
- [ ] I can draw the five architecture layers from memory.
- [ ] I can identify my kernel version, distro, and CPU/memory info.
- [ ] I can explain kernel space vs. user space.
- [ ] I have a working VM lab with a snapshot.

## Further reading
- Linux Foundation free courses (LFS101 introduction)
- The Linux Documentation Project (tldp.org)
