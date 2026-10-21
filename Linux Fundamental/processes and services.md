# Level 6: Processes and Services

## Goal
Monitor, control, and investigate running processes and services, and understand how they persist.

---

## 6.1 What a process is

**Concepts**
- A **process** is a running instance of a program. Each has a **PID** (process ID) and a **PPID** (parent PID).
- Every process is created by `fork()` and typically replaced with a new program by `exec()`.
- A **thread** is a unit of execution inside a process. Threads share memory.
- Process states: running (R), sleeping (S), stopped (T), zombie (Z), disk sleep (D).

**Key commands**
```bash
ps aux                       # all processes with user, CPU, memory
ps -ef --forest              # process tree
pstree -p                    # tree view with PIDs
echo $$                      # PID of your shell
```

---

## 6.2 Monitoring processes

**Key commands**
```bash
top                          # live view (q to quit)
htop                         # friendlier version (install if needed)
pgrep -a sshd                # find PIDs by name
ps -eo pid,user,cmd --sort=-%cpu | head    # top CPU users
lsof -p <PID>                # files opened by a process
```

**Security relevance:** Unexpected processes, processes running from `/tmp`, and processes with no parent or an odd parent are classic signs of compromise.

**Practice**
- Find the top three processes by memory with `ps aux --sort=-%mem | head -4`.
- Check where each one's binary lives using `ls -l /proc/<PID>/exe`.

---

## 6.3 Signals and killing processes

**Concepts**
- Signals are messages sent to processes. Common ones:

| Signal | Number | Meaning |
|---|---|---|
| `SIGHUP` | 1 | reload configuration (many daemons) |
| `SIGINT` | 2 | interrupt (Ctrl+C) |
| `SIGKILL` | 9 | force stop, cannot be caught |
| `SIGTERM` | 15 | polite stop request (default for `kill`) |
| `SIGSTOP` | 19 | pause the process |

**Key commands**
```bash
kill <PID>                   # sends SIGTERM
kill -9 <PID>                # sends SIGKILL (last resort)
pkill -f "python script.py"  # kill by matching command line
kill -l                      # list all signals
```

Warning: Killing a process with `kill -9` gives it no chance to clean up. Try `SIGTERM` first.

---

## 6.4 Priority and scheduling

**Key commands**
```bash
nice -n 10 ./script.sh       # start with lower priority
renice -n 5 -p <PID>         # change priority of a running process
```

Priority ranges from -20 (highest) to 19 (lowest).

---

## 6.5 systemd: units and services

**Concepts**
- systemd manages **units**. Common types: `.service` (daemons), `.timer` (scheduled tasks), `.socket`, `.target`, `.mount`.
- Unit files live in `/usr/lib/systemd/system/` (vendor defaults) and `/etc/systemd/system/` (local overrides and custom units).
- A service file has three sections: `[Unit]`, `[Service]`, `[Install]`.

**Key commands**
```bash
systemctl status ssh         # service status (name may be sshd on some distros)
sudo systemctl start ssh
sudo systemctl stop ssh
sudo systemctl enable ssh    # start at boot
sudo systemctl disable ssh
systemctl list-units --type=service --state=running
systemctl list-unit-files --state=enabled
systemctl cat ssh            # show the unit file
```

**Example unit file** (for practice in your VM only)
```ini
[Unit]
Description=My practice script

[Service]
ExecStart=/usr/local/bin/practice.sh
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Save as `/etc/systemd/system/practice.service`, then run `sudo systemctl daemon-reload` and `sudo systemctl enable --now practice`.

**Security relevance:** Enabled services that you do not recognize are a top persistence technique. Compare `systemctl list-unit-files --state=enabled` against a known-good baseline.

---

## 6.6 Logs with journalctl

**Concepts**
- systemd collects logs in a binary journal, readable with `journalctl`.
- Traditional text logs in `/var/log` may still exist (for example, `auth.log`, `syslog`, or `secure` on RHEL-based systems).

**Key commands**
```bash
journalctl -u ssh            # logs for one service
journalctl -f                # follow live logs
journalctl --since "1 hour ago"
journalctl -p err            # only errors and above
sudo journalctl _UID=0       # logs from root processes
ls /var/log
sudo tail -n 50 /var/log/auth.log    # Debian/Ubuntu login events
```

**Security relevance:** Failed logins, sudo use, and service restarts appear here. Incident responders start with these logs. Check `auth.log` for repeated `Failed password` lines.

---

## 6.7 Scheduled tasks: cron and at

**Concepts**
- **cron** runs jobs on a schedule. User crontabs are edited with `crontab -e`. System jobs live in `/etc/crontab`, `/etc/cron.d/`, and `/etc/cron.daily/` (and similar).
- **at** runs a job once at a specified time.
- systemd **timers** are an alternative to cron.

**Key commands**
```bash
crontab -l                   # list your cron jobs
sudo crontab -l              # list root's cron jobs
ls -la /etc/cron.*
systemctl list-timers        # systemd timers
```

Cron format: `minute hour day-of-month month day-of-week command`
Example: `*/5 * * * * /home/user/check.sh` runs every 5 minutes.

**Security relevance:** Attackers add cron jobs (or timers) for persistence. Check user crontabs, `/etc/cron.*`, and `systemctl list-timers` during investigations.

---

## 6.8 Resource limits

**Concepts**
- `ulimit` sets per-user limits (open files, processes, core dumps).
- Limits are configured in `/etc/security/limits.conf`.

**Key commands**
```bash
ulimit -a
```

**Security relevance:** Disabling core dumps for setuid programs prevents leaking memory contents.

---

## Checklist
- [ ] I can explain PID, PPID, and process states.
- [ ] I can find, inspect, and stop processes safely using signals.
- [ ] I can read and write a basic systemd unit file.
- [ ] I can check enabled services and timers.
- [ ] I can search `journalctl` and `/var/log` for login and service events.
- [ ] I can list cron jobs for a user and for the system.

## Further reading
- `man systemd.service`
- `man 7 signal`
- DigitalOcean and Red Hat guides on systemd and journalctl
