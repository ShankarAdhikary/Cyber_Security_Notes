# Level 8: Packages, Scripting, and Hardening

## Goal
Manage software safely, automate tasks with Bash, and understand the logging, remote access, and isolation controls that protect a Linux system.

---

## 8.1 Package managers

**Concepts**
- Packages are bundles of software with metadata (version, dependencies, checksums).
- Package managers install from **repositories** and verify packages with **GPG signatures**.

| Family | Package manager | Package format |
|---|---|---|
| Debian, Ubuntu, Kali | `apt`, `dpkg` | `.deb` |
| Fedora, RHEL, CentOS | `dnf`, `rpm` | `.rpm` |
| Arch | `pacman` | `.pkg.tar.zst` |

**Key commands (Debian/Ubuntu)**
```bash
sudo apt update              # refresh package lists
sudo apt upgrade             # upgrade installed packages
apt list --upgradable
apt show openssh-server      # package details
dpkg -l | grep ssh           # installed packages matching a name
dpkg -S /usr/bin/ssh         # which package owns a file
apt-cache policy openssh-server   # version and source
```

**Key commands (Fedora/RHEL)**
```bash
sudo dnf check-update
sudo dnf upgrade
rpm -qa | grep ssh
rpm -qf /usr/bin/ssh         # which package owns a file
```

**Security relevance:**
- Keep systems patched. Most successful attacks use known, unpatched vulnerabilities.
- Repositories are a supply chain. Only add trusted sources, and check that signing keys are present (`/etc/apt/trusted.gpg.d/`, `/etc/apt/sources.list.d/`).
- Use `dpkg -S` or `rpm -qf` to check whether a binary belongs to a package. A binary that belongs to no package is suspicious.

**Practice**
- Run `dpkg -S /bin/ls` (or `rpm -qf /bin/ls`) and confirm which package owns it.
- List your repositories with `cat /etc/apt/sources.list` and `ls /etc/apt/sources.list.d/`.

---

## 8.2 Bash scripting basics

**Concepts**
- A script starts with a **shebang**: `#!/bin/bash`.
- Make it executable with `chmod +x script.sh`, then run with `./script.sh`.

**Core syntax**
```bash
#!/bin/bash

NAME="student"                     # variable (no spaces around =)
echo "Hello, $NAME"

if [ -f /etc/passwd ]; then        # conditional
    echo "File exists"
elif [ -d /etc ]; then
    echo "Directory exists"
else
    echo "Neither"
fi

for user in alice bob; do          # loop
    echo "User: $user"
done

count=0
while [ $count -lt 3 ]; do         # while loop
    echo "Count: $count"
    count=$((count + 1))
done

check_root() {                     # function
    if [ "$EUID" -ne 0 ]; then
        echo "Not root"
        return 1
    fi
    echo "Running as root"
}
check_root
```

**Useful test operators**
| Operator | Meaning |
|---|---|
| `-f file` | file exists and is regular |
| `-d dir` | directory exists |
| `-r`, `-w`, `-x` | readable, writable, executable |
| `-z str` | string is empty |
| `-eq`, `-ne`, `-lt`, `-gt` | numeric comparison |

**Input and arguments**
```bash
#!/bin/bash
echo "First argument: $1"
echo "All arguments: $@"
echo "Number of arguments: $#"
read -p "Enter your name: " name
```

**Practice**
- Write a script that prints every user with UID 0 (use `awk` from Level 5).
- Write a script that checks whether a list of files exist and reports any that do not.

---

## 8.3 Security-focused scripting

**Good practices**
```bash
#!/bin/bash
set -euo pipefail        # stop on errors, undefined variables, and pipe failures
IFS=$'\n\t'              # safer word splitting

# Always quote variables
file="$1"
[ -f "$file" ] && echo "Found: $file"
```

**Security relevance:** Unquoted variables and unchecked input in scripts cause command injection and file-handling bugs. Also avoid running scripts as root unless necessary.

**Practice**
- Write an audit script that reports SUID files, UID 0 accounts, and world-writable directories. Save the output to a dated file.

---

## 8.4 Logging and auditing

**Concepts**
- `rsyslog` or `journald` collect logs. Log files are in `/var/log`.
- **auditd** records detailed events such as file access, command execution, and privilege changes.
- Logs are only useful if they are protected. Attackers try to delete or edit them.

