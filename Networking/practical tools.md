# Level 5: Practical Tools and Analysis

## Overview
Theory becomes skill when you can see and test the network yourself. This level teaches the core tools used daily by security analysts and penetration testers: Wireshark, Nmap, tcpdump, netcat, DNS/WHOIS utilities, traceroute/mtr, and a structured scanning methodology that ties them together.

**Prerequisites:** Levels 1 to 4.

**Lab setup:** Kali Linux or Parrot OS VM (attacker), plus one or two target VMs on a **host-only/internal network**: Metasploitable 2, a Linux server VM you built, and a Windows evaluation VM. Never scan networks or hosts you do not own or have written permission to test.

## Table of Contents
1. [Wireshark](#1-wireshark)
2. [Nmap](#2-nmap)
3. [tcpdump](#3-tcpdump)
4. [Netcat](#4-netcat)
5. [dig, nslookup and whois](#5-dig-nslookup-and-whois)
6. [traceroute and mtr](#6-traceroute-and-mtr)
7. [Network Scanning Methodology](#7-network-scanning-methodology)
8. [Level 5 Checklist](#level-5-checklist)
9. [Practice Questions](#practice-questions)

---

## 1. Wireshark

### 1. What it is
Wireshark is the most popular graphical **packet analyser**. It captures network traffic and decodes hundreds of protocols so you can inspect every field.

### 2. How it works
**Interface layout:** Packet list (top), packet details (middle, layered tree), packet bytes (bottom).

**Capture filters** (BPF syntax, applied **before** capture, limit what is recorded):
```
host 192.168.1.10
port 80
tcp port 443 and host 10.0.0.5
net 192.168.1.0/24
```

**Display filters** (applied **after** capture, hide or show packets):
```
ip.addr == 192.168.1.10
tcp.port == 443
http.request.method == "POST"
dns.qry.name contains "example"
tcp.flags.syn == 1 && tcp.flags.ack == 0
!(arp || dns)
frame contains "password"
```

**Useful features**
- **Follow TCP/UDP/HTTP Stream:** reconstructs a conversation.
- **Statistics, Conversations / Endpoints / Protocol Hierarchy / I/O Graph:** big-picture view.
- **Expert Information:** flags retransmissions, resets and anomalies.
- **Export Objects, HTTP:** extract files transferred in cleartext.
- **Colouring rules:** quick visual cues (black = problem packets, for example).
- **Coloured TLS decryption:** with a key log file (`SSLKEYLOGFILE`) you can decrypt your own browser's TLS traffic.
- **Saving:** `.pcap` / `.pcapng`.

### 3. Key facts
- You can only see traffic that reaches your interface. On a switched network you normally see only your own, broadcast and multicast traffic (see port mirroring in Level 4).
- Capturing needs admin privileges and is legal only on networks you are authorised to monitor.
- Wireshark has a command-line twin, **tshark**.

### 4. Security issues and attacks
Wireshark is a passive tool, but unencrypted protocols (HTTP, FTP, Telnet, SMTP, POP3) expose credentials and content to anyone who can capture, which is why encryption matters.

### 5. Defenses
Use encrypted protocols everywhere, protect capture files (they can contain passwords and personal data), restrict who can run captures on production networks.

### 6. Hands-on lab (real life)
1. Capture your own traffic for 60 seconds while browsing. Open **Statistics, Protocol Hierarchy** and note the percentage of TLS, DNS, TCP, UDP.
2. Open `http://neverssl.com` and filter `http`. Use **Follow HTTP Stream**. Find the `User-Agent` and `Server` headers.
3. In a lab, send a login form to a plain HTTP test page (for example DVWA over HTTP) and find the password with `http.request.method == "POST"`.
4. Detect a port scan: run `nmap -sS <target>` from Kali, then capture and use `tcp.flags.syn==1 && tcp.flags.ack==0` and look at the many different destination ports from one source. Check **Statistics, Conversations**.
5. Download a sample capture from **wiki.wireshark.org/SampleCaptures** and answer: which protocols are present, which hosts talk the most, are there any errors?
6. Practise on **malware-traffic-analysis.net** exercises (answers are published): find the infected host's IP, MAC and hostname.
7. Make **two profiles** ("Analysis" and "Troubleshooting") with extra columns such as `tcp.stream` and `dns.qry.name`.
8. Use `tshark -r file.pcap -Y "dns" -T fields -e dns.qry.name | sort | uniq -c | sort -nr | head` to list the most queried domains.

### 7. Quick summary
- Wireshark captures and decodes traffic. Learn capture vs display filters.
- Follow Stream and Statistics windows answer most questions quickly.
- It teaches you why cleartext protocols are dangerous.

---

## 2. Nmap

### 1. What it is
Nmap ("Network Mapper") is the standard tool for **host discovery, port scanning, service/version detection, OS detection** and scripted checks.

### 2. How it works
**Host discovery**
```
nmap -sn 192.168.56.0/24          # ping sweep, no port scan
nmap -Pn <target>                 # skip discovery, assume host is up
nmap -sn -PR 192.168.56.0/24      # ARP discovery on local network (fast and reliable)
```

**Scan types**

| Option | Name | How it works |
|--------|------|--------------|
| `-sS` | SYN (half-open) scan | Sends SYN, never completes handshake (default as root) |
| `-sT` | TCP connect | Full handshake (no root needed, noisier) |
| `-sU` | UDP scan | Slow, finds DNS, SNMP, DHCP |
| `-sA` | ACK scan | Maps firewall rules (filtered vs unfiltered) |
| `-sF -sX -sN` | FIN, Xmas, NULL | Evasive, works against some stacks |
| `-sV` | Version detection | Banner and probe analysis |
| `-O` | OS detection | TCP/IP fingerprinting |
| `-A` | Aggressive | `-sV -O -sC --traceroute` |

**Port selection:** `-p 22,80,443`, `-p 1-1000`, `-p-` (all 65535), `--top-ports 100`, `-F` (fast, 100 ports).

**Timing and output**
```
-T0 ... -T5          # paranoid to insane (T3 default, T4 common in labs)
-oN out.txt          # normal output
-oX out.xml          # XML
-oA basename         # all formats at once
-v / -vv             # verbosity
--reason             # show why a port is marked open/closed
```

**Nmap Scripting Engine (NSE)**
```
nmap -sC <target>                          # default safe scripts
nmap --script vuln <target>                # vulnerability checks
nmap --script http-title,http-headers -p 80,443 <target>
nmap --script smb-enum-shares -p 445 <target>
nmap --script ssl-enum-ciphers -p 443 <target>
nmap --script ftp-anon -p 21 <target>
```
List scripts at `/usr/share/nmap/scripts/`, help with `nmap --script-help <name>`.

**Port states:** open, closed, filtered, unfiltered, open|filtered, closed|filtered.

### 3. Key facts
- SYN scan: SYN-ACK = open, RST = closed, no reply = filtered.
- Scan results depend on firewalls and rate limiting, so repeat and compare scan types.
- Scans are noisy, and intrusion detection systems see them.

### 4. Security issues and attacks
- Attackers use Nmap for reconnaissance.
- **Unauthorised scanning can be illegal** or violate terms of service.
- Scripts in the `intrusive`, `brute` and `exploit` categories can disrupt systems.

### 5. Defenses
Close or filter unneeded ports, rate-limit and detect scans via IDS, deploy deception (honeypots), log and alert on sweep patterns, regularly scan **yourself** first.

### 6. Hands-on lab (real life)
1. Build a lab: Kali + Metasploitable 2 on a host-only network.
2. Discover hosts: `sudo nmap -sn 192.168.56.0/24`.
3. Scan one target fully: `sudo nmap -sS -sV -O -p- -oA full-scan 192.168.56.101`. Read the output: list every open port, service and version.
4. Run `nmap -sC -sV` and compare to a plain `-sV` run. What extra information did scripts reveal?
5. Run `--script vuln` against Metasploitable (lab only) and match findings to CVEs.
6. Compare `-sS`, `-sT` and `-sF` in Wireshark: filter by the target IP and see how the packets differ.
7. Test the effect of firewalls: enable `ufw` on a target with only port 22 open, rescan, and check `-sA` results.
8. Scan with `-T0` (slow) versus `-T4` and compare duration. Check if your Suricata IDS from Level 4 sees the difference.
9. Convert XML results into an HTML report: `xsltproc scan.xml -o scan.html`.
10. Practise on **scanme.nmap.org**, a host the Nmap project explicitly allows for light scanning (be gentle, no aggressive or repeated scans).

### 7. Quick summary
- Nmap = discover hosts, find open ports, identify services and OS, run scripts.
- Know `-sS`, `-sV`, `-sC`, `-O`, `-p`, `-oA`, `-Pn` by heart.
- Always get permission, and cross-check results with a second method.

---

## 3. tcpdump

### 1. What it is
tcpdump is a lightweight **command-line packet capture** tool, ideal for servers with no GUI and for quick captures that you analyse later in Wireshark.

### 2. How it works
```
sudo tcpdump -D                         # list interfaces
sudo tcpdump -i eth0                    # capture on eth0
sudo tcpdump -i eth0 -nn                # no name/port resolution (faster, clearer)
sudo tcpdump -i eth0 -nn -c 20          # stop after 20 packets
sudo tcpdump -i eth0 host 10.0.0.5
sudo tcpdump -i eth0 port 53
sudo tcpdump -i eth0 'tcp port 80 and host 10.0.0.5'
sudo tcpdump -i eth0 'tcp[tcpflags] & (tcp-syn) != 0'
sudo tcpdump -i eth0 -w capture.pcap    # write to file
sudo tcpdump -r capture.pcap            # read from file
sudo tcpdump -i eth0 -A port 80         # show payload as ASCII
sudo tcpdump -i eth0 -X port 80         # hex + ASCII
sudo tcpdump -i eth0 -s 0 -w full.pcap  # capture full packets
```
Filters use **BPF** syntax and combine with `and`, `or`, `not`. Useful keywords: `host`, `src`, `dst`, `net`, `port`, `portrange`, `tcp`, `udp`, `icmp`, `arp`.

### 3. Key facts
- `-nn` avoids slow DNS lookups and makes output unambiguous.
- Capture on the server with tcpdump and then copy the `.pcap` to your workstation for Wireshark analysis.
- Rotate captures to avoid filling the disk: `-C 100 -W 5` (100 MB files, 5 files).

### 4. Security issues and attacks
Attackers with root on a compromised host can run tcpdump to harvest credentials crossing that host. Another reason for encrypted protocols.

### 5. Defenses
Monitor for unexpected capture tools and promiscuous-mode interfaces, restrict root access, encrypt traffic, protect capture files.

### 6. Hands-on lab (real life)
1. `sudo tcpdump -i any -nn -c 10 icmp` and ping something from another terminal.
2. Capture DNS: `sudo tcpdump -i any -nn port 53`, then run `dig example.com` and read the query and answer.
3. Capture HTTP content: `sudo tcpdump -i any -nn -A port 80` and run `curl http://neverssl.com`.
4. Capture a **full TCP handshake**: `sudo tcpdump -i any -nn 'tcp port 80 and host <IP>'`. Identify `[S]`, `[S.]` and `[.]` flags.
5. Write a capture on a lab server (`-w`), copy it with `scp` to your host and analyse it in Wireshark.
6. Create a filter that shows only SYN packets with no ACK, run a scan with Nmap and watch it appear.
7. Capture only ARP and then run `arp -d` plus a ping to see requests and replies.
8. Rotate capture files with `-C` and `-W` for a continuous 5-file ring buffer.

### 7. Quick summary
- tcpdump = fast CLI sniffer, perfect for servers.
- Learn `-i`, `-nn`, `-w`, `-r`, `-A`, `-c` and BPF filters.
- Pair it with Wireshark: capture remotely, analyse locally.

---

## 4. Netcat

### 1. What it is
Netcat (`nc`) is the "Swiss army knife" for TCP/UDP: it reads and writes data across network connections. Useful for testing, simple transfers and understanding protocols.

### 2. How it works
**Connect (client):**
```
nc <host> <port>                  # open a TCP connection
nc -u <host> <port>               # UDP
nc -v -z <host> 20-100            # simple port scan (-z = no data, -v = verbose)
```
**Listen (server):**
```
nc -l -p 9999                     # listen on TCP 9999 (OpenBSD variant: nc -l 9999)
```
**Talk to services by hand:**
```
nc example.com 80
GET / HTTP/1.0
(blank line)
```
**Banner grabbing:** `nc -v <host> 21` or `22` shows the service banner.

**File transfer:**
```
# receiver
nc -l -p 9999 > received.txt
# sender
nc <receiver-IP> 9999 < file.txt
```
**Simple chat:** one side listens, the other connects, then both type.

**Variants:** traditional netcat, OpenBSD netcat, **ncat** (from Nmap, supports TLS: `ncat --ssl`), **socat** (more powerful).

### 3. Key facts
- Netcat does no encryption in its basic form, so everything is cleartext.
- Option names differ between variants, so check `nc -h`.
- Its flexibility is why attackers use it too (see the next sections on defense).

### 4. Security issues and attacks
Attackers abuse netcat for **reverse and bind shells**, data exfiltration and tunnelling after they get initial access. Netcat on a server that does not need it is a warning sign.

### 5. Defenses
Remove netcat from servers that do not need it, use egress filtering so a reverse shell cannot call out, monitor for unexpected listening ports and outbound connections from servers, application allowlisting and EDR.

### 6. Hands-on lab (real life)
1. **Chat:** on VM1 `nc -l -p 9999`, on VM2 `nc <VM1-IP> 9999`, type both ways, and capture in Wireshark (`tcp.port==9999`, **Follow TCP Stream**) to see it is plain text.
2. **Transfer a file:** send a text file between VMs as shown above and verify with `sha256sum` on both sides.
3. **Banner grab:** `nc -v <Metasploitable-IP> 21` then `22`, and note the software versions.
4. **Manual HTTP:** use `nc` to send `GET / HTTP/1.1` plus `Host:` header to your lab web server.
5. **Port check:** `nc -zv <IP> 20-100` and compare to the Nmap result for the same ports.
6. **UDP test:** `nc -u -l 5000` and send messages from another VM.
7. **Defender exercise (lab only):** in your isolated lab, demonstrate how a reverse shell connection looks when it is created (your lab listener plus your lab VM connecting back). Then block it with an **egress rule** on the firewall and show it fails. The point is to see both the traffic pattern and the control that stops it.
8. Try `ncat --ssl -l 9999` and `ncat --ssl <IP> 9999` and see the difference in Wireshark.

### 7. Quick summary
- Netcat = read/write on any TCP/UDP port, for testing and transfers.
- Great for learning protocols and checking connectivity.
- Treat unexpected netcat on a server, and unexpected outbound connections, as red flags.

---

## 5. dig, nslookup and whois

### 1. What it is
Command-line tools for querying **DNS** (`dig`, `nslookup`, `host`) and **domain/IP registration data** (`whois`). Core to reconnaissance and to investigating suspicious domains.

### 2. How it works
**dig**
```
dig example.com                     # A record
dig example.com ANY                 # many servers limit ANY responses
dig example.com MX
dig example.com NS
dig example.com TXT
dig -x 8.8.8.8                      # reverse lookup
dig @8.8.8.8 example.com            # query a specific server
dig +short example.com
dig +trace example.com              # follow the delegation path from the root
dig example.com AXFR @ns1.example.com   # zone transfer attempt (only on systems you may test)
dig +noall +answer example.com
```
**nslookup** (cross-platform)
```
nslookup example.com
nslookup -type=MX example.com
nslookup example.com 8.8.8.8
```
**whois**
```
whois example.com                   # registrar, creation/expiry dates, name servers
whois 8.8.8.8                       # IP block owner, ASN, abuse contact
```
**Related tools:** `host`, `dnsrecon`, `dnsenum`, `fierce` (subdomain/zone discovery), `amass`, online services like crt.sh (certificate transparency), SecurityTrails, ViewDNS, VirusTotal.

**What to look for**
- **Domain age:** newly registered domains are typical for phishing.
- **Name servers and registrar** (grouping suspicious domains).
- **SPF/DMARC** (email security posture).
- **Subdomains:** forgotten dev, test and admin hosts.
- **Dangling CNAMEs** (possible subdomain takeover).

### 3. Key facts
- Many registrars now redact personal WHOIS data (privacy services/GDPR).
- DNS queries to the target's own authoritative servers are visible in their logs. Third-party sources like crt.sh are passive.
- Tools like `dig` show exactly what a resolver sees, unlike a browser which caches.

### 4. Security issues and attacks
Attackers use DNS and WHOIS to map targets, find forgotten subdomains and craft phishing domains. Misconfigured DNS (open zone transfer, dangling records) leaks internal structure.

### 5. Defenses
Restrict zone transfers, remove stale DNS records, monitor certificate transparency for your domain, use registrar lock and MFA, enable DNSSEC, apply WHOIS privacy where appropriate, monitor for look-alike domains.

### 6. Hands-on lab (real life)
1. Pick a domain you own (or a well-known one) and collect: A, AAAA, MX, NS, TXT, SOA. Put them in a table in your notes.
2. Run `dig +trace` and note each server in the chain.
3. Find the SPF and DMARC records and write down whether the policy is strict.
4. Use `whois` on the domain and on its IP. Record the registrar, creation date, name servers and the network owner.
5. Look up **crt.sh** (`https://crt.sh/?q=%25.example.com`) to see all subdomains that have had certificates issued. This is passive.
6. Try a zone transfer against the intentionally vulnerable training domain `zonetransfer.me`: `dig axfr zonetransfer.me @nsztm1.digi.ninja`.
7. Check a suspicious-looking domain (from a phishing report site, view only) for its age and name servers. Compare to a legitimate domain.
8. Run `dnsrecon -d <your-domain> -t std` against **your own** domain.
9. Create a "domain investigation template" with the fields above and reuse it.

### 7. Quick summary
- `dig` and `whois` reveal how a domain is built and who owns it.
- Passive sources (crt.sh) are quiet, direct queries are logged by the target.
- Domain age, name servers and records quickly expose phishing infrastructure.

---

## 6. traceroute and mtr

### 1. What it is
Tools that show the **path** packets take to a destination and the **delay** at each hop. They help troubleshooting, and reveal network structure and filtering during recon.

### 2. How it works
**traceroute** sends probes with increasing TTL (1, 2, 3...). Each router that drops the packet at TTL 0 returns an ICMP "Time Exceeded", revealing its address.
```
traceroute example.com               # Linux: UDP probes by default
traceroute -I example.com            # ICMP probes
traceroute -T -p 443 example.com     # TCP probes to port 443 (passes more firewalls)
traceroute -n example.com            # no DNS lookups
tracert example.com                  # Windows (ICMP)
tracert -d example.com               # Windows, no name resolution
```
**mtr** combines ping and traceroute in a live updating display:
```
mtr example.com
mtr -rwc 50 example.com              # report mode, 50 cycles, wide
mtr -T -P 443 example.com            # TCP mode
```
**Reading the output**
- Columns: hop number, host, loss %, packets sent, last/avg/best/worst latency, standard deviation.
- `* * *` means no reply (firewall or ICMP rate limit). It does **not** always mean a problem.
- Latency jumping at one hop and staying high afterwards = real delay from that point on. A spike at only one hop that vanishes at later hops is usually just that router deprioritising ICMP replies.
- Packet loss that continues to the final hop = real loss.

### 3. Key facts
- Paths can change and can differ in each direction (asymmetric routing).
- Load balancing means different probes may take different paths.
- Tracing to a closed UDP port or blocked ICMP may stop at a firewall, so try TCP mode.

### 4. Security issues and attacks
Traceroute helps attackers map a target's network edge, find firewalls and identify ISP/hosting providers. It also reveals internal addressing if misconfigured routers respond from private IPs.

### 5. Defenses
Limit ICMP responses at the edge, avoid exposing internal topology in replies, rate-limit ICMP, monitor for scanning that uses TTL tricks.

### 6. Hands-on lab (real life)
1. `traceroute -n 8.8.8.8` and note the number of hops, your default gateway (hop 1) and ISP routers.
2. Compare `traceroute -n`, `traceroute -I -n` and `traceroute -T -p 443 -n` to the same site, and see which gets furthest.
3. Trace to a server in another country and plot approximate locations of each hop (look up IPs with `whois` or an IP-geolocation site).
4. Run `mtr -rwc 100 google.com` and identify whether any hop shows real packet loss.
5. Capture traceroute in Wireshark and see the TTL values increase and the ICMP Time Exceeded replies.
6. Trace to a host inside your lab through your firewall VM. Block ICMP and see how the output changes, then try TCP mode.
7. During a slow-internet moment, run `mtr` to your router, then your ISP gateway, then a public DNS server. Work out where the delay starts.

### 7. Quick summary
- traceroute uses TTL tricks to list hops, mtr gives live path quality.
- `* * *` is not always a fault. Use different probe types.
- Useful for troubleshooting and for mapping a target's edge.

---

## 7. Network Scanning Methodology

### 1. What it is
A repeatable, documented process for discovering what is on a network and what weaknesses it has. It turns the individual tools above into a workflow, and is the heart of both vulnerability assessment and the reconnaissance phase of a penetration test.

### 2. How it works
**Step 0: Scope and authorisation (always first)**
Get written permission, define IP ranges/domains in scope, testing windows, excluded systems, contacts, and rules of engagement. Without this, it is not testing, it is an attack.

**Step 1: Passive reconnaissance (no direct contact)**
WHOIS, DNS records, certificate transparency (crt.sh), search engines, Shodan/Censys, public code repositories, job postings, social media.

**Step 2: Host discovery (what is alive)**
```
sudo nmap -sn -PE -PS443 -PA80 10.0.0.0/24 -oA 01-discovery
```
Use ARP discovery on local networks. Save the live host list: `grep "Up" 01-discovery.gnmap | cut -d' ' -f2 > live.txt`.

**Step 3: Port scanning (what is open)**
```
sudo nmap -sS -p- --min-rate 1000 -iL live.txt -oA 02-allports
```
Then UDP on important ports: `sudo nmap -sU --top-ports 50 -iL live.txt -oA 03-udp`.

**Step 4: Service and version detection**
```
sudo nmap -sV -sC -O -p <open-ports> <host> -oA 04-services
```

**Step 5: Enumeration (dig deeper into each service)**
- Web: browse, `whatweb`, directory discovery, check headers, `nikto` (lab/authorised only).
- SMB: share listing, null sessions. DNS: zone transfer. SNMP: community strings. FTP: anonymous. SMTP: user enumeration. LDAP: anonymous bind.

**Step 6: Vulnerability analysis**
Match versions and configurations to known vulnerabilities (CVE databases, NSE `vuln`, OpenVAS/Nessus). Verify manually and **rule out false positives**.

**Step 7: Document and report**
For each finding: what, where, evidence (screenshot/output), impact, severity, remediation. Keep raw output files.

**Step 8: Clean up and retest** after fixes.

**Workflow summary**
```
Scope -> Passive recon -> Host discovery -> Port scan -> Service detection
      -> Enumeration -> Vulnerability analysis -> Report -> Retest
```

### 3. Key facts
- Go from **broad and quiet to narrow and detailed**.
- Save every output (`-oA`) with a clear naming convention.
- Scan impact: aggressive scans can crash fragile systems (printers, old devices). Use appropriate timing.
- **Frameworks to know:** PTES (Penetration Testing Execution Standard), OWASP Testing Guide, NIST SP 800-115, MITRE ATT&CK for how attackers act.

### 4. Security issues and attacks
Attackers follow the same method. Defenders benefit from understanding it: every step leaves traces (firewall logs, IDS alerts, DNS queries).

### 5. Defenses
Run your own regular scans, keep an up-to-date asset inventory, close or filter unneeded ports, detect sweeps in logs and the IDS, use deception (honeypots/canaries), and rate-limit or blacklist scanners where appropriate.

### 6. Hands-on lab (real life)
1. **Lab network:** Kali, Metasploitable 2, a Windows VM and your own hardened Linux VM, all on a host-only network.
2. Write a one-page **scope statement** for your lab (IP range, allowed actions, dates).
3. Follow the 8 steps above end to end. Save output in numbered files in a project folder (`01-discovery`, `02-allports` and so on).
4. For each discovered service write 2 to 3 lines: version, risk, and a recommended fix.
5. Build a **findings table**: ID, host, port/service, description, evidence file, severity (use CVSS if you can), recommendation.
6. Write a **short report**: executive summary (5 lines for a manager), methodology, findings, remediation priorities. Publish it as `Lab-Scan-Report.md` in your GitHub.
7. Fix the top 3 issues on your own VM, then rescan and document the improvement. This "before and after" is excellent portfolio material.
8. Compare what your IDS from Level 4 saw during each scanning step. Which steps were noisiest?
9. Repeat on a **TryHackMe or Hack The Box** machine, using the same methodology and notes template each time.
10. Learn to use **Shodan** (view-only) to search for your **own** public IP or domain and see what the internet sees about you.

### 7. Quick summary
- Scan in order: scope, passive, discovery, ports, services, enumeration, vulnerabilities, report.
- Authorisation first, documentation always.
- Doing the same process on every target makes you fast, thorough and professional.

---

## Level 5 Checklist
- [ ] Capture traffic in Wireshark and use both capture and display filters
- [ ] Follow a stream and extract credentials from a cleartext protocol (in a lab)
- [ ] Run discovery, SYN, UDP, version and script scans with Nmap and explain each result
- [ ] Explain open, closed and filtered ports using packets
- [ ] Capture on a server with tcpdump and analyse the file in Wireshark
- [ ] Use netcat for chat, file transfer and banner grabbing
- [ ] Gather DNS and WHOIS data for a domain, including subdomains via crt.sh
- [ ] Read traceroute and mtr output correctly
- [ ] Complete the full scanning methodology on a lab network and write a report
- [ ] Explain why authorisation and scope come first

## Practice Questions
1. What is the difference between a Wireshark capture filter and a display filter?
2. What does an Nmap SYN scan send and how does it decide a port is open?
3. Why might `nmap` show a port as `filtered`?
4. Which `tcpdump` command saves 5 files of 100 MB each in rotation?
5. Why is netcat on a production server a concern?
6. Which source lets you find subdomains without touching the target?
7. When traceroute shows `* * *` at one hop but later hops respond, is there a fault?
8. Why do we run host discovery before port scanning?
9. What goes into a scan finding in a professional report?

**Answers:** (1) Capture filters limit what is recorded (BPF), display filters only hide packets after capture. (2) A SYN. A SYN-ACK reply means open, RST means closed, nothing means filtered. (3) A firewall drops the probe, or no reply arrives. (4) `tcpdump -i eth0 -w cap.pcap -C 100 -W 5`. (5) It can be used for reverse shells and data transfer, so it is suspicious unless it is needed. (6) Certificate transparency logs (crt.sh) and other passive sources. (7) Not necessarily: that router may simply not answer ICMP/TTL-expired probes. (8) To avoid wasting time scanning dead addresses and to reduce noise. (9) Description, affected host and port, evidence, impact, severity, remediation recommendation.

## Real-Life Practice Platforms for This Level
- **TryHackMe:** "Nmap", "Nmap Live Host Discovery", "Wireshark: The Basics", "Wireshark: Packet Operations", "Wireshark: Traffic Analysis", "tcpdump: The Basics", "Passive Reconnaissance", "Active Reconnaissance", "Network Services"
- **Hack The Box Academy:** "Network Enumeration with Nmap", "Introduction to Network Traffic Analysis"
- **Packet captures:** malware-traffic-analysis.net, wiki.wireshark.org/SampleCaptures, Netresec's pcap repository
- **scanme.nmap.org:** the one public host you may scan lightly
- **Metasploitable 2 / DVWA / VulnHub VMs:** safe local targets
- **Official docs:** nmap.org/book (free online), Wireshark User Guide
