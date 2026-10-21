# Linux Architecture Notes for Cybersecurity

Study notes for the **Linux Architecture** stage of the cybersecurity roadmap. The material is split into 8 levels, each with sub-topics, key concepts, commands, security relevance, and hands-on practice.

This README explains how the notes are organized and how to use them.

---

## Folder structure

```
linux-architecture-notes/
├── README.md                          ← this guide
├── 01-linux-fundamentals.md           ← Level 1
├── 02-boot-process-and-init.md        ← Level 2
├── 03-kernel-and-system-calls.md      ← Level 3
├── 04-filesystem-hierarchy.md         ← Level 4
├── 05-command-line-and-shell.md       ← Level 5
├── 06-processes-and-services.md       ← Level 6
├── 07-users-and-permissions.md        ← Level 7
└── 08-packages-scripting-and-hardening.md  ← Level 8
```

---

## The 8 levels at a glance

| Level | File | Topic | Roadmap topic it covers |
|---|---|---|---|
| 1 | `01-linux-fundamentals.md` | Linux fundamentals and architecture layers | Linux Architecture (intro) |
| 2 | `02-boot-process-and-init.md` | Boot process, bootloader, init, systemd startup | Extra: Boot process |
| 3 | `03-kernel-and-system-calls.md` | Kernel, system calls, kernel modules, `/proc`, `/sys` | Extra: Kernel vs. user space, `/proc` and `/sys` |
| 4 | `04-filesystem-hierarchy.md` | Filesystem hierarchy, mounts, storage | **File System Hierarchy** |
| 5 | `05-command-line-and-shell.md` | Shell, CLI, pipes, redirection, environment | **Command Line Interface** |
| 6 | `06-processes-and-services.md` | Processes, signals, systemd services, logs, cron | **Process Management** + systemd, cron |
| 7 | `07-users-and-permissions.md` | Users, groups, permissions, sudo, PAM, ACLs | **File Permissions**, **User Management** + SUID/SGID, ACLs, sudoers |
| 8 | `08-packages-scripting-and-hardening.md` | Package managers, Bash scripting, logging, SSH, SELinux/AppArmor, containers | **Package Managers**, **Bash Scripting** + logging, SSH, MAC, containers |

Levels build on each other. Work in order the first time. Level 7 relies on Levels 4 and 6, and Level 8 uses everything before it.

---

## How each level file is structured

Every level file uses the same template so you always know where to find things:

1. **Goal:** what you should be able to do after this level
2. **Sub-topics:** each one has
   - **Concepts:** what it is and why it exists
   - **Key commands:** commands to type and try
   - **Security relevance:** how attackers use it and how defenders check it
   - **Practice:** hands-on tasks
3. **Checklist:** tick these off when you're confident
4. **Further reading:** official docs and resources

Sub-topic headings are numbered (for example `4.2 Mounts`) so you can refer to them in your own notes and commits.

---

## Suggested workflow

1. **Set up a lab first.** Use an Ubuntu or Debian VM, or Kali Linux for security tools. Never practice security techniques on systems you do not own or have permission to test.
2. **Read one sub-topic at a time.** Skim the concepts, then type every command yourself. Do not copy-paste blindly.
3. **Do the practice tasks** before moving on. If a task takes too long, write down what you tried and move on.
4. **Keep a personal log.** Add your own notes, errors, and solutions in a separate file or a `my-notes/` folder. Writing down mistakes is one of the fastest ways to learn.
5. **Tick the checklist** at the end of each level. Do not move on until most items are checked.
6. **Revisit older levels** every week or two with a short review. Commands fade quickly if not used.

---

## Time estimate

| Level | Suggested time |
|---|---|
| 1 | 2–3 days |
| 2 | 3–4 days |
| 3 | 4–5 days |
| 4 | 3–4 days |
| 5 | 1–2 weeks (the most important level for daily work) |
| 6 | 1 week |
| 7 | 1–2 weeks |
| 8 | 2 weeks |

These are rough estimates for about 1–2 hours of study per day. Go slower if you need to.

---

## Conventions used in the notes

- Commands appear in code blocks and are meant to be run on a Linux system. Commands marked `sudo` need administrator rights.
- Paths like `/etc/passwd` are shown in backticks.
- Warning: marks a common mistake or a dangerous command. Read these carefully.
- marks a security-relevant point, such as an attack technique or a hardening step.
- Replace placeholder values like `<username>` or `<file>` with your own.

---