**Key commands**
```bash
sudo systemctl status rsyslog
sudo auditctl -l                     # list audit rules (needs auditd)
sudo ausearch -k <key> | tail        # search audit events by key
sudo aureport --summary              # audit summary
```

**Example audit rule** (watch changes to `/etc/passwd`)
```bash
sudo auditctl -w /etc/passwd -p wa -k passwd_changes
```

**Security relevance:** Forward logs to a separate host where possible. Attackers who gain root can clear local logs, so remote logging makes evidence harder to destroy.

---

## 8.5 SSH hardening

**Concepts**
- SSH provides encrypted remote access. Configured in `/etc/ssh/sshd_config`.
- Key-based authentication is stronger than passwords.

**Important settings**
| Setting | Recommended value |
|---|---|
| `PermitRootLogin` | `no` |
| `PasswordAuthentication` | `no` (after keys work) |
| `PubkeyAuthentication` | `yes` |
| `MaxAuthTries` | `3` to `4` |
| `AllowUsers` | list of allowed users |

**Key commands**
```bash
sudo sshd -t                         # test config for syntax errors
sudo systemctl restart ssh           # apply changes (name may be sshd)
ssh-keygen -t ed25519 -C "lab key"   # generate a key pair
ssh-copy-id user@host                # install your public key on a server
sudo ss -tlnp | grep ssh             # confirm sshd is listening
```

Warning: Before disabling password login, confirm key login works in a second terminal. Otherwise you can lock yourself out.

**Security relevance:** Check `~/.ssh/authorized_keys` for unknown keys. Attackers often add their own key for persistence.

---

## 8.6 Mandatory access control: SELinux and AppArmor

**Concepts**
- **Discretionary access control (DAC):** the owner decides permissions. This is the standard `chmod` model from Level 7.
- **Mandatory access control (MAC):** a central policy restricts even root-owned processes. Examples are SELinux (RHEL, Fedora) and AppArmor (Ubuntu, Debian, SUSE).
- MAC can stop an exploited service from reading files outside its allowed scope.

**Key commands**
```bash
# AppArmor (Ubuntu/Debian)
sudo aa-status
# SELinux (RHEL/Fedora)
getenforce                  # Enforcing, Permissive, or Disabled
sestatus
```

**Security relevance:** Disabling SELinux or AppArmor to fix a problem removes a major security layer. Put profiles in complain mode while troubleshooting, then return to enforce mode.

---

## 8.7 Containers and isolation (introduction)

**Concepts**
- **Namespaces** isolate what a process can see (PIDs, network, mounts, users).
- **cgroups** limit what a process can use (CPU, memory).
- **chroot** changes a process's root directory. It is not a strong security boundary on its own.
- Docker and Kubernetes combine these features. Containers share the host kernel, so a kernel exploit can escape them.

**Key commands**
```bash
lsns                         # list namespaces
cat /proc/self/cgroup        # cgroups of your shell
ls -l /proc/self/ns/         # namespace links for your process
```

**Security relevance:** Running a container with `--privileged` or mounting the Docker socket gives near-root access to the host.

---

## 8.8 Keeping the system updated and reviewing it

**Routine tasks**
- Apply security updates on a schedule (`unattended-upgrades` on Debian/Ubuntu, `dnf-automatic` on Fedora/RHEL).
- Review listening services: `sudo ss -tulpn`.
- Review scheduled jobs and enabled services (Level 6).
- Review users, groups, and SUID files (Level 7).

---

## Checklist
- [ ] I can install, update, and remove packages and find which package owns a file.
- [ ] I can write a Bash script with variables, conditions, loops, and functions.
- [ ] I can use `set -euo pipefail` and quote variables.
- [ ] I can search audit logs and explain what auditd records.
- [ ] I can harden `sshd_config` and check authorized keys.
- [ ] I can explain DAC vs. MAC and check SELinux or AppArmor status.
- [ ] I can explain namespaces, cgroups, and why containers are not full isolation.

## Further reading
- Bash Guide for Beginners (tldp.org)
- ShellCheck (shellcheck.net) to check scripts for mistakes
- CIS Benchmarks for Linux (free PDFs for hardening checklists)
- Ubuntu Server Security documentation
