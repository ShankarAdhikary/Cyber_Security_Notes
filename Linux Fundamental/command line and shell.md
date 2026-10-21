# Level 5: Command Line Interface and Shell

## Goal
Work fluently in the terminal. This is the most important daily skill in security work.

---

## 5.1 Shells and the terminal

**Concepts**
- A **terminal emulator** is the window. A **shell** is the program that interprets your commands.
- Common shells: `bash` (default on most distros), `zsh` (default on macOS), `sh` (POSIX, minimal).
- The shell reads `/etc/passwd` to know your default shell.

**Key commands**
```bash
echo $SHELL                  # your login shell
ps -p $$                     # the shell currently running
cat /etc/shells              # shells installed on the system
```

---

## 5.2 Navigation and files

**Key commands**
```bash
pwd                          # current directory
cd /var/log                  # change directory
cd -                         # go back to previous directory
ls -lah                      # detailed listing with hidden files
tree -L 2                    # directory tree (install tree if needed)
file <name>                  # identify file type
```

**Practice**
- Navigate to `/var/log` and list the files by size using `ls -lhS`.

---

## 5.3 Reading and searching files

**Key commands**
```bash
cat file.txt                 # print the whole file
less file.txt                # page through it (q to quit, / to search)
head -n 20 file.txt
tail -f /var/log/syslog      # follow a growing file
grep -i "error" file.txt     # search (case-insensitive)
grep -r "password" /etc 2>/dev/null   # search recursively
find / -name "*.conf" 2>/dev/null     # find files by name
find /home -type f -perm -o+w 2>/dev/null   # world-writable files under /home
```

**Security relevance:** `grep -r` and `find` are the core tools for searching a compromised or unfamiliar system for secrets, misconfigurations, and suspicious files.

---

## 5.4 Pipes and redirection

**Concepts**
- `|` (pipe) sends the output of one command into another.
- `>` writes output to a file (overwrites). `>>` appends.
- `2>` redirects errors. `2>/dev/null` discards them.
- `&>` or `> file 2>&1` sends both output and errors to a file.
- `<` feeds a file into a command as input.

**Key commands**
```bash
ps aux | grep sshd | grep -v grep
cat /etc/passwd | cut -d: -f1,7        # usernames and login shells
sort file.txt | uniq -c | sort -rn     # count duplicate lines
find / -type f -name "*.log" 2>/dev/null > logs.txt
```

**Practice**
- List all users who have a login shell (not `/usr/sbin/nologin` or `/bin/false`) using `cut`, `grep`, and a pipe.

---

## 5.5 Text processing tools

| Tool | Use |
|---|---|
| `cut` | pull out fields or columns |
| `awk` | field-based processing and logic |
| `sed` | find and replace, stream editing |
| `sort`, `uniq` | ordering and deduplication |
| `wc` | counting lines, words, bytes |
| `xargs` | turn input into command arguments |

**Key commands**
```bash
awk -F: '{print $1, $3}' /etc/passwd    # username and UID
sed -n '1,10p' file.txt                  # first 10 lines
wc -l /etc/passwd
cat file.txt | xargs -I{} echo "Line: {}"
```

---

## 5.6 Environment variables and shell configuration

**Concepts**
- Environment variables (`PATH`, `HOME`, `USER`) control how programs run.
- `PATH` is the list of folders searched for commands. A malicious directory placed early in `PATH` can hijack commands.
- Variables set with `export` are passed to child processes.
- Startup files: `~/.bashrc` (interactive shells), `~/.profile` or `~/.bash_profile` (login shells), `/etc/profile` (system-wide).

**Key commands**
```bash
echo $PATH
env                          # all environment variables
export MYVAR="test"
echo $MYVAR
```

**Security relevance:** Attackers add entries to `~/.bashrc` or `~/.profile` to run code every time a user logs in. Check these files during investigations. A `.` or empty entry in `PATH` is a security risk.

**Practice**
- Print your `PATH` and check whether any directory listed is world-writable (`ls -ld <dir>`).

---

## 5.7 Command history and job control

**Key commands**
```bash
history | tail -20
!!                           # repeat last command
!$                           # last argument of previous command
Ctrl+R                       # search history
Ctrl+Z                       # suspend a foreground job
bg                           # continue suspended job in background
fg                           # bring job to foreground
jobs                         # list jobs
```

**Security relevance:** `~/.bash_history` can contain passwords typed on the command line. Also, attackers may clear history, so check for gaps or missing files.

---

## 5.8 Man pages and help

**Key commands**
```bash
man ls                       # manual page
man -k "network"             # search manual pages by keyword
ls --help
apropos ssh                  # similar to man -k
```

---

## Checklist
- [ ] I can navigate, list, and search files without a GUI.
- [ ] I can use pipes and redirection to combine commands.
- [ ] I can use `grep`, `find`, `cut`, `awk`, and `sed` for basic tasks.
- [ ] I can explain what `PATH` does and why it matters for security.
- [ ] I know which startup files run at login.
- [ ] I can use `man` to look up any command.

## Further reading
- "The Linux Command Line" by William Shotts (free online)
- explainshell.com (paste any command to see what each part does)
- OverTheWire "Bandit" wargame (the best hands-on practice for this level)
