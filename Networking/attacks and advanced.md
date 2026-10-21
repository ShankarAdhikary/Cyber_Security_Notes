# Level 6: Network Attacks and Advanced Topics

## Overview
This final level combines everything you have learned. You will study how attackers work (reconnaissance, sniffing, spoofing, denial of service, tunnelling), how a professional network penetration test is structured, how to craft packets with Scapy, and how to investigate incidents through network forensics. Each topic is written from both the **attacker's** view (to understand it) and the **defender's** view (to detect and stop it).

**Prerequisites:** Levels 1 to 5.

**Ethics and legal rules (read first)**
- Only test systems **you own** or have **written permission** to test.
- Run all attack labs on an **isolated host-only/internal virtual network**. Never bridge lab VMs to your home or college network while testing.
- Unauthorised access, scanning with intent, interception and disruption are crimes in most countries (in India, the IT Act 2000 covers these).
- Learn the techniques to defend, detect and report, not to harm.

**Lab setup:** Kali Linux (attacker), a Linux target (Ubuntu/Metasploitable 2), a Windows evaluation VM, an optional pfSense/Security Onion VM for detection. All on a host-only network.

## Table of Contents
1. [Reconnaissance and Enumeration](#1-reconnaissance-and-enumeration)
2. [Sniffing and Man-in-the-Middle](#2-sniffing-and-man-in-the-middle)
3. [Spoofing Attacks](#3-spoofing-attacks)
4. [DoS and DDoS](#4-dos-and-ddos)
5. [Port Forwarding and Tunnelling](#5-port-forwarding-and-tunnelling)
6. [Network Penetration Testing Workflow](#6-network-penetration-testing-workflow)
7. [Packet Crafting with Scapy](#7-packet-crafting-with-scapy)
8. [Network Forensics and PCAP Analysis](#8-network-forensics-and-pcap-analysis)
9. [Level 6 Checklist](#level-6-checklist)
10. [Practice Questions](#practice-questions)
11. [What to Learn Next](#what-to-learn-next)

---

## 1. Reconnaissance and Enumeration

### 1. What it is
- **Reconnaissance** = gathering information about a target. **Passive** (no contact: WHOIS, DNS records, certificate transparency, search engines, social media, Shodan) or **Active** (direct contact: scanning).
- **Enumeration** = extracting detailed information from discovered services (users, shares, versions, configurations).

### 2. How it works
**Passive sources:** WHOIS, `dig`, crt.sh, Google dorks, GitHub/code leaks, LinkedIn and job ads (technology stack hints), Wayback Machine, Shodan/Censys, breach data awareness.

**Active steps:** host discovery, port scans, service detection (Level 5).

**Enumeration by service**

| Service | What can be enumerated | Tools |
|---------|-----------------------|-------|
| DNS (53) | Zone transfer, subdomains | `dig`, `dnsrecon`, `amass` |
| SMB (445) | Shares, users, policies, null sessions | `smbclient`, `enum4linux`, `nmap` NSE |
| SNMP (161) | Device info, interfaces, processes | `snmpwalk`, `onesixtyone` |
| LDAP (389) | Users, groups, computers | `ldapsearch` |
| SMTP (25) | Valid users (`VRFY`, `RCPT TO`) | `smtp-user-enum`, `nmap` |
| FTP (21) | Anonymous access, files | `ftp`, `nmap` |
| HTTP/HTTPS | Technologies, directories, headers, robots.txt | `whatweb`, `gobuster`, `nikto`, browser dev tools |
| NFS (2049) | Exported shares | `showmount -e` |

### 3. Key facts
- Good recon decides the success of everything afterwards.
- Enumeration is where "just a version number" turns into a real finding.
- Banners and error messages often reveal versions. Hiding them is helpful but not a fix.

### 4. Security issues and attacks
Information exposed by recon enables targeted attacks: exposed admin panels, old software, leaked credentials, forgotten subdomains, open shares.

### 5. Defenses
Minimise public information, remove stale DNS/subdomains, restrict zone transfers, disable anonymous enumeration (SMB, LDAP, SNMP), hide version banners, monitor for scanning, use honeypots/canary tokens, run your own recon against yourself regularly (attack surface management).

### 6. Hands-on lab (real life)
1. **Passive recon on yourself or a domain you own:** list its subdomains with crt.sh, check WHOIS, DNS records, and what Shodan shows for your public IP.
2. **Google dorks (safe, own domain):** `site:yourdomain.com filetype:pdf`, `site:yourdomain.com intitle:"index of"`.
3. On the Metasploitable lab: run `nmap -sV -sC -p- <IP>`, then enumerate each service with the table above.
4. `enum4linux -a <IP>` and `smbclient -L //<IP> -N` against the lab Windows/Samba target.
5. `showmount -e <IP>` if NFS is open, and `snmpwalk -v2c -c public <IP>` if SNMP is open.
6. Directory discovery against **DVWA** in your lab: `gobuster dir -u http://<IP>/dvwa -w /usr/share/wordlists/dirb/common.txt`.
7. Write an **"Information exposed" list** for your lab target with each item and how you would remove it.
8. Capture your recon in Wireshark/IDS and note how visible each technique is.
9. Practise on TryHackMe: "Passive Reconnaissance", "Active Reconnaissance", "Network Services".

### 7. Quick summary
- Recon (passive then active) then enumeration builds the picture of a target.
- Anonymous or default access on services is the most common easy win.
- Defenders: shrink the exposed surface and watch for scanning.

---

## 2. Sniffing and Man-in-the-Middle

### 1. What it is
- **Sniffing** = capturing network traffic passively.
- **Man-in-the-Middle (MITM)** = an attacker positions themselves between two parties to read and possibly alter their traffic.

### 2. How it works
**Getting in the path**
- **Shared/wireless medium:** anyone in range can capture Wi-Fi frames (encrypted if WPA2/3).
- **ARP spoofing / poisoning:** attacker tells the victim "the gateway's IP is at my MAC" (Level 2).
- **Rogue DHCP / rogue gateway,** **DNS spoofing,** **evil twin Wi-Fi,** **rogue proxy,** **ICMP redirects.**
- **Compromised router or ISP-level access.**
- **Port mirroring/TAP** (legitimate monitoring).

**What the attacker gets**
- Cleartext protocols: credentials, cookies, emails, files (HTTP, FTP, Telnet, POP3).
- Encrypted protocols: only metadata (IPs, domain via SNI, sizes, timing) unless the attacker defeats TLS (**SSL stripping**, **fake certificates** the user accepts, a malicious CA installed on the device).

**Typical flow:** enable IP forwarding, poison ARP both ways, capture traffic, optionally modify it, restore ARP tables to hide.

**Tools (lab use):** Wireshark, tcpdump, Ettercap, `arpspoof`, Bettercap, mitmproxy.

### 3. Key facts
- MITM works because lower layers (ARP, DHCP, DNS) trust by default.
- HTTPS with proper certificate validation plus HSTS makes MITM against web traffic very hard.
- Browser warnings about certificates are the user's last line of defence, and users often click through.

### 4. Security issues and attacks
Credential theft, session hijacking, malware injection into downloads, tampering with transactions, SSL stripping, and surveillance on public Wi-Fi.

### 5. Defenses
Encrypt everything (HTTPS+HSTS, SSH, VPN, TLS email), Dynamic ARP Inspection and DHCP snooping, port security, 802.1X, WPA3 and management frame protection, DNSSEC/DoH/DoT, certificate pinning for apps, user training on certificate warnings, use VPN on untrusted Wi-Fi, monitor for duplicate-IP and gratuitous ARP anomalies.

### 6. Hands-on lab (real life)
**Isolated host-only network with three of your VMs: victim, gateway, attacker.**
1. **Baseline:** on the victim, run `arp -a` and note the gateway MAC.
2. **Attack observation:** from Kali, run an ARP-poisoning tool (`sudo ettercap -T -q -M arp:remote /<victim>// /<gateway>//` or Bettercap) with IP forwarding on (`sudo sysctl -w net.ipv4.ip_forward=1`).
3. On the victim, run `arp -a` again: the gateway now shows the **attacker's MAC**. Screenshot both.
4. From the victim, log in to a **plain HTTP** test page (DVWA over HTTP) and watch the attacker capture the password in Wireshark/Ettercap.
5. Repeat with an **HTTPS** page: the attacker sees encrypted traffic only. Note what **is** still visible (destination IP, SNI).
6. **Detection:** run `arpwatch` or Suricata/Security Onion on the network and see the alert. Check for duplicate MAC addresses in Wireshark (**Expert Information**, filter `arp.duplicate-address-detected`).
7. **Defence:** add a **static ARP entry** on the victim (`sudo arp -s <gateway-IP> <gateway-MAC>`) and show the attack no longer redirects traffic. If you have a managed-switch simulator (Packet Tracer), enable DHCP snooping and Dynamic ARP Inspection.
8. Run `mitmproxy` as a transparent proxy in the lab to see how the browser reacts to an untrusted certificate.
9. Stop and **restore** the network (the poisoning tool should re-ARP the victim), and verify with `arp -a`.

### 7. Quick summary
- MITM = attacker sits in the path (often via ARP, DHCP, DNS or evil twin).
- Cleartext is fully exposed, encrypted traffic leaks only metadata.
- Layer 2 protections plus end-to-end encryption are the answer.

---

## 3. Spoofing Attacks

### 1. What it is
Spoofing = forging identifying information to impersonate another device or user. Networking spoofs include **MAC, ARP, IP, DNS, DHCP, email and caller/SMS sender** forging.

### 2. How it works

| Spoof | What is forged | Purpose | Main defence |
|-------|----------------|---------|--------------|
| **MAC spoofing** | Source MAC | Bypass MAC filtering, impersonate a device | Port security, 802.1X |
| **ARP spoofing** | IP-to-MAC mapping | MITM, DoS | DAI, static ARP, segmentation |
| **IP spoofing** | Source IP | Reflection/amplification, hide origin, bypass IP trust | Ingress/egress filtering (BCP 38), uRPF, never trust IP alone |
| **DNS spoofing / cache poisoning** | DNS answers | Redirect users to fake sites | DNSSEC, randomised ports/IDs, patched resolvers, DoH/DoT |
| **DHCP spoofing** | Rogue DHCP offers | Malicious gateway/DNS | DHCP snooping |
| **Email spoofing** | From address | Phishing, BEC | SPF, DKIM, DMARC enforcement |
| **Web/URL spoofing** | Look-alike domains and pages | Credential theft | User training, browser protections, brand monitoring |
| **Caller ID / SMS spoofing** | Sender identity | Vishing/smishing | Verification procedures |

**Why it works:** many protocols assume the source information is honest. IP spoofing is simplest with **connectionless** protocols (UDP, ICMP) because no handshake is needed. With TCP the attacker usually cannot see replies unless they are also on the path.

### 3. Key facts
- Spoofing is the building block of MITM, amplification DDoS and phishing.
- "Blind" spoofing (cannot see replies) still works for one-way attacks such as floods.
- Attribution is hard when the source is forged, so logs from the network edge and from the targets both matter.

### 4. Security issues and attacks
Impersonation, redirected traffic, bypassed access control, anonymous floods, phishing and fraud.

### 5. Defenses
Authenticate rather than trust addresses (802.1X, certificates, SSH keys, MFA), enforce SPF/DKIM/DMARC, apply anti-spoofing filters at network edges (drop packets with private/bogon source addresses arriving from outside, and block outbound packets whose source is not yours), enable DNSSEC, use switch security features, monitor for anomalies.

### 6. Hands-on lab (real life)
1. **MAC:** on a lab Linux VM change the MAC, `sudo ip link set eth0 down; sudo ip link set eth0 address 02:11:22:33:44:55; sudo ip link set eth0 up`, then `arp -a` from another VM to see the new MAC appear. Then enable **port security** in Packet Tracer and show the port shutting down on a second MAC.
2. **ARP:** see Topic 2 lab.
3. **DNS (local, isolated):** run a small DNS server (dnsmasq) in the lab with a fake record for a **lab-only domain**, point a victim VM's resolver to it and observe how clients obey whatever DNS says. Then explain why DNSSEC, DoH and trusted resolvers matter.
4. **Rogue DHCP (isolated):** start a second dnsmasq DHCP server in the lab and watch clients receive a different gateway. Enable **DHCP snooping** in Packet Tracer and block it.
5. **Email:** use **MXToolbox** to inspect a domain with weak SPF/DMARC versus a strong one. Write down which could be spoofed.
6. **IP spoofing awareness (no attacks):** in Wireshark, find a packet from a lab VM and explain which fields a spoofer would forge. Then look up **BCP 38** and write 5 lines on how ISPs filter spoofed traffic.
7. **Phishing awareness:** compare a real login URL and 5 lookalikes (homograph characters, subdomain tricks). Make a personal checklist for spotting spoofed pages.

### 7. Quick summary
- Spoofing abuses trust in addresses and names.
- Authentication beats address-based trust.
- Edge filtering, switch security, DNSSEC and email authentication are the main controls.

---

## 4. DoS and DDoS

### 1. What it is
- **DoS (Denial of Service):** making a service unavailable by exhausting resources or crashing it, from one source.
- **DDoS (Distributed DoS):** the same, from many sources at once (often a **botnet** of infected devices).

### 2. How it works

| Category | Target resource | Examples |
|----------|----------------|----------|
| **Volumetric** | Bandwidth | UDP floods, DNS/NTP/memcached amplification, ICMP floods |
| **Protocol** | Connection tables, CPU on devices | SYN flood, ACK flood, fragmentation attacks, Ping of Death (historic) |
| **Application layer (L7)** | Web app and database resources | HTTP GET/POST floods, Slowloris (slow headers), expensive search or login requests |

**Amplification:** attacker sends small requests with the victim's spoofed IP to open servers, which send much larger replies to the victim.
**Botnets:** thousands of compromised devices (IoT cameras, routers) controlled by a command server.
**Not always malicious:** a viral post or a sale can create a "flash crowd" that looks like a DDoS.
**Motives:** extortion, hacktivism, competition, distraction during another attack, gaming disputes.

### 3. Key facts
- DDoS mitigation is mostly about **capacity and filtering upstream**, not just a local firewall.
- Layer 7 attacks can be low-volume yet effective.
- Early detection depends on baselines: you must know what normal traffic looks like.

### 4. Security issues and attacks
Downtime, financial loss, reputation damage, smokescreen for intrusions, and collateral damage to shared infrastructure.

### 5. Defenses
- **Architecture:** redundancy, autoscaling, CDN and Anycast distribution.
- **Scrubbing/DDoS protection services** (Cloudflare, Akamai, AWS Shield).
- **Network:** rate limiting, SYN cookies, connection limits, ingress filtering, blackholing/RTBH as a last resort, disable unneeded UDP services and open resolvers.
- **Application:** caching, WAF rules, CAPTCHA/bot management, timeouts for slow connections, optimise expensive queries.
- **Process:** incident response plan, ISP contact list, monitoring and alerting, regular tests.
- **Prevent being a source:** patch/secure your own devices and block spoofed outbound traffic.

### 6. Hands-on lab (real life)
**Defender-focused, tiny scale, own VMs only, isolated network. Do not run floods against any system you do not own, and never against shared or production networks.**
1. **Baseline:** run a simple web server on a lab VM (`python3 -m http.server`), and watch `top`/`htop`, `ss -s` and Wireshark I/O graph during normal use.
2. **Observe the SYN backlog mechanism:** from a second VM, send a **short, low-rate burst** of SYNs to your own VM (for example `sudo hping3 -S -p 80 -c 200 --fast <victim-VM-IP>`). Watch `ss -ant state syn-recv | wc -l` on the victim and the Wireshark I/O graph, then stop.
3. **Mitigation test:** check `cat /proc/sys/net/ipv4/tcp_syncookies`, enable it (`sudo sysctl -w net.ipv4.tcp_syncookies=1`) and repeat to compare behaviour. Add an iptables rate limit: `sudo iptables -A INPUT -p tcp --dport 80 --syn -m limit --limit 20/s --limit-burst 40 -j ACCEPT` followed by a DROP rule for the rest.
4. **Application-layer protection:** put Nginx in front of your test app with `limit_req_zone` rate limiting, and show that a rapid request loop (`for i in $(seq 1 200); do curl -s -o /dev/null http://<IP>/; done`) gets throttled with `503/429` responses.
5. **Detection:** write a Suricata threshold rule that alerts when one source sends more than N SYNs in 10 seconds, and see it fire during step 2.
6. **Analysis exercise:** download a DDoS-related sample pcap from public sample collections, load it in Wireshark, and use **Statistics, Conversations** and **I/O Graph** to identify the attack type and the top sources.
7. **Plan:** write a one-page **DDoS response playbook** for a small web business: who to call (ISP/CDN), what to enable, how to communicate, how to confirm recovery.
8. **Read case studies:** Dyn 2016 (Mirai botnet), GitHub 2018 (memcached amplification). Summarise each in 5 lines: vector, size, mitigation.

### 7. Quick summary
- DoS/DDoS exhausts bandwidth, protocol state or application resources.
- Defence is layered: upstream scrubbing/CDN, rate limits, SYN cookies, WAF and a rehearsed plan.
- Know your normal traffic so you can spot the abnormal.

---

## 5. Port Forwarding and Tunnelling

### 1. What it is
- **Port forwarding** redirects traffic from one address/port to another.
- **Tunnelling** wraps one protocol inside another to pass through a network that would otherwise block or expose it.
Both are everyday admin tools, and also standard techniques for **pivoting** (moving through a compromised host into internal networks) and for hiding or exfiltrating data.

### 2. How it works
**SSH tunnels**
```
ssh -L 8080:10.0.0.5:80 user@jumphost      # Local forward: localhost:8080 -> internal 10.0.0.5:80 via jumphost
ssh -R 9000:localhost:3000 user@server     # Remote forward: server:9000 -> my localhost:3000
ssh -D 1080 user@server                    # Dynamic forward: SOCKS proxy on localhost:1080
ssh -J user@jump user@internal-host        # ProxyJump (jump host) for multi-hop
```
**Other tools:** `socat`, `netcat` relays, `chisel`, `ngrok`/`cloudflared` (expose local services), `sshuttle` (VPN-like over SSH), Windows `netsh interface portproxy`, Metasploit/`proxychains` (pivoting through a compromised lab host).

**Protocol tunnelling and covert channels:** DNS tunnelling, ICMP tunnelling, HTTP/HTTPS tunnelling, tunnelling over allowed ports (443, 53). Used to bypass firewalls and hide exfiltration.

**Router port forwarding (NAT):** public port to an internal host. Convenient, but it exposes an internal service directly to the internet.

### 3. Key facts
- Tunnels inside allowed protocols make **egress filtering and TLS/DNS inspection** valuable.
- Many tunnels are legitimate (remote work, DevOps). Policy and monitoring separate normal from malicious.
- Pivoting is why one compromised machine in a flat network often leads to a full breach.

### 4. Security issues and attacks
Firewall bypass, hidden command-and-control, data exfiltration, lateral movement to internal-only systems, and accidentally exposed internal services through forgotten port forwards or tunnels.

### 5. Defenses
Egress filtering and proxy-only outbound access, restrict SSH forwarding on servers (`AllowTcpForwarding no`, `PermitTunnel no` where not needed), monitor DNS for long/high-entropy queries and unusual volume, watch for long-lived outbound connections and beaconing patterns, segmentation, inventory port forwards on routers, limit admin tools on servers, EDR.

### 6. Hands-on lab (real life)
Use a 3-VM chain: **Client (A) -> Jump (B) -> Internal web server (C)**, where C is only reachable from B.
1. Show that A cannot reach C directly (`curl` times out), using firewall rules on C.
2. **Local forward:** from A, `ssh -L 8080:<C-IP>:80 user@<B-IP>`, then browse `http://localhost:8080` on A and reach C's page through B.
3. **Dynamic forward:** `ssh -D 1080 user@<B-IP>` and `curl --socks5-hostname localhost:1080 http://<C-IP>`. Configure `proxychains` and run `proxychains curl http://<C-IP>`.
4. **ProxyJump:** `ssh -J user@<B-IP> user@<C-IP>` to log in to C via B.
5. **Detection view:** capture on B in Wireshark. Outside the tunnel you only see encrypted SSH. Note what the firewall and logs on B record (`/var/log/auth.log`, `ss -tnp`).
6. **Prevention:** on B set `AllowTcpForwarding no` in `sshd_config`, restart SSH and show that the forwards now fail.
7. **Router exposure check:** list the port forwards on your home router, and justify or remove each one. Check your public IP with a **view-only** Shodan search.
8. **DNS tunnelling awareness (analysis only):** open a sample DNS-tunnelling pcap in Wireshark, filter `dns`, sort by query name length, and spot the long random-looking subdomains. Write the detection rule you would use (for example "queries longer than 50 characters, or more than N unique subdomains per minute").
9. Windows lab: `netsh interface portproxy add v4tov4 listenport=8080 connectaddress=<C-IP> connectport=80` then check with `netsh interface portproxy show all` and remove it.

### 7. Quick summary
- SSH offers local (-L), remote (-R) and dynamic (-D) tunnels.
- Attackers use tunnels to pivot and exfiltrate through allowed ports.
- Egress controls, segmentation, restricted forwarding and DNS/flow monitoring catch them.

---

## 6. Network Penetration Testing Workflow

### 1. What it is
A **network penetration test** is an authorised, structured attempt to find and safely demonstrate weaknesses in a network, then report how to fix them. It differs from a vulnerability scan (automated, lists issues) because it **validates and chains** weaknesses to show real impact.

### 2. How it works
**Common methodology: PTES / NIST SP 800-115, mapped to MITRE ATT&CK**

| Phase | Goal | Example activities |
|-------|------|---------------------|
| 1. Pre-engagement | Scope and legal | Contract, rules of engagement, scope, testing windows, emergency contacts, data handling |
| 2. Reconnaissance | Learn the target | Passive and active recon (Level 5 and Topic 1) |
| 3. Scanning and enumeration | Find services | Nmap, service enumeration |
| 4. Vulnerability analysis | Find weaknesses | Version matching, scanners, manual checks, weak credentials, misconfigurations |
| 5. Exploitation | Prove impact safely | Controlled exploitation of selected findings in scope |
| 6. Post-exploitation | Show business impact | Privilege escalation, credential access, lateral movement (within rules), evidence collection |
| 7. Reporting | Communicate | Executive summary, technical findings, risk ratings, remediation |
| 8. Clean-up and retest | Leave no harm | Remove tools/accounts, restore state, verify fixes |

**Types:** black box (no knowledge), grey box (partial), white box (full). **External** (from the internet) vs **internal** (inside the network). Related: red team (goal-based, stealthy), purple team (red and blue working together).

**Typical internal network test flow:** discover hosts, enumerate SMB/AD/services, check for default/weak credentials and missing patches, capture or relay credentials (LLMNR/NBT-NS/NTLM) in the lab, escalate privileges, move laterally, show access to a "crown jewel", report.

**Rating findings:** CVSS score plus business context (exposure, data, ease of exploitation).

### 3. Key facts
- **Scope and written authorisation are mandatory.** A test outside scope is an attack.
- Safety first: avoid techniques that may crash production, and agree on stop conditions.
- Quality of the **report** and **remediation advice** matters as much as the hacking.
- Document everything while you work (timestamps, commands, screenshots).

### 4. Security issues and attacks
The same techniques are used by real attackers. Understanding the sequence helps defenders place detection at each phase (recon alerts, exploit detection, lateral-movement detection, exfiltration monitoring).

### 5. Defenses
Regular tests and retests, a vulnerability management programme, patch SLAs, segmentation, credential hygiene (unique local admin passwords, LAPS, MFA), disabling legacy protocols (LLMNR, NBT-NS, SMB1), logging and detection mapped to MITRE ATT&CK, and an incident response plan.

### 6. Hands-on lab (real life)
1. Build a safe practice environment: Kali plus **Metasploitable 2**, **Metasploitable 3** or a **VulnHub** machine, plus a Windows evaluation VM, on a host-only network.
2. Write a **mini rules of engagement** document for your lab (scope, allowed actions, timeline).
3. Follow the phase table end to end on one target and keep a **notes template**: Target, Date, Phase, Command, Output summary, Finding, Evidence file.
4. Use **Metasploit** (`msfconsole`) in the lab to check one known vulnerable service (for example the old vsftpd or Samba version on Metasploitable) and record what access it gave. Then fix it (patch/disable) and retest.
5. Try basic credential checks only on your own lab services with a small wordlist (for example `hydra` against your lab SSH/FTP). Then enable Fail2ban or account lockout and show the effect.
6. Map each step you took to a **MITRE ATT&CK** technique ID (for example T1046 Network Service Discovery, T1110 Brute Force) in a table.
7. Write a **full report** (`Pentest-Report-Lab.md`): executive summary, scope, methodology, findings table with CVSS, evidence, remediation, retest results.
8. Practise on legal platforms: TryHackMe paths ("Pre Security", "Jr Penetration Tester", "Network Security" rooms), Hack The Box Academy and starting-point machines, **PortSwigger Web Security Academy** (for later web topics), and **OverTheWire** wargames.
9. Join beginner CTFs (picoCTF, TryHackMe events) to practise under time pressure.
10. Study for certification ideas later: CompTIA Network+ and Security+ (foundations), then eJPT, PNPT, CEH, OSCP depending on your goals.

### 7. Quick summary
- A pentest is authorised, scoped, methodical and ends in a useful report.
- Follow the phases: scope, recon, scan, analyse, exploit, post-exploit, report, retest.
- Map what you do to MITRE ATT&CK so defenders can learn from it.

---

## 7. Packet Crafting with Scapy

### 1. What it is
Scapy is a Python library for **building, sending, sniffing and dissecting packets** at any layer. It teaches you how protocols really look "on the wire", and lets you write custom network tests and learning experiments.

### 2. How it works
Install: `pip install scapy` (or use the Kali version). Run `sudo scapy` for the interactive shell, or write scripts. Packets are built by **stacking layers with `/`**.

```python
from scapy.all import *

# Build layers
pkt = IP(dst="192.168.56.102") / ICMP()
pkt.show()                              # inspect all fields

# Send and receive (layer 3)
reply = sr1(pkt, timeout=2)
reply.show()

# TCP SYN to one port (your lab host)
syn = IP(dst="192.168.56.102") / TCP(dport=80, flags="S")
resp = sr1(syn, timeout=2)
print(resp.summary())                   # SYN-ACK = open, RST = closed

# DNS query
dns = IP(dst="8.8.8.8") / UDP(dport=53) / DNSQR(qname="example.com")
ans = sr1(dns, timeout=2)
print(ans[DNS].an.rdata)

# ARP "who-has" for local hosts (layer 2)
ans, unans = srp(Ether(dst="ff:ff:ff:ff:ff:ff") / ARP(pdst="192.168.56.0/24"), timeout=2)
for sent, recv in ans:
    print(recv.psrc, recv.hwsrc)

# Sniff packets
pkts = sniff(iface="eth0", filter="tcp port 80", count=10)
pkts.summary()
wrpcap("capture.pcap", pkts)            # save / rdpcap("capture.pcap") to read

# Traceroute
res, _ = traceroute("example.com", maxttl=15)
```
**Key functions:** `send` / `sr` / `sr1` (layer 3), `sendp` / `srp` (layer 2), `sniff`, `rdpcap`, `wrpcap`, `.show()`, `.summary()`, `ls(TCP)` (list fields).

**Other uses:** protocol learning, custom scanners, testing firewall and IDS rules, fuzzing, validating your own defences, parsing pcaps in scripts.

### 3. Key facts
- Needs root/admin to send raw packets.
- The OS may interfere (for example the kernel sends a RST for a handshake it did not start). That is normal when crafting TCP.
- Great for **understanding**: building packets yourself makes headers stick in your memory.

### 4. Security issues and attacks
Packet crafting can be used for scanning, spoofing and flooding, which is why you must use it **only against your own lab systems**.

### 5. Defenses
Stateful inspection, ingress/egress filtering, IDS signatures that detect malformed packets and unusual flag combinations, and normalisation at firewalls.

### 6. Hands-on lab (real life)
All targets are your own lab VMs.
1. **Hello Scapy:** build `IP()/ICMP()`, call `.show()`, send it with `sr1`, and compare the reply in Wireshark with what `ping` sends.
2. **Reimplement a mini ping sweep** for your lab subnet using ARP (`srp`) and print live hosts and their MAC addresses.
3. **Mini port checker:** loop over ports 20 to 100 on one lab host with SYN packets, classify open, closed and filtered from the replies, and compare your results to `nmap -sS` on the same ports.
4. **DNS lookup by hand:** craft a DNS query, parse the answer, and compare TTL and fields to `dig`.
5. **Sniffer:** write a script that sniffs HTTP on the lab network and prints the `Host` header and requested path of each request (cleartext lab traffic only).
6. **Firewall test:** craft packets with different flags (SYN, ACK, FIN, no flags) against your `iptables` stateful firewall VM and record which are dropped and which get a response. Explain why in terms of connection tracking.
7. **IDS test:** send a crafted packet that should trigger one of your Suricata rules, verify the alert, then change a field to evade it and discuss why signature rules can be bypassed.
8. **Pcap parser:** use `rdpcap()` to read a sample pcap, count packets per protocol and print the top 5 talkers.
9. **Write-up:** put your scripts in a `scapy-labs/` folder on GitHub with a short README for each (what it does, how to run it, what you learned).

### 7. Quick summary
- Scapy builds packets layer by layer and sends, sniffs and dissects them.
- Perfect for learning protocols and for testing your own firewall/IDS rules.
- Use it only on systems you own or are authorised to test.

---

## 8. Network Forensics and PCAP Analysis

### 1. What it is
Network forensics is capturing, recording and analysing network traffic and logs to **investigate incidents**: what happened, when, who was involved, what data moved, and how the attacker got in.

### 2. How it works
**Evidence sources**
- **Full packet capture (PCAP):** the most complete, but large and mostly encrypted today.
- **Flow data (NetFlow/IPFIX):** who talked to whom, when, how much (no content).
- **Logs:** firewall, proxy, DNS, DHCP, VPN, web server, authentication, IDS/IPS, endpoint (EDR), cloud logs.
- **Zeek logs:** rich summaries (`conn.log`, `dns.log`, `http.log`, `ssl.log`, `files.log`).
- **Memory and disk images** from affected hosts (correlate with network data).

**Investigation steps**
1. **Define the question** (which host is infected? what was stolen?).
2. **Preserve evidence:** copy the pcap/logs, record **SHA-256 hashes**, keep a **chain of custody**, work on copies.
3. **Get an overview:** Protocol Hierarchy, Conversations, Endpoints, top talkers, timeline.
4. **Find the suspicious host/time:** unusual protocols, beaconing, large uploads, odd destinations.
5. **Drill down:** DNS queries, HTTP requests/user-agents, TLS server names/JA3 fingerprints, file transfers, authentication events.
6. **Extract artefacts:** Export Objects (HTTP/SMB), follow streams, carve files, calculate hashes, check hashes and domains against threat intelligence (VirusTotal, AbuseIPDB), always in a safe environment.
7. **Build a timeline** and map to MITRE ATT&CK stages (initial access, execution, C2, lateral movement, exfiltration).
8. **Report** findings with evidence and recommendations.

**Useful Wireshark filters**
```
dns.qry.name contains "suspicious"
http.request.method == "POST"
http.user_agent contains "curl" || http.user_agent contains "python"
tls.handshake.extensions_server_name contains "example"
tcp.analysis.retransmission
frame.len > 1000 && udp
smb2 || ntlmssp
kerberos
ip.addr == 10.0.0.15 && !(dns || arp)
```
**Command line:** `tshark`, `tcpdump -r`, `zeek -r`, `capinfos`, `editcap`, `mergecap`, `NetworkMiner` (extracts files, hosts, credentials), `Zeek`, `Suricata -r`, `ngrep`, `tcpflow`.

**Patterns to look for**
- **Beaconing:** regular, evenly spaced connections to the same external host (C2).
- **DNS anomalies:** very long or random subdomains, many NXDOMAIN, rare TLDs.
- **Large outbound transfers** at odd hours (exfiltration).
- **Internal scanning** (one host contacting many internal IPs/ports).
- **Cleartext credentials** and suspicious file downloads (executables, scripts).
- **Unusual user-agents** or direct-IP HTTP requests.
- **Lateral movement:** SMB/RDP/WinRM between workstations.

### 3. Key facts
- Encrypted traffic still yields metadata (IPs, SNI, timing, volume, JA3/JA4 fingerprints), which is often enough.
- Time sync across devices is vital for correlating events.
- Handle pcaps like sensitive data: they may include passwords and personal information.
- Never open suspicious files or visit malicious URLs from your normal machine.

### 4. Security issues and attacks
Attackers try to **reduce evidence**: encryption, tunnelling, clearing logs, timestomping, living-off-the-land tools. Short log retention and missing time sync hurt investigations badly.

### 5. Defenses (for investigators and organisations)
Centralised logging with long retention, synchronised time, DNS/proxy/firewall logging, full-packet capture at key chokepoints if affordable, flow collection everywhere, an incident response plan, regular tabletop exercises, and a clear evidence-handling procedure.

### 6. Hands-on lab (real life)
1. **Practice pcaps with answers:** download exercises from **malware-traffic-analysis.net** (each has a published write-up). Work out: the victim's IP, MAC and hostname, the malicious domain/IP, what was downloaded, and the timeline. Then compare to the official answers.
2. **Basic hygiene drill:** take any pcap, record its SHA-256 (`sha256sum file.pcap`), run `capinfos file.pcap`, and note start/end time and packet count in your notes.
3. **Overview workflow:** Protocol Hierarchy, Conversations (sort by bytes), Endpoints. Write the top 3 "interesting" things you find.
4. **Extract files:** **File, Export Objects, HTTP**, hash each file and look the hashes up on VirusTotal (hash only, do not upload private files).
5. **Zeek practice:** `zeek -r sample.pcap` then explore: `cat dns.log | zeek-cut query | sort | uniq -c | sort -nr | head`, and look at `conn.log` for long or large connections.
6. **Detect beaconing:** from `conn.log` group connections by destination and look at the time gaps. Regular intervals suggest C2.
7. **Create your own incident:** in your lab, run a few actions (a download over HTTP, a port scan, an SSH login, a DNS lookup of a made-up domain) while capturing with tcpdump. Then analyse the capture **as if you did not know** what happened and reconstruct the timeline.
8. **Use Security Onion** (free) in a VM to ingest a pcap and explore alerts, Zeek logs and the dashboards.
9. **TryHackMe / CyberDefenders:** "Network Traffic Analysis" rooms, "Carnage", "Warzone", "Traffic Analysis" challenges, and Blue Team Labs Online incident investigations.
10. **Write an incident report** in markdown: summary, timeline table, indicators of compromise (IPs, domains, hashes), affected hosts, root cause, recommendations.

### 7. Quick summary
- Forensics = preserve, overview, drill down, extract, timeline, report.
- Look for beaconing, DNS anomalies, large uploads and internal scanning.
- Metadata and logs matter more every year as traffic becomes encrypted.

---

## Level 6 Checklist
- [ ] Do passive and active recon and enumerate at least 5 different service types in a lab
- [ ] Explain how ARP-based MITM works, show the poisoned ARP table in a lab, and show two defences
- [ ] Describe 6 spoofing types and the control that stops each
- [ ] Classify a DoS attack as volumetric, protocol or application layer and name two mitigations for each
- [ ] Build and explain local, remote and dynamic SSH tunnels, and show how to block them
- [ ] Describe all phases of a pentest and map actions to MITRE ATT&CK
- [ ] Craft ICMP, TCP SYN, DNS and ARP packets with Scapy and read the replies
- [ ] Investigate a pcap: identify the victim, the malicious host and a timeline
- [ ] Write a professional-style report (pentest or incident) in markdown
- [ ] State clearly the legal and ethical rules for testing

## Practice Questions
1. Why does ARP spoofing work, and why does HTTPS limit the damage?
2. What makes UDP-based amplification attacks possible and how can ISPs reduce them?
3. How does `ssh -D` help a pentester (and a malicious actor) and how do you detect it?
4. What is the difference between a vulnerability scan and a penetration test?
5. In a pcap, how would you recognise C2 beaconing?
6. Why is hashing a pcap file the first step of a forensic investigation?
7. In Scapy, how do you tell whether a TCP port is open from the reply?
8. Which three log sources would you check first for suspected data exfiltration over DNS?
9. Explain why egress filtering is one of the strongest controls against reverse shells and tunnels.

**Answers:** (1) ARP has no authentication so hosts accept unsolicited replies. HTTPS encrypts and authenticates content so the attacker sees little beyond metadata. (2) UDP lacks a handshake so the source IP can be forged and small requests yield large replies. ISPs use ingress filtering (BCP 38) and operators close open resolvers/servers. (3) It creates a SOCKS proxy through a host to reach other networks. Detect via SSH logs, long-lived sessions and unusual internal connections from the SSH host, and restrict forwarding in `sshd_config`. (4) A scan lists potential issues automatically, a pentest validates, chains and exploits them within scope to show impact. (5) Regular, evenly spaced connections to the same external destination, often with similar sizes. (6) It proves the evidence was not altered and supports chain of custody. (7) A SYN-ACK reply (flags `SA`) means open, an RST means closed, no reply suggests filtered. (8) DNS logs, firewall/proxy logs and flow data (also resolver query statistics). (9) The malware needs to call out, and if outbound traffic is restricted to approved paths it cannot reach the attacker.

## What to Learn Next
You have now covered the full networking base for cyber security. Natural next steps from your teacher's roadmap:
1. **Linux Architecture** (file system, permissions, processes, bash scripting), the other topic group in your roadmap image
2. **Core Web Concepts** (URLs, HTML, CSS, JavaScript, DOM, REST APIs)
3. **Common Web Security Flaws** (XSS, SQL injection, CSRF, SSRF, RCE, broken authentication, misconfigurations)
4. **Advanced Web Exploitation** (XXE and beyond)
5. Blue-team path: SIEM, log analysis, incident response. Red-team path: Active Directory attacks, privilege escalation.

## Real-Life Practice Platforms for This Level
- **TryHackMe:** "Jr Penetration Tester" path, "Network Security" and "Network Traffic Analysis" rooms, "Active Directory Basics", "Security Onion" and SOC paths
- **Hack The Box:** Starting Point, Academy (Network Enumeration, Footprinting, Attacking Common Services)
- **OverTheWire (Bandit, Natas):** learn Linux and security thinking
- **VulnHub / Metasploitable / DVWA:** local vulnerable targets
- **PCAP practice:** malware-traffic-analysis.net, Netresec pcap repository, CyberDefenders, Blue Team Labs Online
- **Reading:** Wireshark User Guide, Nmap Network Scanning book (free online), MITRE ATT&CK, PTES and NIST SP 800-115
- **CTFs:** picoCTF and others for regular, legal practice
