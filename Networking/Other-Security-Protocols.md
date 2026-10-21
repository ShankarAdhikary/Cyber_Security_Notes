# Level 3: Other Security-Relevant Protocols

## Overview
Level 2 covered the protocols everything depends on. This level covers the protocols you meet in real company networks and in almost every penetration test or incident: file transfer, email, Windows networking (SMB, RDP, LDAP, Kerberos), monitoring protocols, VPNs and wireless.

**Prerequisites:** Levels 1 and 2.

**Lab setup:** Linux VM (Kali or Ubuntu), optionally a Windows VM (evaluation ISOs are free from Microsoft) for SMB, RDP and Active Directory topics. Keep all labs on a host-only or internal network.

## Table of Contents
1. [FTP, SFTP and FTPS](#1-ftp-sftp-and-ftps)
2. [Telnet](#2-telnet)
3. [SMTP, POP3 and IMAP](#3-smtp-pop3-and-imap)
4. [SMB](#4-smb)
5. [RDP](#5-rdp)
6. [LDAP and Kerberos](#6-ldap-and-kerberos)
7. [SNMP, NTP and Syslog](#7-snmp-ntp-and-syslog)
8. [VPN Protocols](#8-vpn-protocols)
9. [Wireless Networking](#9-wireless-networking)
10. [Level 3 Checklist](#level-3-checklist)
11. [Practice Questions](#practice-questions)

---

## 1. FTP, SFTP and FTPS

### 1. What it is
Protocols for transferring files.
- **FTP:** original, **unencrypted** (ports 21 control, 20 data).
- **FTPS:** FTP wrapped in TLS.
- **SFTP:** file transfer over SSH (port 22), not related to FTP despite the name.

### 2. How it works
FTP uses **two channels**: a control connection (commands, login) and a data connection.
- **Active mode:** server connects back to the client's port. This is often blocked by firewalls/NAT.
- **Passive mode:** client opens both connections to the server (the common approach).

Typical commands: `USER`, `PASS`, `LIST`, `RETR` (download), `STOR` (upload).
**Anonymous FTP** allows login as `anonymous` without a password.

### 3. Key facts
- In plain FTP, usernames, passwords and files all travel in cleartext.
- Response codes: 220 ready, 230 login ok, 530 login failed, 226 transfer complete.

### 4. Security issues and attacks
- **Credential sniffing** on plain FTP.
- **Anonymous access** exposing sensitive files, or writable anonymous directories used to host malware.
- **Brute force** of logins.
- **FTP bounce attack** (old, abusing the PORT command to scan other hosts).
- **Vulnerable server versions** (for example the infamous vsftpd 2.3.4 backdoor in training VMs).

### 5. Defenses
Replace FTP with SFTP or FTPS, disable anonymous login, chroot users to their directory, restrict by IP, patch the server, monitor transfers.

### 6. Hands-on lab (real life)
1. On a Linux VM: `sudo apt install vsftpd` and start it. In `/etc/vsftpd.conf` first allow `anonymous_enable=YES` for the exercise.
2. From another VM connect with `ftp <IP>`, log in as `anonymous`, then `ls`, `get`, `put`.
3. Capture with Wireshark, filter `ftp`, and **find the username and password in plain text**.
4. Fix it: disable anonymous login and enable TLS in vsftpd (`ssl_enable=YES` with a self-signed certificate), then compare the capture.
5. Use SFTP instead: `sftp user@<IP>` and confirm the capture is unreadable.
6. Run `nmap -sV -p 21 --script ftp-anon <your-VM-IP>` to detect anonymous access.

### 7. Quick summary
- FTP = cleartext. Use SFTP or FTPS.
- Watch for anonymous access and old server versions.
- Credentials in FTP can be read directly from a capture.

---

## 2. Telnet

### 1. What it is
Telnet is an old protocol (TCP port 23) for remote command-line access. It sends **everything including passwords in cleartext**. SSH replaced it, but you will still find Telnet on legacy servers, network devices and IoT.

### 2. How it works
A simple text session over TCP with minimal negotiation of options. Because it is just text over TCP, you can also use the Telnet client as a basic tool to connect to any TCP port (for example `telnet example.com 80`) and type raw protocol commands, which helps in understanding other protocols.

### 3. Key facts
- No encryption, no integrity, weak or no authentication.
- Finding port 23 open on a modern network is a red flag.
- IoT botnets such as Mirai spread by guessing default Telnet credentials.

### 4. Security issues and attacks
- Sniffing credentials and sessions.
- Default or weak passwords on devices.
- Session hijacking.
- MITM (nothing is authenticated or encrypted).

### 5. Defenses
Disable Telnet everywhere and use SSH. Where it cannot be removed, isolate the device in its own VLAN and restrict access with ACLs. Change default credentials.

### 6. Hands-on lab (real life)
1. On a Linux VM: `sudo apt install telnetd` (or `inetutils-telnetd`), start it, and `telnet <IP>` from another VM.
2. Capture in Wireshark and use **Follow TCP Stream** to read your username, password and every typed command in plain text.
3. Use Telnet as a manual client: `telnet example.com 80`, then type `GET / HTTP/1.0` and press Enter twice to see a raw HTTP response.
4. Scan your lab for port 23: `nmap -p 23 --open 192.168.56.0/24`.
5. Disable the service and repeat the scan to verify it is closed.

### 7. Quick summary
- Telnet = remote shell in cleartext, replaced by SSH.
- Still exists on legacy and IoT, a common finding in assessments.
- Disable or isolate it.

---

## 3. SMTP, POP3 and IMAP

### 1. What it is
The email protocols.
- **SMTP** (25, 587, 465): sends and relays mail between servers.
- **POP3** (110, 995 secure): downloads mail to the client and usually deletes it from the server.
- **IMAP** (143, 993 secure): keeps mail on the server and syncs across devices.

### 2. How it works
Sender's mail client, then sender's SMTP server, then (using DNS **MX** lookup) recipient's SMTP server, then recipient's mailbox, then fetched via IMAP or POP3.

Basic SMTP dialogue: `HELO` / `EHLO`, `MAIL FROM`, `RCPT TO`, `DATA`, `QUIT`.

**Ports:** 25 = server-to-server, 587 = client submission (STARTTLS), 465 = implicit TLS.

**Email authentication (DNS TXT records)**
- **SPF:** which servers may send for a domain.
- **DKIM:** a cryptographic signature on the message.
- **DMARC:** policy telling receivers what to do when SPF/DKIM fail, and where to send reports.

### 3. Key facts
- The "From" header in SMTP can be forged easily unless SPF/DKIM/DMARC are enforced.
- Email headers (`Received:` lines) show the real path of a message and are central to investigations.

### 4. Security issues and attacks
- **Phishing and spear phishing.**
- **Email spoofing** of the sender address.
- **Open relays** used to send spam.
- **User enumeration** (`VRFY`, `EXPN`, `RCPT TO` responses).
- **Malicious attachments and links.**
- **Credential theft** over unencrypted POP3/IMAP/SMTP, and Business Email Compromise (BEC).

### 5. Defenses
Enforce SPF, DKIM and DMARC (`p=reject`), require TLS, disable `VRFY`, block open relay, mail filtering and sandboxing, user awareness training, MFA on mailboxes.

### 6. Hands-on lab (real life)
1. Check any domain's email security: `dig example.com TXT` (find SPF), `dig _dmarc.example.com TXT` (DMARC), `dig example.com MX`.
2. Read a real email's headers: in Gmail choose **Show original**. Trace the `Received:` lines from bottom to top and see SPF/DKIM/DMARC results.
3. Talk SMTP manually to a **local** test server: install `postfix` (local only) or use a Python debug server (`python3 -m smtpd -c DebuggingServer -n localhost:1025`; on newer Python use `aiosmtpd`), then `telnet localhost 1025` and send a message with the commands above.
4. Analyse a suspicious email safely: look at the headers, hover over links without clicking, and check URLs in a sandbox or on VirusTotal.
5. Use **MXToolbox** (mxtoolbox.com) to check SPF/DMARC of a few domains you know.
6. Run `nmap -p 25,110,143,465,587,993,995 --script smtp-commands <your-lab-mail-server>`.

### 7. Quick summary
- SMTP sends, IMAP and POP3 receive.
- SPF, DKIM and DMARC fight spoofing. Check them with `dig`.
- Email headers tell the real story in investigations.

---

## 4. SMB

### 1. What it is
SMB (Server Message Block, TCP 445) is the Windows protocol for file shares, printers and inter-process communication. Linux uses it via Samba. It is central to Windows networks.

### 2. How it works
A client connects to `\\server\share`, authenticates (NTLM or Kerberos), then reads and writes files. Versions: SMB1 (old, insecure), SMB2, SMB3 (encryption support). Special shares: `C$`, `ADMIN$`, `IPC$`.

**Null session:** older configurations allowed connecting without credentials to list users and shares.

### 3. Key facts
- NTLM authentication uses challenge and response, with hashes that can be captured and cracked or relayed.
- SMB signing helps prevent relay attacks.
- Port 139 is the older NetBIOS-based variant.

### 4. Security issues and attacks
- **EternalBlue (MS17-010):** SMB1 vulnerability behind WannaCry and NotPetya ransomware.
- **Anonymous/null sessions and open shares** leaking data.
- **NTLM relay and hash capture** (for example via LLMNR/NBT-NS poisoning).
- **Brute force and password spraying.**
- **Lateral movement** using admin shares inside networks.

### 5. Defenses
Disable SMB1, block 445 at the perimeter, enable SMB signing and encryption, disable LLMNR and NBT-NS, patch promptly, strong passwords with account lockout, least privilege on shares.

### 6. Hands-on lab (real life)
1. On a Windows VM create a shared folder. From a second machine run `net view \\<IP>` and `net use`.
2. On a Linux VM: `sudo apt install samba smbclient`, configure a share, then from Kali run `smbclient -L //<IP> -N` to list shares anonymously.
3. Use `nmap -p 445 --script smb-os-discovery,smb-enum-shares,smb-protocols <lab-IP>` and read the results.
4. Capture SMB traffic in Wireshark (`smb || smb2`) and find the negotiated version and the authentication exchange.
5. Check SMB1 status on Windows: `Get-SmbServerConfiguration | Select EnableSMB1Protocol`. Disable it if enabled.
6. Practise on purpose-built vulnerable machines: TryHackMe room "Blue" (EternalBlue) or "Network Services" (SMB section).

### 7. Quick summary
- SMB (445) = Windows file sharing and a major attack surface.
- SMB1 and EternalBlue are classic examples of why patching matters.
- Enable signing, disable SMB1 and LLMNR.

---

## 5. RDP

### 1. What it is
RDP (Remote Desktop Protocol, TCP/UDP 3389) gives a graphical remote session to a Windows machine.

### 2. How it works
The client connects to port 3389, negotiates security (TLS, optionally Network Level Authentication, NLA), the user authenticates, then the screen, keyboard, clipboard and drives are redirected.

### 3. Key facts
- NLA requires authentication before a session is created, reducing exposure.
- RDP is one of the top initial access methods for ransomware groups.

### 4. Security issues and attacks
- **Brute force and password spraying** on internet-exposed RDP.
- **Stolen credentials** reused from breaches.
- **BlueKeep (CVE-2019-0708)** and similar vulnerabilities.
- **MITM** when certificates are ignored.
- **Lateral movement** inside networks with captured credentials.

### 5. Defenses
Never expose 3389 directly to the internet, use a VPN or RD Gateway, enable NLA, MFA, account lockout, limit allowed users, keep Windows patched, monitor event IDs 4624/4625 (logons) and 4776.

### 6. Hands-on lab (real life)
1. On a Windows VM enable Remote Desktop (**Settings, System, Remote Desktop**). Connect from another machine with `mstsc`.
2. Capture in Wireshark and filter `tcp.port==3389`. Note that the content is encrypted.
3. Open **Event Viewer, Windows Logs, Security**, and compare logon events 4624 (success) and 4625 (failure) after a failed attempt.
4. Enable NLA and compare behaviour. Configure account lockout policy (`secpol.msc`).
5. Scan: `nmap -p 3389 --script rdp-enum-encryption <lab-IP>`.
6. Read about real RDP exposure: search "Shodan RDP" and see how many hosts are exposed worldwide (view only, do not touch anything).

### 7. Quick summary
- RDP = Windows remote desktop on 3389.
- A favourite target for brute force and ransomware.
- Never expose it directly. Use VPN, NLA, MFA and lockout policies.

---

## 6. LDAP and Kerberos

### 1. What it is
- **LDAP** (Lightweight Directory Access Protocol, 389, 636 for LDAPS) queries and manages directory services such as **Active Directory (AD)**.
- **Kerberos** (port 88) is the ticket-based authentication protocol AD uses.

### 2. How it works
**Active Directory basics:** a **Domain** contains users, computers and groups, managed by **Domain Controllers (DC)**. Group Policy applies settings. LDAP is how tools query "who are the users and groups".

**Kerberos flow (simplified)**
1. User logs in, requests a **TGT** (Ticket Granting Ticket) from the KDC (on the DC).
2. When accessing a service, the user presents the TGT to get a **service ticket**.
3. The user presents the service ticket to the service.
Passwords are never sent over the network; time must be in sync (usually within 5 minutes).

### 3. Key facts
- AD is the heart of most enterprise networks, so compromising it usually means compromising everything.
- LDAP on 389 is unencrypted unless protected by StartTLS or LDAPS.
- DNS is critical for AD to function.

### 4. Security issues and attacks
- **LDAP enumeration** (listing users, groups, computers), including anonymous binds.
- **Kerberoasting:** requesting service tickets for accounts with SPNs and cracking them offline.
- **AS-REP roasting:** attacking accounts without pre-authentication.
- **Pass-the-hash / pass-the-ticket / golden ticket** attacks.
- **Password spraying** against domain accounts.
- **LDAP injection** in applications.

### 5. Defenses
Require LDAP signing and channel binding, disable anonymous LDAP, long random passwords for service accounts (or gMSA), enable Kerberos pre-authentication, tiered admin model, monitor DC logs, protect the `krbtgt` account, least privilege.

### 6. Hands-on lab (real life)
1. Build a small lab: Windows Server evaluation VM promoted to a domain controller, plus a Windows client joined to the domain. (Many step-by-step "AD home lab" guides exist.)
2. Create users, groups and an OU. Apply a Group Policy.
3. Query from Linux: `ldapsearch -x -H ldap://<DC-IP> -b "dc=lab,dc=local"` and see what an unauthenticated query returns.
4. Capture a Kerberos logon in Wireshark with filter `kerberos` and identify AS-REQ, AS-REP, TGS-REQ, TGS-REP.
5. Run `klist` on the Windows client to see your tickets.
6. Practise ethically on TryHackMe rooms: "Active Directory Basics", "Attacktive Directory", "Kerberos".
7. Use **BloodHound** (in your lab only) to visualise AD relationships and understand why attack paths exist.

### 7. Quick summary
- LDAP queries the directory, Kerberos authenticates with tickets.
- AD compromise = network compromise, so it is the top target.
- Learn the AD basics well before attempting any internal pentest content.

---

## 7. SNMP, NTP and Syslog

### 1. What it is
Management and monitoring protocols.
- **SNMP** (UDP 161, traps 162): monitors and configures network devices.
- **NTP** (UDP 123): synchronises clocks.
- **Syslog** (UDP 514 or TCP/TLS 6514): central log collection.

### 2. How it works
- **SNMP:** a manager polls agents on devices (routers, switches, printers). Data is organised in a **MIB** tree of **OIDs**. v1/v2c use **community strings** (like passwords, defaults `public` and `private`) in cleartext. **v3** adds authentication and encryption.
- **NTP:** hierarchical "strata" of time servers. Accurate time is essential for logs, certificates and Kerberos.
- **Syslog:** devices send log messages with a facility and severity (0 Emergency to 7 Debug) to a log server or SIEM.

### 3. Key facts
- SNMP exposes a lot: device info, interfaces, routing tables, ARP tables, sometimes configuration.
- Wrong time ruins forensic timelines and breaks authentication.
- Syslog over UDP is unreliable and unauthenticated.

### 4. Security issues and attacks
- **Default community strings** (`public`, `private`) giving read or even write access.
- **SNMP amplification** and information gathering.
- **NTP amplification** (the old `monlist` command), and **time manipulation** to break Kerberos or hide events.
- **Log tampering**, or attackers clearing local logs, so central logging matters.
- **Spoofed syslog messages** to flood or mislead analysts.

### 5. Defenses
Use SNMPv3, change or disable default communities, restrict SNMP by source IP, use trusted internal NTP sources and authentication, patch NTP servers, forward logs centrally with TLS, protect and back up the log server, monitor log gaps.

### 6. Hands-on lab (real life)
1. Install SNMP on a Linux VM: `sudo apt install snmpd snmp`, then query it: `snmpwalk -v2c -c public localhost`. Observe the large amount of data returned.
2. Change the community string and confirm that the old one no longer works. Then try configuring SNMPv3 (`net-snmp-create-v3-user`).
3. Scan your lab: `nmap -sU -p 161 --script snmp-info <lab-IP>`.
4. Check time sync: `timedatectl status` (Linux) or `w32tm /query /status` (Windows). Query NTP: `ntpdate -q pool.ntp.org` or `chronyc sources`.
5. Run Linux logs: `journalctl -f`, then generate events by `logger "test message"` and view them in `/var/log/syslog`.
6. Forward logs to a second VM with `rsyslog` (`*.* @@<log-server-IP>:514`) and watch them arrive.
7. Set up a lightweight SIEM-like view with **Wazuh**, **Graylog** or the **ELK stack** in your lab to see centralised logs.

### 7. Quick summary
- SNMP monitors but leaks. Default `public` strings are a classic finding.
- NTP keeps time, and time matters for logs and Kerberos.
- Syslog centralises evidence. Forward logs off the device.

---

## 8. VPN Protocols

### 1. What it is
A VPN (Virtual Private Network) creates an encrypted tunnel across an untrusted network, such as the internet, so remote users or sites can connect as if on a private network.

### 2. How it works

| Protocol | Notes |
|----------|-------|
| **IPsec** (IKE, UDP 500/4500) | Standard for site-to-site, with two modes: transport and tunnel. Uses AH/ESP, IKE negotiates keys. |
| **OpenVPN** (UDP/TCP 1194, commonly) | Open source, TLS-based, very flexible. |
| **WireGuard** (UDP, 51820 by default) | Modern, small codebase, fast, uses modern cryptography. |
| **L2TP/IPsec**, **PPTP**, **SSTP** | PPTP is broken and should not be used. |

**Types:** **Remote access** (one user to company) and **site-to-site** (office to office). **Split tunnelling** sends only some traffic via the VPN, **full tunnel** sends all.

**Commercial "privacy VPNs"** hide your IP from websites and encrypt traffic to the VPN provider, but you are trusting the provider instead of your ISP. That is different from corporate VPNs.

### 3. Key facts
- VPN protects traffic in transit between the endpoints only.
- A VPN does not make you anonymous or make malware safe.
- Authentication quality (certificates, MFA) is as important as encryption.

### 4. Security issues and attacks
- **Weak or obsolete protocols and ciphers** (PPTP, IKEv1 aggressive mode with weak pre-shared keys).
- **Stolen VPN credentials** (no MFA), the top route into corporate networks.
- **Unpatched VPN appliances**, repeatedly exploited in real attacks.
- **DNS or IP leaks** that bypass the tunnel.
- **Split tunnelling** creating a bridge between a compromised home PC and the company network.

### 5. Defenses
Use WireGuard, IPsec with strong parameters or modern OpenVPN, MFA, patch VPN gateways quickly, limit VPN user access with segmentation, enable logging, test for leaks, disable legacy protocols.

### 6. Hands-on lab (real life)
1. Install **WireGuard** on two VMs (`sudo apt install wireguard`). Generate keys (`wg genkey | tee private.key | wg pubkey > public.key`), write `wg0.conf` on both sides, and bring it up with `wg-quick up wg0`.
2. Ping across the tunnel IPs and verify with `sudo wg show`.
3. Capture on the physical interface in Wireshark and compare it to the traffic inside the tunnel. Only encrypted UDP is visible outside.
4. Optional: set up **OpenVPN** with the easy-rsa helper scripts and compare the setup.
5. Test leaks: with a VPN on, visit `ipleak.net` and `dnsleaktest.com` to see which IP and DNS servers are exposed.
6. Packet Tracer: configure a **site-to-site IPsec VPN** between two routers.

### 7. Quick summary
- VPN = encrypted tunnel over untrusted networks.
- Prefer WireGuard, IPsec or OpenVPN. Avoid PPTP.
- Weak authentication and unpatched appliances are the real VPN risks.

---

## 9. Wireless Networking

### 1. What it is
Wi-Fi (IEEE 802.11) provides wireless LAN access. Because the medium is air, anyone nearby can receive the signal, which makes security essential.

### 2. How it works
- **Components:** Access Point (AP), clients, **SSID** (network name), BSSID (AP's MAC).
- **Bands:** 2.4 GHz (longer range, crowded), 5 GHz and 6 GHz (faster, shorter range).
- **Standards:** 802.11n (Wi-Fi 4), ac (Wi-Fi 5), ax (Wi-Fi 6/6E), be (Wi-Fi 7).

**Security evolution**

| Protocol | Status |
|----------|--------|
| WEP | Broken, crackable in minutes, never use |
| WPA (TKIP) | Obsolete |
| **WPA2-PSK (AES/CCMP)** | Acceptable if the passphrase is strong |
| **WPA2/3-Enterprise (802.1X)** | Best for organisations: each user has own credentials (RADIUS) |
| **WPA3-SAE** | Current best for home. Protects against offline guessing |

**WPA2 4-way handshake:** derives session keys from the passphrase. An attacker who captures it can try to crack the passphrase **offline**.

### 3. Key facts
- Hiding the SSID is not security.
- MAC filtering is easy to bypass and is not security.
- WPS (PIN setup) has known weaknesses and should be disabled.
- Monitor mode on a compatible adapter lets you observe wireless frames.

### 4. Security issues and attacks
- **Handshake capture and offline cracking** of weak WPA2 passphrases.
- **Deauthentication attacks** to disconnect clients (mitigated by 802.11w).
- **Evil twin / rogue AP:** a fake access point with the same SSID, used for MITM or credential capture (captive portals).
- **WPS PIN attacks.**
- **Unauthorised rogue APs** plugged into the corporate network.
- **Weak passphrases and default router credentials.**

### 5. Defenses
Use WPA3 or WPA2-AES with a long random passphrase (or 802.1X for organisations), disable WPS, change default admin password, keep firmware updated, enable management-frame protection, separate guest and IoT networks, run wireless intrusion detection and rogue AP scans.

### 6. Hands-on lab (real life)
**Only test your own network.**
1. Survey: use your phone's Wi-Fi analyser app or `nmcli dev wifi` (Linux) / `netsh wlan show networks mode=bssid` (Windows) to list nearby networks, their channels and security types.
2. Check your own router: confirm WPA2/WPA3 with AES, WPS disabled, strong passphrase, guest network enabled, firmware current.
3. With a monitor-mode capable adapter and your **own** AP, use `airmon-ng`, `airodump-ng` to observe beacon frames and the four-way handshake (`eapol` filter in Wireshark).
4. Test passphrase strength on **your own** handshake with a small custom wordlist to see why long random passphrases matter. A strong passphrase makes this impractical.
5. Use **Wi-Fi Analyzer / Kismet** to detect channel congestion and unknown APs.
6. Learning platforms: TryHackMe "Wifi Hacking 101", the Wireshark sample captures for WPA, and **WiFi Challenge Lab** style CTFs.

### 7. Quick summary
- Use WPA3, or WPA2-AES with a long random passphrase. Never WEP.
- Evil twins, deauth and handshake cracking are the classic attacks.
- Disable WPS and segment guest/IoT devices.

---

## Level 3 Checklist
- [ ] Explain why FTP and Telnet are insecure and what replaces them
- [ ] Trace an email from sender to recipient and read its headers
- [ ] Explain SPF, DKIM and DMARC and check them with `dig`
- [ ] Explain SMB, EternalBlue and why SMB1 must be disabled
- [ ] Explain why exposed RDP is dangerous and how to secure it
- [ ] Describe the Kerberos ticket flow and name two attacks on it
- [ ] Explain what an AD domain and domain controller are
- [ ] Query SNMP and explain why `public` is a risk
- [ ] Compare IPsec, OpenVPN and WireGuard
- [ ] List WPA2 vs WPA3 differences and 3 wireless attacks
- [ ] Capture and read cleartext FTP and Telnet credentials in Wireshark

## Practice Questions
1. Which two protocols from this level should be disabled on almost all modern networks?
2. A domain has SPF but no DMARC. Why can spoofing still succeed?
3. Why is SMB on port 445 blocked at internet perimeters?
4. How does Kerberoasting work, in one sentence?
5. What makes SNMPv3 better than v2c?
6. Why is accurate time important for security?
7. Is a VPN the same as anonymity?
8. Why is an evil twin hard to spot for users?

**Answers:** (1) Telnet and plain FTP (also SMB1). (2) Without DMARC the receiver has no policy telling it to reject failing mail. (3) Heavy exploitation history, such as EternalBlue and ransomware spreading. (4) Request service tickets for service accounts and crack them offline to recover passwords. (5) Authentication and encryption. (6) Log correlation, certificate validity and Kerberos all depend on it. (7) No: you shift trust to the VPN provider and other identifiers still exist. (8) It copies the real SSID and users' devices auto-connect to the strongest signal.

## Real-Life Practice Platforms for This Level
- **TryHackMe:** "Network Services", "Network Services 2", "Blue", "Active Directory Basics", "Wifi Hacking 101", "Intro to Cryptography"
- **Hack The Box Academy:** "Network Enumeration with Nmap", "Footprinting" (FTP, SMB, SNMP, email)
- **Metasploitable 2** and **Windows evaluation VMs** for isolated labs
- **VulnHub:** downloadable vulnerable machines for offline practice
- **Wazuh / Graylog / ELK:** to practise log collection
