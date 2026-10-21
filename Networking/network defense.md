# Level 4: Network Defense and Infrastructure

## Overview
Now that you know how protocols work and how they are attacked, this level covers how networks are protected: firewalls, intrusion detection and prevention, proxies, segmentation, load balancers and web application firewalls, and general hardening. You will build several of these yourself in a lab.

**Prerequisites:** Levels 1 to 3.

**Lab setup:** Two or three Linux VMs on a VirtualBox/VMware host-only or internal network (one acting as a firewall/router, one as a client, one as a server). Optional: pfSense or OPNsense VM.

## Table of Contents
1. [Firewalls](#1-firewalls)
2. [IDS and IPS](#2-ids-and-ips)
3. [Proxies and Reverse Proxies](#3-proxies-and-reverse-proxies)
4. [DMZ and Network Segmentation](#4-dmz-and-network-segmentation)
5. [Load Balancers and WAF](#5-load-balancers-and-waf)
6. [Network Hardening](#6-network-hardening)
7. [Level 4 Checklist](#level-4-checklist)
8. [Practice Questions](#practice-questions)

---

## 1. Firewalls

### 1. What it is
A firewall is a device or software that allows or blocks network traffic based on a set of rules.

### 2. How it works

| Type | Looks at | Notes |
|------|----------|-------|
| **Packet filter (stateless)** | IPs, ports, protocol per packet | Fast, simple, no memory of connections |
| **Stateful** | Tracks connection state | Allows replies to allowed outbound traffic automatically |
| **Application / proxy firewall** | Layer 7 content | Understands HTTP, DNS, etc. |
| **NGFW (Next-Generation)** | Stateful + application awareness + IPS + user identity + TLS inspection | Common in enterprises |
| **Host-based** | Traffic on a single machine | Windows Defender Firewall, `iptables`/`nftables`, `ufw` |

**Rule structure:** source, destination, protocol, port, action (allow, deny, drop). Rules are processed **top to bottom, first match wins**, ending with an **implicit deny**.

**Deny vs drop:** *reject* sends back an error, *drop* silently discards (slows scanners).

**Default policy:** "default deny" (allow only what is needed) is much safer than "default allow".

**Inbound vs outbound:** egress filtering (controlling outbound traffic) stops malware calling home and data leaving.

### 3. Key facts
- Firewalls enforce policy only on traffic that **passes through** them.
- Misconfigured firewalls are as dangerous as no firewall (an "any any allow" rule).
- A firewall does not stop attacks that come through allowed ports (for example web application attacks over 443).

### 4. Security issues and attacks
- Overly permissive or outdated rules ("rule bloat").
- Firewall evasion: fragmentation, using allowed ports (443, 53) for tunnelling, source-port tricks.
- Firewall appliance vulnerabilities and default credentials.
- Insiders or malware that make outbound connections on allowed ports.

### 5. Defenses
Default deny, least privilege rules, review rules regularly and remove unused ones, document each rule, log denies, patch the firewall, restrict management access, use egress filtering.

### 6. Hands-on lab (real life)
1. Check your host firewall: `sudo ufw status verbose` (Linux) or `Get-NetFirewallProfile` (Windows).
2. **Linux iptables basics** on a lab VM:
   ```
   sudo iptables -L -n -v                          # list rules
   sudo iptables -P INPUT DROP                     # default deny incoming (do this on a VM console, not SSH!)
   sudo iptables -A INPUT -i lo -j ACCEPT
   sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
   sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
   sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
   ```
   Test with `nmap` from another VM and compare open, closed and filtered ports. Try `-j DROP` versus `-j REJECT` and compare scan times.
3. Use `ufw`: `sudo ufw default deny incoming`, `sudo ufw allow 22/tcp`, `sudo ufw enable`, `sudo ufw status numbered`.
4. Make a **router/firewall VM** with two network adapters (one internal LAN, one outside) and enable forwarding (`echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward`). Write FORWARD rules so the LAN can reach the web but outside hosts cannot reach the LAN.
5. Install **pfSense** or **OPNsense** in a VM, open its web UI and create a block rule, then test and read the firewall logs.
6. Add an **egress rule** that blocks the LAN from using any DNS server except your own, then test with `dig @8.8.8.8 example.com`.
7. Save and restore rules (`sudo iptables-save > rules.v4`) to understand persistence.

### 7. Quick summary
- Firewalls filter traffic by rules. Stateful and NGFW are the norm.
- Default deny, first match wins, and review rules regularly.
- Control egress too, not just ingress.

---

## 2. IDS and IPS

### 1. What it is
- **IDS (Intrusion Detection System):** monitors traffic or hosts and **alerts** on suspicious activity.
- **IPS (Intrusion Prevention System):** sits **inline** and can **block** the traffic.

### 2. How it works
**Placement**
- **NIDS/NIPS:** watches network traffic (via a SPAN/mirror port or a network TAP). Examples: **Snort**, **Suricata**, Zeek.
- **HIDS/HIPS:** installed on hosts, watches logs and files. Examples: **OSSEC**, **Wazuh**.

**Detection methods**
- **Signature-based:** matches known attack patterns. Accurate for known threats, blind to new ones.
- **Anomaly-based:** compares against a normal baseline. Can find new attacks but produces more false positives.

**Rule example (Snort/Suricata):**
```
alert tcp any any -> $HOME_NET 22 (msg:"Possible SSH brute force"; flags:S; threshold:type both, track by_src, count 5, seconds 60; sid:1000001; rev:1;)
```
Parts: action, protocol, source, direction, destination, then options in brackets.

**Alert types:** True positive, false positive (harmless flagged), false negative (attack missed, the dangerous one), true negative.

**Zeek** is different: it records rich logs of network activity (connections, DNS, HTTP, files) for analysis rather than relying on rules.

### 3. Key facts
- IDS = detect and alert, IPS = detect and block.
- Encrypted traffic limits network IDS visibility unless TLS is decrypted or metadata is analysed.
- Tuning is essential. Untuned IDS drowns analysts in noise.

### 4. Security issues and attacks
- **Evasion:** fragmentation, encoding tricks, slow scans, encryption.
- **Alert fatigue**, where real alerts get missed.
- **Blind spots** (traffic that never passes the sensor).
- IPS misconfiguration blocking legitimate business traffic.

### 5. Defenses
Place sensors at key points (perimeter, between segments, in front of critical servers), keep rules updated, tune for your environment, combine NIDS and HIDS, forward alerts to a SIEM, test detection regularly.

### 6. Hands-on lab (real life)
1. Install **Suricata** on a Linux VM: `sudo apt install suricata`, update rules with `sudo suricata-update`.
2. Run it on your lab interface: `sudo suricata -c /etc/suricata/suricata.yaml -i <interface>`.
3. From another lab VM run a scan against it: `nmap -sS -p 1-1000 <suricata-VM-IP>` and `nmap -A <IP>`. Read `/var/log/suricata/fast.log` for alerts.
4. Write a **custom rule** in `/var/lib/suricata/rules/local.rules` to alert on any HTTP request containing the word `admin` in the URL, then reload and test with `curl "http://<IP>/admin"`.
5. Install **Snort** the same way and compare the rule syntax.
6. Test with a safe test: `curl http://testmynids.org/uid/index.html` (a harmless request designed to trigger the classic "GPL ATTACK_RESPONSE id check returned root" alert) and see if your IDS fires.
7. Run **Zeek** on a recorded pcap file (`zeek -r capture.pcap`) and explore `conn.log`, `dns.log` and `http.log`.
8. Install **Wazuh** (single-node, in a VM), enrol an agent, and watch file-integrity and login alerts appear.

### 7. Quick summary
- IDS alerts, IPS blocks. Network-based and host-based both matter.
- Signatures catch known attacks, anomaly detection finds the unusual.
- Tuning and central alert handling decide whether an IDS is useful.

---

## 3. Proxies and Reverse Proxies

### 1. What it is
- A **forward proxy** sits between internal users and the internet, making requests on their behalf.
- A **reverse proxy** sits in front of servers and receives requests from the internet on their behalf.

### 2. How it works
**Forward proxy (for example Squid):** users configure it (or traffic is redirected). It can cache, filter websites, log activity, require authentication, and hide internal client IPs.
**Transparent proxy:** intercepts traffic without client configuration.
**TLS inspection proxy:** decrypts and re-encrypts HTTPS, using a company-installed CA certificate, so content can be inspected.

**Reverse proxy (for example Nginx, HAProxy, Cloudflare):** terminates TLS, hides the real servers, distributes load, caches static content, adds security headers, and can enforce WAF rules.

**SOCKS proxy:** lower-level, forwards any TCP (and sometimes UDP) traffic. Used by SSH `-D` and tools such as Tor.

### 3. Key facts
- Forward proxy protects and controls **clients**. Reverse proxy protects and optimises **servers**.
- `X-Forwarded-For` header carries the original client IP when a proxy is in the middle.
- Proxy logs are valuable evidence of user web activity.

### 4. Security issues and attacks
- **Open proxies** abused to hide attacker traffic or relay spam.
- **Misconfigured reverse proxies** that expose internal admin pages or allow request smuggling.
- **TLS inspection risks:** a stolen interception CA key gives access to all traffic, and bad inspection weakens security.
- Spoofed `X-Forwarded-For` values to bypass IP-based controls or poison logs.
- **Proxy bypass** through direct connections, VPNs or tunnels.

### 5. Defenses
Require authentication on proxies, never run open proxies, block direct outbound traffic so only the proxy can reach the internet, protect inspection CA keys, validate and sanitise forwarded headers, keep proxy software patched, log everything.

### 6. Hands-on lab (real life)
1. Install **Squid** on a lab VM: `sudo apt install squid`. Edit `/etc/squid/squid.conf` to allow only your lab subnet and add a **blocklist**: `acl blocked dstdomain .facebook.com` and `http_access deny blocked`.
2. Configure a client browser or `export http_proxy=http://<squid-IP>:3128` and visit blocked and allowed sites. Watch `/var/log/squid/access.log`.
3. Build a **reverse proxy** with Nginx in front of a simple web server:
   ```
   server {
       listen 80;
       location / { proxy_pass http://<backend-IP>:8080; }
   }
   ```
   Check that clients only ever talk to the proxy, and confirm the backend's logs show the proxy IP. Then add `proxy_set_header X-Forwarded-For $remote_addr;`.
4. Add TLS to Nginx using a self-signed certificate, and add security headers (`add_header X-Frame-Options DENY;`, HSTS).
5. Create a SOCKS proxy with SSH: `ssh -D 1080 user@<VM>`, point your browser at `localhost:1080`, and check your visible IP changes (use `curl --socks5 localhost:1080 ifconfig.me`).
6. Use **Burp Suite Community** or **OWASP ZAP** as an intercepting proxy against a local test app (for example DVWA) to see requests and responses.

### 7. Quick summary
- Forward proxy = for clients. Reverse proxy = for servers.
- Great for filtering, caching, logging and hiding backends.
- Never leave a proxy open and always protect its logs and keys.

---

## 4. DMZ and Network Segmentation

### 1. What it is
- **Segmentation** splits a network into zones so that a breach in one zone cannot spread freely.
- A **DMZ (Demilitarised Zone)** is a semi-trusted zone that hosts public-facing services (web, mail, DNS) separately from the internal network.

### 2. How it works
**Typical layout**
```
Internet --> [Firewall] --> DMZ (web, mail servers)
                |
                +--> Internal LAN (users)
                +--> Server zone (databases)
                +--> Management zone (admin tools)
                +--> Guest / IoT zone
```
- Internet to DMZ: allowed on specific ports only.
- DMZ to internal: **heavily restricted** (for example web server to database on one port only).
- Internal to DMZ: allowed for administration/users as needed.
- Internal to Internet: through proxy or filtered.

**Techniques:** VLANs plus firewall or ACL rules between them, separate subnets, **micro-segmentation** (per-workload rules), **Zero Trust** (never trust by location, always verify).

**Single vs dual firewall DMZ:** dual firewall (one on each side of the DMZ, ideally from different vendors) adds depth.

### 3. Key facts
- Flat networks let attackers move laterally everywhere ("one phishing email, whole company down").
- Public servers are assumed to get compromised eventually, so design so that it does not matter much.
- Management interfaces should be on a separate, tightly controlled network.

### 4. Security issues and attacks
- **Lateral movement** across flat networks.
- **VLAN hopping** (see Level 1) and misconfigured trunks.
- Overly broad "allow all" rules between zones.
- DMZ hosts with a direct path to the internal database.
- Forgotten dual-homed servers that bridge zones.

### 5. Defenses
Segment by function and sensitivity, default deny between zones, restrict DMZ to internal connections, separate management and backup networks, isolate IoT and guests, monitor east-west traffic, adopt Zero Trust principles.

### 6. Hands-on lab (real life)
1. **Packet Tracer / GNS3 design:** build a network with an internet cloud, a firewall/router, a DMZ with a web server, an internal LAN and a database server. Write ACLs so that:
   - Internet can reach only the DMZ web server on 80/443.
   - DMZ web server can reach the database on 3306 only.
   - Internet cannot reach the internal LAN or database.
   Test each path with ping and `telnet <ip> <port>` and document the results.
2. **VirtualBox version:** create three internal networks (`lan`, `dmz`, `wan`), a firewall VM with three adapters, and VMs in each zone. Use `iptables` FORWARD rules to enforce the same policy.
3. **Home practice:** enable your router's **guest Wi-Fi** and check that guests cannot reach your main devices (try pinging your PC from a guest device).
4. Put smart-home devices on a separate IoT VLAN/SSID if your router allows it.
5. Draw a segmentation diagram for a fictional company with servers, staff, CCTV, printers and guests, and list the allowed flows in a table.
6. Practise attack path thinking: from the DMZ web server, what could an attacker reach if you had a flat network versus a segmented one?

### 7. Quick summary
- Segmentation limits the blast radius of an attack.
- DMZ holds public services away from the internal network.
- Default deny between zones, and keep management separate.

---

## 5. Load Balancers and WAF

### 1. What it is
- A **load balancer** distributes incoming traffic across several servers for availability and performance.
- A **WAF (Web Application Firewall)** inspects HTTP(S) traffic and blocks web attacks such as SQL injection and XSS.

### 2. How it works
**Load balancing**
- **Layer 4:** decides by IP and port (fast).
- **Layer 7:** decides by URL, headers, cookies (smart routing).
- **Algorithms:** round robin, least connections, IP hash.
- **Health checks** remove failed servers from rotation.
- **Session persistence (sticky sessions)** keeps a user on the same server.
- Examples: HAProxy, Nginx, AWS ELB, F5.

**WAF**
- Sits in front of web apps (usually as a reverse proxy) and applies rules, for example the **OWASP Core Rule Set**.
- **Positive model** (allow known-good) or **negative model** (block known-bad).
- Examples: **ModSecurity** (open source), Cloudflare WAF, AWS WAF.
- Often combined with **rate limiting, bot protection and DDoS mitigation**.

### 3. Key facts
- A WAF is a **safety net, not a fix**: vulnerable code should still be fixed.
- WAFs can be bypassed by encoding or novel payloads, and can block legitimate requests (false positives).
- Load balancers terminate TLS, so protect their certificates and configuration.

### 4. Security issues and attacks
- **WAF bypass** using encoding, case changes, comments or unusual parameters.
- **Origin exposure:** attackers find the real server IP and attack it directly, bypassing the WAF.
- **Session persistence issues** and cookie tampering.
- **Load balancer as single point of failure** or high-value target.
- **DDoS** aimed at the balancer.

### 5. Defenses
Restrict the origin server to accept traffic only from the WAF/load balancer, keep rule sets updated, run the WAF in detection mode first and tune, combine with secure coding and testing, redundancy for balancers, rate limiting and bot controls.

### 6. Hands-on lab (real life)
1. **Nginx load balancer** in front of two simple web servers:
   ```
   upstream backend {
       server 192.168.56.11:8080;
       server 192.168.56.12:8080;
   }
   server { listen 80; location / { proxy_pass http://backend; } }
   ```
   Make each backend show a different page, refresh repeatedly, and watch round-robin. Stop one backend and watch the health failure handling.
2. Try **HAProxy** with `balance leastconn` and enable its stats page.
3. Install **ModSecurity** with Nginx or Apache and the **OWASP Core Rule Set**. Test with a harmless attack string against **your own** test page, for example `curl "http://localhost/?id=1' OR '1'='1"`, and check `modsec_audit.log`.
4. Run a lab web app (**DVWA** or **Juice Shop**) behind the WAF and compare which of your test requests get blocked and which pass.
5. Create a free **Cloudflare** account for a domain you own and see how it hides the origin IP. Then check whether the origin IP is still reachable directly.
6. Compare WAF modes: detection only versus blocking mode.

### 7. Quick summary
- Load balancers share traffic and remove failed servers.
- WAFs block common web attacks but cannot replace secure code.
- Always lock the origin so attackers cannot bypass the front door.

---

## 6. Network Hardening

### 1. What it is
Hardening is reducing the attack surface of devices and networks by removing what is not needed and securing what remains.

### 2. How it works
**General principles:** least privilege, defence in depth, secure by default, minimise services, patch regularly, log and monitor, assume breach.

**Hardening checklist**
- **Services and ports:** disable unused services, close unused ports, remove legacy protocols (Telnet, FTP, SMB1, SNMPv1/2).
- **Accounts:** change default passwords, unique admin credentials, MFA, disable unused accounts, lockout policies.
- **Patching:** OS, firmware (routers, switches, IoT) and applications on a schedule.
- **Network devices:** secure management access (SSH only, from a management VLAN), disable HTTP management, set banners, disable unused switch ports, enable port security, DHCP snooping and Dynamic ARP Inspection.
- **Encryption:** TLS 1.2+/1.3, WPA3, VPN for remote access.
- **Logging:** central logs, accurate time, alerting.
- **Backups:** offline/immutable copies and tested restores.
- **Benchmarks:** follow **CIS Benchmarks** or vendor hardening guides.

**Vulnerability management cycle:** discover assets, scan for vulnerabilities, prioritise (by severity and exposure), fix, verify, repeat.

### 3. Key facts
- Most breaches use known weaknesses: default credentials, missing patches, exposed services, misconfigurations.
- Hardening is continuous, not a one-time job.
- Documentation and asset inventory come first: you cannot protect what you do not know.

### 4. Security issues and attacks
- **Default credentials** on routers, cameras and printers.
- **Configuration drift** over time.
- **Shadow IT** and forgotten devices.
- **Unpatched firmware**, especially on edge devices.

### 5. Defenses
Baseline configuration, automated compliance scanning, change management, regular audits, patch SLAs, secure configuration templates (golden images), periodic penetration testing.

### 6. Hands-on lab (real life)
1. **Audit your own machine's open ports:** `sudo ss -tulpn` and for each listening service decide: needed or not? Disable the extras (`sudo systemctl disable --now <service>`).
2. **Scan yourself** with `nmap -sV -p- localhost` and compare against your expectations.
3. Run **Lynis**: `sudo apt install lynis && sudo lynis audit system`. Read the hardening index and fix 5 suggestions.
4. Run **OpenSCAP** or the CIS-CAT lite tool against a lab VM and review a benchmark report.
5. **Secure your home router:** change the admin password, disable WPS and remote management, update firmware, turn off UPnP, enable WPA3/WPA2-AES, and enable the guest network.
6. Run a vulnerability scan with **OpenVAS/Greenbone** or **Nessus Essentials** against your lab VMs (**never** against systems you do not own), read the report and fix the top three findings.
7. Harden SSH with the steps from Level 2 and check your result with `ssh-audit <IP>`.
8. Create a one-page hardening checklist for a Linux server and for a small office router, in your own words.

### 7. Quick summary
- Hardening = remove, restrict, patch, monitor.
- Most real breaches exploit basics: defaults, patches and exposed services.
- Use CIS Benchmarks and automated scanning to stay consistent.

---

## Level 4 Checklist
- [ ] Explain stateless vs stateful vs NGFW firewalls
- [ ] Write basic iptables or ufw rules with a default-deny policy
- [ ] Explain why egress filtering matters
- [ ] Differentiate IDS from IPS and NIDS from HIDS
- [ ] Read and write a simple Snort/Suricata rule
- [ ] Explain forward vs reverse proxy with an example of each
- [ ] Design a DMZ and write the allowed traffic flows
- [ ] Explain what a WAF does and why it is not a substitute for secure code
- [ ] Set up a simple Nginx load balancer
- [ ] Complete a hardening audit of one Linux machine and your home router

## Practice Questions
1. What is the difference between a firewall `DROP` and `REJECT`, and which is stealthier?
2. Why is a false negative worse than a false positive in an IDS?
3. Where would you place an IPS and why?
4. What risk does a TLS-inspection proxy create?
5. In a DMZ design, which connection must be most restricted?
6. How can an attacker bypass a WAF-protected site, and how do you prevent it?
7. Give 5 hardening steps for a new Linux web server.
8. Why is "default allow" dangerous?

**Answers:** (1) DROP silently discards (stealthier, slows scanning), REJECT sends an error. (2) A false negative is a missed real attack. (3) Inline at chokepoints such as the perimeter or in front of critical segments, so it can block traffic. (4) The interception CA and the proxy become high-value targets, and improper handling weakens security. (5) DMZ to internal network. (6) By finding and attacking the origin IP directly. Restrict the origin to accept traffic only from the WAF. (7) Patch, disable unused services, firewall with default deny, SSH keys only and no root login, logging and Fail2ban. (8) Anything not explicitly considered is allowed, including attacker traffic.

## Real-Life Practice Platforms for This Level
- **TryHackMe:** "Firewalls", "Snort", "Snort Challenge", "Suricata", "Zeek", "Wazuh", "Network Security" modules
- **Cisco Packet Tracer / GNS3 / EVE-NG:** build segmented networks and ACLs
- **pfSense / OPNsense:** free firewall distributions for VMs
- **CIS Benchmarks:** free hardening guides at cisecurity.org
- **Security Onion:** a free all-in-one distribution with Suricata, Zeek and dashboards, great for blue-team practice
- **Blue Team Labs Online / CyberDefenders:** investigation challenges
