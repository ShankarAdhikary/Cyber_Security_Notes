# Level 7: Users, Groups, and Permissions

## Goal
Understand how Linux decides who can do what. This is the core of privilege escalation and hardening.

---

## 7.1 Users and UIDs

**Concepts**
- Every user has a **UID**. UID 0 is root, with unrestricted access.
- System accounts (typically UIDs below 1000) run services. Regular users usually start at 1000.
- Account data is stored in `/etc/passwd`, password hashes in `/etc/shadow`.

**`/etc/passwd` format** (seven colon-separated fields)
```
username:x:UID:GID:comment:home_directory:login_shell
```

**Key commands**
```bash
id                           # your UID, GID, groups
id <username>
getent passwd <username>
sudo useradd -m testuser     # create user with home directory
sudo passwd testuser         # set password
sudo userdel -r testuser     # delete user and home
```

**Security relevance:** Any UID 0 account other than root is a backdoor. Check with:
```bash
awk -F: '$3 == 0 {print $1}' /etc/passwd
```

---

## 7.2 Password storage

**Concepts**
- `/etc/shadow` holds the password hash and aging information. Only root can read it.
- The hash format `$id$salt$hash` shows the algorithm: `$1$` MD5 (weak), `$5$` SHA-256, `$6$` SHA-512 (common), `$y$` yescrypt (modern default).
- Accounts with `!` or `*` in the password field cannot log in with a password.

**Key commands**
```bash
sudo cat /etc/shadow
sudo chage -l <username>     # password aging info
```

**Security relevance:** Weak hash types, empty password fields, and accounts with no expiry are common audit findings.

---

## 7.3 Groups

**Concepts**
- Groups give several users shared access. Each user has a primary group and can belong to supplementary groups.
- Group membership is listed in `/etc/group`.
- Some groups grant powerful access: `sudo` or `wheel` (admin rights), `docker` (effectively root), `adm` (read logs).

**Key commands**
```bash
groups                       # your groups
getent group sudo            # members of the sudo group
sudo usermod -aG docker testuser    # add user to a group (-a = append)
sudo gpasswd -d testuser docker     # remove from a group
```

Warning: Membership in `docker` lets a user mount the host filesystem and become root. Treat it like sudo access.

---

## 7.4 Basic file permissions

**Concepts**
Each file has three permission sets: **owner**, **group**, and **others**. Each set can have:
- `r` (read) = 4
- `w` (write) = 2
- `x` (execute) = 1; on directories it means "enter/traverse"

Example: `rwxr-x---` = 750.

**Key commands**
```bash
ls -l file.txt
chmod 640 file.txt           # octal notation
chmod u+x script.sh          # symbolic notation
chown user:group file.txt
chown -R user:group dir/     # recursive
umask                        # default permissions for new files
```

**Common `umask` logic:** new files start at 666 and new directories at 777, then the umask bits are removed. With `umask 022`, files become 644 and directories 755.

**Practice**
- Create a file, set it to `600`, and confirm that another user cannot read it.
- Run `umask`, then create a file and a directory and compare their permissions.

---

## 7.5 Special permission bits

**Concepts**
- **SUID** (`4000`, shown as `s` in the owner execute slot): the program runs with the file owner's rights. `/usr/bin/passwd` is SUID root so users can change their password.
- **SGID** (`2000`): on files, runs with the group's rights. On directories, new files inherit the directory's group.
- **Sticky bit** (`1000`, shown as `t`): on directories, only the owner can delete their files. `/tmp` uses it.

**Key commands**
```bash
find / -perm -4000 -type f 2>/dev/null     # find all SUID files
find / -perm -2000 -type f 2>/dev/null     # find all SGID files
ls -l /usr/bin/passwd
```

**Security relevance:** SUID binaries are a major privilege-escalation vector. A custom or outdated SUID program owned by root can give a normal user root access. Audit the list regularly and compare it against a known-good list. Check the results against GTFOBins (gtfobins.github.io) to see if any are exploitable.

**Practice**
- Run the SUID find command and count the results. Check three of them on GTFOBins (in your lab only, to understand the risk).

---

## 7.6 Access Control Lists (ACLs)

**Concepts**
- ACLs allow permissions for specific users or groups beyond owner, group, and others.
- Requires filesystem support (most ext4 and xfs setups have it).

**Key commands**
```bash
getfacl file.txt
setfacl -m u:alice:r file.txt       # give alice read access
setfacl -x u:alice file.txt         # remove alice's entry
setfacl -b file.txt                 # remove all ACLs
```

ACLs are easy to forget during audits. A file can look locked down in `ls -l` but still be readable through an ACL. Always check `getfacl` too.

---

## 7.7 sudo and sudoers

**Concepts**
- `sudo` lets permitted users run commands as root (or another user) after authenticating with their own password.
- Rules live in `/etc/sudoers` and `/etc/sudoers.d/`. Always edit with `visudo`, which checks syntax first.
- Each rule states who can run what, as whom, and sometimes without a password (`NOPASSWD`).

**Key commands**
```bash
sudo -l                      # what you are allowed to run
sudo visudo                  # safe editor for sudoers
sudo -u <user> <command>     # run as another user
```

**Security relevance:** Entries such as `NOPASSWD: ALL` or allowed editors, interpreters, or file tools (like `vim`, `python`, or `find`) can be abused to get a root shell. Check every `sudo -l` result for these.

**Practice**
- In your VM, run `sudo -l` and identify each allowed command. Look each one up on GTFOBins to understand how it could be abused.

---

## 7.8 PAM (Pluggable Authentication Modules)

**Concepts**
- PAM decides how authentication works for login, sudo, SSH, and other services.
- Configs are in `/etc/pam.d/`. Each file lists rules for a service (for example, `/etc/pam.d/sshd`).
- Common modules: `pam_unix` (standard passwords), `pam_pwquality` (password strength), `pam_faillock` (lock accounts after failed attempts).

**Key commands**
```bash
ls /etc/pam.d/
cat /etc/pam.d/sudo
```

**Security relevance:** A modified PAM module can accept a backdoor password or log credentials. Attackers who gain root may tamper with PAM files for persistence.

---

## 7.9 Login and account policies

**Key files**
- `/etc/login.defs`: default password aging, UID ranges, and umask
- `/etc/security/limits.conf`: resource limits per user
- `/var/log/wtmp` and `last`: login history
- `/var/log/btmp` and `lastb`: failed logins (root only)

**Key commands**
```bash
last | head
sudo lastb | head
```

---

## Checklist
- [ ] I can read a line of `/etc/passwd` and explain each field.
- [ ] I can find all UID 0 accounts and explain why they matter.
- [ ] I can convert between `rwx` and octal notation without help.
- [ ] I can explain SUID, SGID, and the sticky bit with examples.
- [ ] I can list SUID files and explain why they are risky.
- [ ] I can read `sudo -l` output and spot dangerous entries.
- [ ] I can use `getfacl` and `setfacl`.
- [ ] I know what PAM does and where its config lives.

## Further reading
- GTFOBins (gtfobins.github.io)
- `man sudoers`, `man 5 shadow`, `man 5 passwd`
- Linux Journey: "Users and Groups" and "Permissions" lessons
