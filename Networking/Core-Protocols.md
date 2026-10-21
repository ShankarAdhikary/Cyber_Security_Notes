# Level 2: Core Protocols

## Overview
These nine protocols carry almost all internet traffic and are involved in most attacks and investigations. For each one you will learn how it works at packet level, how it is abused, and how to defend it. This level includes the 7 protocols your teacher listed (TCP, UDP, HTTP, DNS, SSH, DHCP, ARP) plus IP/ICMP and HTTPS/TLS.

**Prerequisites:** Level 1 (OSI/TCP-IP, IP addressing, ports).

**Lab setup:** Wireshark installed. A Linux VM (Kali, Ubuntu or Parrot) is recommended. Use only your own network and VMs.

## Table of Contents
1. [IP and ICMP](#1-ip-and-icmp)
2. [TCP](#2-tcp)
3. [UDP](#3-udp)
4. [ARP](#4-arp)
5. [DHCP](#5-dhcp)
6. [DNS](#6-dns)
7. [HTTP](#7-http)
8. [HTTPS, TLS and SSL](#8-https-tls-and-ssl)
9. [SSH](#9-ssh)
10. [Level 2 Checklist](#level-2-checklist)
11. [Practice Questions](#practice-questions)

---

## 1. IP and ICMP

### 1. What it is
**IP (Internet Protocol)** is the Layer 3 protocol that addresses and routes packets. **ICMP (Internet Control Message Protocol)** carries error and diagnostic messages (used by `ping` and `traceroute`).

### 2. How it works
**IPv4 header key fields:** version, header length, TTL, protocol (6 = TCP, 17 = UDP, 1 = ICMP), source IP, destination IP, flags and fragment offset.

IP is **connectionless and unreliable**: no guarantee of delivery or order. Reliability comes from TCP above it.

**Fragmentation:** if a packet is bigger than the link's MTU (usually 1500 bytes), it is split into fragments and reassembled at the destination.

**Common ICMP types**

| Type | Meaning |
|------|---------|
| 0 | Echo Reply (ping response) |
| 3 | Destination Unreachable |
| 8 | Echo Request (ping) |
| 11 | Time Exceeded (TTL hit 0, used by traceroute) |

### 3. Key facts
- Protocol number identifies what is inside the IP packet.
- ICMP has no ports.
- Ping uses ICMP echo, `traceroute` increases TTL step by step and reads the Time Exceeded replies.

### 4. Security issues and attacks
- **Ping sweep:** pinging a whole range to find live hosts (reconnaissance).
- **ICMP flood / Smurf attack:** overwhelm a target with echo traffic.
- **Ping of Death (historic):** malformed oversized packets.
- **ICMP tunnelling:** hiding data inside ICMP packets to bypass filters.
- **IP spoofing and fragmentation tricks** to evade simple filters.
- **OS fingerprinting:** TTL and header behaviour hint at the target OS.

### 5. Defenses
Rate-limit ICMP, block unnecessary ICMP types at the edge (but allow "needs fragmentation" messages), ingress/egress filtering against spoofed addresses, monitor unusual ICMP payload sizes.

### 6. Hands-on lab (real life)
1. `ping -c 4 8.8.8.8` (Linux) or `ping 8.8.8.8` (Windows). Note the TTL in replies.
2. In Wireshark, filter `icmp`. Open an Echo Request, find the type (8), the code and the payload.
3. Run `traceroute 8.8.8.8` and in Wireshark watch the ICMP Time Exceeded replies from each hop.
4. Test MTU: `ping -f -l 1472 8.8.8.8` (Windows) or `ping -M do -s 1472 8.8.8.8` (Linux). Increase the size until it fails to find your path MTU.
5. Ping a host on your LAN versus a distant server and compare TTL and latency.

### 7. Quick summary
- IP routes packets, unreliable by design.
- ICMP reports errors and powers ping/traceroute.
- Attackers use ICMP for recon, floods and tunnels, so filter it sensibly.

---

## 2. TCP

### 1. What it is
TCP (Transmission Control Protocol) is a Layer 4 protocol providing **reliable, ordered, connection-oriented** delivery. Used by HTTP(S), SSH, SMTP, FTP and most applications that cannot lose data.

### 2. How it works
**Three-way handshake**
1. Client sends **SYN** (seq = x).
2. Server replies **SYN-ACK** (seq = y, ack = x+1).
3. Client sends **ACK** (ack = y+1). Connection established.

**Closing:** FIN, ACK from each side (four-step), or **RST** to abort immediately.

**TCP flags:** SYN, ACK, FIN, RST, PSH, URG.

**Reliability features**
- Sequence and acknowledgement numbers for ordering and lost-packet detection.
- Retransmission of unacknowledged data.
- Flow control with a **window size**.
- Congestion control.

**State examples:** LISTEN, SYN_SENT, SYN_RECEIVED, ESTABLISHED, FIN_WAIT, TIME_WAIT.

### 3. Key facts
- Header includes source port, destination port, sequence, acknowledgement, flags, window.
- Slower than UDP because of the overhead, but reliable.
- Closed port response to SYN: **RST**. Open port: **SYN-ACK**. Filtered: no response (this is how port scans read results).

### 4. Security issues and attacks
- **SYN flood:** many SYNs never completed, exhausting the server's half-open connection table.
- **Port scanning:** SYN, connect, FIN, Xmas and NULL scans.
- **TCP session hijacking:** guessing sequence numbers to inject into a session.
- **RST injection:** forging reset packets to kill connections.

### 5. Defenses
SYN cookies, connection rate limiting, firewalls with stateful inspection, randomised initial sequence numbers (default on modern OS), IDS rules for scan patterns.

### 6. Hands-on lab (real life)
1. In Wireshark, filter `tcp.flags.syn==1` and visit a website. Find the SYN, SYN-ACK, ACK packets.
2. Right click a packet, **Follow, TCP Stream** to see the conversation.
3. Run `ss -ant` (or `netstat -an`) and identify states like LISTEN and ESTABLISHED.
4. On your own Linux VM, start `python3 -m http.server 8000`, then from another VM run `nmap -sS -p 8000 <VM-IP>`. Capture it and see SYN, SYN-ACK, RST.
5. Check SYN cookie status on Linux: `cat /proc/sys/net/ipv4/tcp_syncookies`.

### 7. Quick summary
- TCP = reliable, connection-oriented, with a 3-way handshake.
- Flags control the connection (SYN, ACK, FIN, RST).
- Scans and SYN floods abuse the handshake. Defend with SYN cookies and stateful firewalls.

---

## 3. UDP

### 1. What it is
UDP (User Datagram Protocol) is a Layer 4 protocol that is **connectionless and unreliable** but fast and lightweight.

### 2. How it works
- No handshake, no ordering, no retransmission, no congestion control.
- Header is only 8 bytes: source port, destination port, length, checksum.
- The application must handle any loss it cares about.

**Used by:** DNS queries, DHCP, NTP, SNMP, VoIP, video streaming, online games, QUIC (HTTP/3).

### 3. Key facts

| Feature | TCP | UDP |
|---------|-----|-----|
| Connection | Yes (handshake) | No |
| Reliable | Yes | No |
| Ordered | Yes | No |
| Speed / overhead | Slower / higher | Faster / lower |
| Use cases | Web, email, SSH | DNS, VoIP, streaming |

- Closed UDP port usually returns an ICMP "port unreachable". Open ports often return nothing, which makes UDP scanning slow.

### 4. Security issues and attacks
- **UDP flood:** overwhelming a host with datagrams.
- **Amplification/reflection DDoS:** attacker spoofs the victim's IP and sends small requests to open DNS, NTP or memcached servers, which send large replies to the victim.
- **Easy spoofing** because there is no handshake.
- **UDP scanning** to find DNS, SNMP and NTP services.

### 5. Defenses
Disable open resolvers and unneeded UDP services, rate limiting, ingress filtering (BCP 38), response rate limiting on DNS, restrict `monlist`-style features on NTP.

### 6. Hands-on lab (real life)
1. In Wireshark, filter `udp` and browse. Find DNS (port 53) queries and responses.
2. Run `nslookup example.com` and capture the single UDP query and response. Compare packet sizes.
3. Between two VMs: `nc -u -l 9999` on one and `nc -u <IP> 9999` on the other. Type messages and watch the datagrams in Wireshark. Notice that there is no handshake.
4. Run `nmap -sU -p 53,123,161 <your-own-router-IP>` and compare how slow it is versus a TCP scan.

### 7. Quick summary
- UDP = fast, connectionless, no guarantees.
- No handshake makes spoofing and amplification attacks possible.
- Used wherever speed matters more than perfect delivery.

---

## 4. ARP

### 1. What it is
ARP (Address Resolution Protocol) maps an IPv4 address to a MAC address on a local network.

### 2. How it works
1. Host A wants to reach `192.168.1.20` on the same subnet but needs its MAC.
2. A **broadcasts** "Who has 192.168.1.20? Tell 192.168.1.10" (ARP Request).
3. The owner replies with a unicast "192.168.1.20 is at AA:BB:CC:DD:EE:FF" (ARP Reply).
4. A stores this in its **ARP cache** for a short time.

For traffic leaving the subnet, the host ARPs for the **default gateway's MAC**, not the final destination.

**Gratuitous ARP:** a host announces its own IP-MAC mapping (used on startup or failover).

### 3. Key facts
- ARP works only inside one broadcast domain.
- **ARP has no authentication.** Any host can send a reply, and most systems accept unsolicited replies.
- IPv6 replaces ARP with Neighbor Discovery Protocol (NDP).

### 4. Security issues and attacks
- **ARP spoofing / poisoning:** attacker sends fake replies, so the victim maps the gateway's IP to the attacker's MAC.
- **Man-in-the-Middle (MITM):** attacker forwards the traffic while reading or altering it.
- **Denial of service:** mapping the gateway to a non-existent MAC.

### 5. Defenses
Dynamic ARP Inspection (DAI) with DHCP snooping on managed switches, static ARP entries for critical hosts, network segmentation, encrypted protocols (HTTPS, SSH) so that a MITM sees little, ARP monitoring tools such as `arpwatch`.

### 6. Hands-on lab (real life)
1. Run `arp -a` and match entries to devices on your home network.
2. In Wireshark, filter `arp`. Clear your cache (`sudo ip -s -s neigh flush all` on Linux, `arp -d *` on Windows as admin), then ping a neighbour and capture the request/reply.
3. **Isolated lab only:** in a VirtualBox **host-only** network with three of your own VMs (victim, gateway, attacker), observe ARP spoofing using a tool such as `arpspoof` or Ettercap from the attacker VM, then check the victim's `arp -a`. You will see the gateway's MAC replaced. Never do this on a network you do not own.
4. Install `arpwatch` on a Linux VM and watch it flag the change.

### 7. Quick summary
- ARP resolves IP to MAC with a broadcast request.
- No authentication means ARP can be poisoned.
- Defend with DAI, segmentation and encryption.

---

## 5. DHCP

### 1. What it is
DHCP (Dynamic Host Configuration Protocol) automatically gives devices an IP address, subnet mask, default gateway and DNS server.

### 2. How it works
The **DORA** process (UDP, server port 67, client port 68):
1. **Discover:** client broadcasts "Is there a DHCP server?"
2. **Offer:** server offers an IP.
3. **Request:** client requests that IP.
4. **Acknowledge:** server confirms with a **lease time**.

Clients **renew** at 50% of the lease time. The server keeps a pool of addresses and can make **reservations** (a fixed IP tied to a MAC).

### 3. Key facts
- Initial messages are broadcast because the client has no IP yet.
- DHCP options deliver gateway (option 3), DNS (option 6), domain name, and more.
- No built-in authentication.

### 4. Security issues and attacks
- **Rogue DHCP server:** attacker answers first, giving victims a malicious gateway or DNS, so enabling MITM.
- **DHCP starvation:** attacker requests all addresses with fake MACs, so legitimate clients get nothing (often a step before a rogue server).
- **Information leakage** from hostnames sent in requests.

### 5. Defenses
**DHCP snooping** on switches (only trusted ports may send Offers), port security to limit MACs per port, monitoring for unexpected DHCP servers, short-lived scopes with logging.

### 6. Hands-on lab (real life)
1. In Wireshark filter `bootp` (or `dhcp`). Renew your IP (`ipconfig /renew` or `sudo dhclient -r && sudo dhclient`) and capture all four DORA packets.
2. Inspect the Offer packet and find the offered IP, gateway, DNS, and lease time options.
3. See your lease details: `ipconfig /all` (Windows) or `nmcli device show` (Linux).
4. Log in to your router and view the DHCP client list, the pool range, and lease time. Create one DHCP reservation for your own device.
5. In Packet Tracer, configure a DHCP server and watch PCs obtain addresses.

### 7. Quick summary
- DHCP = automatic IP configuration via DORA.
- No authentication, so rogue servers are a real threat.
- DHCP snooping is the main defence.

---

## 6. DNS

### 1. What it is
DNS (Domain Name System) translates human-readable names (`example.com`) into IP addresses. It is the internet's phone book.

### 2. How it works
**Resolution path:** client, then **recursive resolver** (ISP or 8.8.8.8), then **root server**, then **TLD server** (.com), then **authoritative server** for the domain. Answers are **cached** for the record's TTL.

**Common record types**

| Record | Purpose |
|--------|---------|
| A | Name to IPv4 |
| AAAA | Name to IPv6 |
| CNAME | Alias to another name |
| MX | Mail servers |
| NS | Authoritative name servers |
| TXT | Free text (SPF, DKIM, verification) |
| PTR | IP to name (reverse lookup) |
| SOA | Zone authority info |

Uses UDP port 53 for most queries, TCP port 53 for large responses and zone transfers. Newer secure variants: DNS over HTTPS (DoH) and DNS over TLS (DoT).

### 3. Key facts
- Plain DNS is **unencrypted and unauthenticated** by default.
- DNSSEC adds signatures to prove answers are authentic (not privacy).
- DNS logs are a goldmine for defenders because every connection often starts with a lookup.

### 4. Security issues and attacks
- **DNS spoofing / cache poisoning:** fake answers inserted into a resolver's cache.
- **DNS hijacking:** changing resolver settings or registrar records.
- **DNS tunnelling / exfiltration:** hiding data in long subdomain queries.
- **Zone transfer leaks (AXFR):** misconfigured servers hand out their full record list.
- **Subdomain enumeration** for recon.
- **DNS amplification DDoS.**
- **Phishing with look-alike domains** (typosquatting).

### 5. Defenses
DNSSEC, DoH/DoT, restrict zone transfers to known servers, close open resolvers, DNS filtering and logging, monitor for high-volume or high-entropy queries, registrar lock and MFA on domain accounts.

### 6. Hands-on lab (real life)
1. `nslookup example.com` and `dig example.com A` (Linux). Then try `dig example.com MX`, `NS`, `TXT`.
2. Follow the whole chain: `dig +trace example.com` and identify root, TLD and authoritative answers.
3. Reverse lookup: `dig -x 8.8.8.8`.
4. In Wireshark filter `dns`, flush the cache (`ipconfig /flushdns`) and capture query and response. Find the transaction ID and TTL.
5. Try a zone transfer against the **intentionally open** training domain: `dig axfr zonetransfer.me @nsztm1.digi.ninja`. Never try this on domains you do not own or have no permission to test.
6. Change your DNS to `1.1.1.1` and compare lookup speed using `dig` (look at "Query time").
7. Use `whois example.com` to see registration data.

### 7. Quick summary
- DNS maps names to IPs via a hierarchy of servers with caching.
- Unencrypted by default, so spoofing and tunnelling are real risks.
- Know record types and use `dig` fluently. It is a core recon and defence skill.

---

## 7. HTTP

### 1. What it is
HTTP (Hypertext Transfer Protocol) is the Layer 7 protocol of the web. It is a request-response protocol over TCP (port 80).

### 2. How it works
**Request**
```
GET /login HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0
Cookie: session=abc123
```
**Response**
```
HTTP/1.1 200 OK
Content-Type: text/html
Set-Cookie: session=abc123; HttpOnly; Secure

<html>...</html>
```

**Methods:** GET (read), POST (send data), PUT (replace), DELETE (remove), HEAD (headers only), OPTIONS (allowed methods), PATCH (partial update).

**Status codes**

| Range | Meaning | Examples |
|-------|---------|----------|
| 1xx | Informational | 101 Switching Protocols |
| 2xx | Success | 200 OK, 201 Created |
| 3xx | Redirect | 301 Moved, 302 Found |
| 4xx | Client error | 400, 401, 403, 404 |
| 5xx | Server error | 500, 502, 503 |

**Important headers:** Host, User-Agent, Cookie, Set-Cookie, Authorization, Content-Type, Referer, Location, plus security headers (Content-Security-Policy, X-Frame-Options, Strict-Transport-Security).

HTTP is **stateless**; cookies and tokens keep a session. HTTP/2 and HTTP/3 improve speed but keep the same semantics.

### 3. Key facts
- Plain HTTP is **unencrypted**: anyone on the path can read it.
- Parameters appear in GET URLs, request bodies for POST.
- Cookies flags: `HttpOnly`, `Secure`, `SameSite`.

### 4. Security issues and attacks
- **Sniffing credentials and cookies** over plain HTTP.
- **Session hijacking** by stealing cookies.
- **Web attacks** at the application layer: SQL injection, XSS, CSRF, SSRF (covered in the web security part of your roadmap).
- **Information leakage** through headers and verbose errors.
- **Dangerous methods enabled** (PUT, DELETE, TRACE).

### 5. Defenses
Use HTTPS everywhere with HSTS, secure cookie flags, security headers, input validation, disable unneeded methods, hide server version info.

### 6. Hands-on lab (real life)
1. In Wireshark, filter `http` and visit `http://neverssl.com` (a plain HTTP site made for testing). Read the request and response in plain text with **Follow, HTTP Stream**.
2. Use `curl -v http://example.com` and then `curl -I https://example.com` to see headers.
3. Open your browser's **Developer Tools, Network tab**, load a site and inspect the headers, cookies and status codes.
4. Use `curl -X OPTIONS -i http://example.com` to see allowed methods.
5. Run your own server: `python3 -m http.server 8080`, request it, and read its log lines.
6. Check a site's security headers at securityheaders.com.
7. Intro to a safe target: set up **DVWA** or **OWASP Juice Shop** locally and watch its HTTP traffic in the browser dev tools and Wireshark.

### 7. Quick summary
- HTTP = stateless request/response protocol, plain text on port 80.
- Know methods, status codes, headers and cookies.
- It is the foundation of web attacks. Use HTTPS and secure headers.

---

## 8. HTTPS, TLS and SSL

### 1. What it is
HTTPS is HTTP carried inside **TLS** (Transport Layer Security), giving encryption, integrity and server authentication. SSL is the older, now-deprecated predecessor; people still say "SSL" but should use TLS 1.2 or 1.3.

### 2. How it works
**Simplified TLS 1.2 handshake**
1. **ClientHello:** supported versions, ciphers, random value.
2. **ServerHello + Certificate:** chosen cipher, server's certificate.
3. Client **verifies the certificate** (trusted CA, valid dates, matching domain).
4. **Key exchange:** both sides derive a shared session key.
5. Encrypted application data flows.

TLS 1.3 reduces the handshake to one round trip and removes weak options.

**Certificates:** issued by Certificate Authorities (CAs), contain the domain, public key, issuer, validity. A chain of trust runs from the leaf to an intermediate to a trusted root.

**Combines:** asymmetric cryptography (key exchange and authentication) and symmetric cryptography (fast bulk encryption).

### 3. Key facts
- Port 443.
- Provides: confidentiality, integrity, authentication (not anonymity: the destination IP and often the domain name via SNI stay visible).
- HSTS tells browsers to always use HTTPS.

### 4. Security issues and attacks
- **SSL stripping:** downgrading a victim from HTTPS to HTTP during a MITM.
- **Weak or old versions and ciphers** (SSLv3, TLS 1.0, RC4, export ciphers): vulnerable to attacks such as POODLE and BEAST.
- **Heartbleed** (an OpenSSL bug that leaked memory).
- **Fake or stolen certificates**, expired certificates, self-signed certificates that users learn to click through.
- **Phishing sites with valid HTTPS:** the padlock does not mean the site is trustworthy.

### 5. Defenses
Disable old protocols, use strong cipher suites, automate certificate renewal (Let's Encrypt), enable HSTS and certificate transparency monitoring, validate certificates in applications, never train users to ignore warnings.

### 6. Hands-on lab (real life)
1. In Wireshark filter `tls`. Visit an HTTPS site and find **Client Hello** and **Server Hello**. Check the TLS version and chosen cipher. Notice that the application data is unreadable.
2. View a certificate: click the padlock in your browser, then view the certificate chain, issuer and expiry.
3. `openssl s_client -connect example.com:443 -showcerts` to see the chain from the command line.
4. Test a server's configuration at **ssllabs.com/ssltest** (public sites only) or locally with `nmap --script ssl-enum-ciphers -p 443 <your-own-host>`.
5. Create a self-signed certificate: `openssl req -x509 -newkey rsa:2048 -nodes -keyout key.pem -out cert.pem -days 30`, then see the browser warning when you use it on a local test server.

### 7. Quick summary
- HTTPS = HTTP inside TLS: encryption, integrity, authentication.
- Trust comes from certificates and CAs.
- Use TLS 1.2+ only. The padlock proves encryption, not honesty.

---

## 9. SSH

### 1. What it is
SSH (Secure Shell) provides an encrypted remote command-line login and secure file transfer (SCP/SFTP). It replaces Telnet. Port 22 (TCP).

### 2. How it works
1. Client connects, both agree on protocol and ciphers.
2. The client checks the **server host key** (first time: "trust this fingerprint?", stored in `known_hosts`).
3. The user authenticates by **password** or by **public key**.
4. An encrypted session starts.

**Key-based authentication:** you keep a private key, the server holds your public key in `~/.ssh/authorized_keys`. Much stronger than passwords.

**Tunnelling (port forwarding)**
- Local: `ssh -L 8080:internal-host:80 user@jump` (reach an internal service through SSH).
- Remote: `ssh -R ...` (expose a local service through the server).
- Dynamic: `ssh -D 1080 user@server` (SOCKS proxy).

### 3. Key facts
- Config files: `/etc/ssh/sshd_config` (server), `~/.ssh/config` (client).
- Never share the private key. Protect it with a passphrase.
- A changed host key warning can mean a MITM attack or just a reinstalled server, so investigate.

### 4. Security issues and attacks
- **Brute force and password spraying** against port 22 (constantly happening on the internet).
- **Stolen or weak private keys.**
- **Root login enabled** and weak passwords.
- **Abused tunnelling** by attackers to pivot or bypass firewalls.
- **Old SSH versions** with known vulnerabilities.

### 5. Defenses
Disable password login and use keys, disable root login (`PermitRootLogin no`), use Fail2ban, restrict by IP or VPN, use a non-default port only as a minor extra, keep software updated, enable MFA, monitor `auth.log`.

### 6. Hands-on lab (real life)
1. On a Linux VM: `sudo apt install openssh-server` and `sudo systemctl start ssh`.
2. From your host or another VM: `ssh user@<VM-IP>`. Read the fingerprint prompt.
3. Generate keys: `ssh-keygen -t ed25519`, then copy with `ssh-copy-id user@<VM-IP>` and log in without a password.
4. Edit `/etc/ssh/sshd_config`: set `PasswordAuthentication no` and `PermitRootLogin no`, restart, and confirm password login is refused.
5. Look at the log: `sudo tail -f /var/log/auth.log` while you log in successfully and unsuccessfully.
6. Install `fail2ban` and trigger a ban by failing logins from a second VM.
7. Try a local tunnel to a web server running on the VM (`ssh -L 8080:localhost:8000 user@<VM-IP>`) and browse `http://localhost:8080`.
8. Capture the session in Wireshark and notice the encrypted payload.

### 7. Quick summary
- SSH = encrypted remote shell, tunnelling and file transfer on port 22.
- Use keys, disable passwords and root login.
- Attackers love SSH for brute force and tunnelling, so monitor it.

---

## Level 2 Checklist
- [ ] Explain IP header fields and what ICMP is used for
- [ ] Draw the TCP 3-way handshake and name the flags
- [ ] Compare TCP and UDP and give 3 uses of each
- [ ] Explain ARP resolution and how ARP spoofing works
- [ ] Walk through DHCP DORA and explain rogue DHCP
- [ ] Describe full DNS resolution and 6 record types
- [ ] Read an HTTP request and response including methods, status codes and cookies
- [ ] Explain the TLS handshake and what a certificate proves
- [ ] Set up SSH key authentication and harden `sshd_config`
- [ ] Capture and identify each of these 9 protocols in Wireshark

## Practice Questions
1. Why does a closed TCP port reply with RST, and how does a scanner use that?
2. Why are DNS and NTP often used for amplification attacks?
3. What would you see in Wireshark if someone ARP-poisoned your gateway?
4. What does DHCP snooping do?
5. Why does HTTPS not protect you from a phishing site?
6. A server returns `403` versus `404`. What is the difference?
7. What does the browser verify in a TLS certificate?
8. Name 4 hardening steps for SSH.
9. Which protocol would you look at to detect data exfiltration hidden in subdomains?

**Answers:** (1) RST says "nothing here" and so the scanner knows the host is alive and the port is closed (SYN-ACK means open). (2) They use UDP (spoofable) and small requests produce large replies. (3) Duplicate or changing MAC for the gateway IP, many unsolicited ARP replies. (4) Blocks DHCP offers from untrusted switch ports and builds a trusted binding table. (5) HTTPS encrypts but a criminal can get a valid certificate for a look-alike domain. (6) 403 = forbidden (exists but not allowed), 404 = not found. (7) Trusted CA chain, validity dates, domain name match, not revoked. (8) Keys only, no root login, Fail2ban, restrict source IPs, update, MFA. (9) DNS.

## Real-Life Practice Platforms for This Level
- **TryHackMe:** "Wireshark: The Basics", "Wireshark: Packet Operations", "DNS in Detail", "HTTP in Detail", "Intro to Networking"
- **Wireshark sample captures:** wiki.wireshark.org/SampleCaptures
- **malware-traffic-analysis.net:** real packet captures for analysis practice
- **Packet Tracer:** DHCP, DNS and ARP labs
- **OverTheWire "Bandit":** practice SSH while learning Linux
