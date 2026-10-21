# Level 1: Networking Fundamentals

## Overview
This level builds the base for everything else. You will learn what a network is, how data is organised into layers (OSI and TCP/IP), how devices are addressed (IP, MAC, ports), how traffic is switched and routed, and how NAT lets private networks reach the internet.

**Prerequisites:** Basic computer use. Nothing else.

**Lab setup for this level:** Your own PC or laptop, a terminal (Command Prompt/PowerShell on Windows, Terminal on Linux/macOS), and optionally VirtualBox with one Linux VM.

## Table of Contents
1. [What is a Network](#1-what-is-a-network)
2. [OSI Model](#2-osi-model)
3. [TCP/IP Model](#3-tcpip-model)
4. [IP Addressing](#4-ip-addressing)
5. [Subnetting and CIDR](#5-subnetting-and-cidr)
6. [MAC Addresses and Switching](#6-mac-addresses-and-switching)
7. [Routing Basics](#7-routing-basics)
8. [Ports and Sockets](#8-ports-and-sockets)
9. [NAT and PAT](#9-nat-and-pat)
10. [Level 1 Checklist](#level-1-checklist)
11. [Practice Questions](#practice-questions)

---

## 1. What is a Network

### 1. What it is
A network is two or more devices connected so they can share data and resources. The internet is a network of networks.

### 2. How it works
Devices (hosts) connect through media (cables, Wi-Fi) and intermediate devices (switches, routers). Data is broken into small units called **packets**, sent across the network, and reassembled at the destination.

**Network types by size**

| Type | Full name | Example |
|------|-----------|---------|
| PAN | Personal Area Network | Phone + Bluetooth earbuds |
| LAN | Local Area Network | Home or office network |
| MAN | Metropolitan Area Network | City-wide ISP or campus network |
| WAN | Wide Area Network | The internet, company links between cities |

**Topologies:** Bus, Star (most common today), Ring, Mesh, Hybrid.

**Models of communication:**
- **Client-Server:** clients request, a server responds (web, email).
- **Peer-to-Peer:** every device can act as both (BitTorrent, file sharing).

**Common devices**

| Device | Job |
|--------|-----|
| Hub | Repeats data to all ports (obsolete, insecure) |
| Switch | Forwards frames to the correct port using MAC addresses |
| Router | Connects different networks, forwards packets using IP addresses |
| Access Point | Lets wireless devices join a wired network |
| Modem | Converts ISP signal to digital data |
| Firewall | Filters traffic based on rules |

### 3. Key facts
- Bandwidth = capacity (Mbps/Gbps). Latency = delay (ms). Throughput = actual speed achieved.
- Protocol = an agreed set of rules for communication.

### 4. Security issues and attacks
- Flat networks let an attacker who gets in reach everything.
- Hubs allow easy sniffing since everyone receives all traffic.
- Unsecured Wi-Fi exposes traffic to anyone nearby.

### 5. Defenses
Segment networks, use switches instead of hubs, encrypt wireless, apply least privilege.

### 6. Hands-on lab (real life)
1. Draw your home network on paper: ISP modem, router, your devices. Label which are switches, routers and end hosts.
2. Log in to your router admin page (usually `192.168.0.1` or `192.168.1.1`) and list connected devices. Do you recognise all of them?
3. Run `ipconfig /all` (Windows) or `ip a` (Linux) and note your IP, gateway and DNS.

### 7. Quick summary
- Network = connected devices sharing data as packets.
- LAN is local, WAN spans large distances, the internet is a WAN of WANs.
- Switch works with MAC, router works with IP.
- Star topology dominates modern LANs.

---

## 2. OSI Model

### 1. What it is
The OSI (Open Systems Interconnection) model is a 7-layer reference model that describes how data moves from one application to another across a network. It is a teaching and troubleshooting tool.

### 2. How it works
Data moves **down** the layers on the sender (encapsulation, each layer adds a header) and **up** on the receiver (decapsulation).

| # | Layer | Job | Data unit | Examples |
|---|-------|-----|-----------|----------|
| 7 | Application | Network services for apps | Data | HTTP, DNS, SMTP, FTP |
| 6 | Presentation | Format, encryption, compression | Data | TLS/SSL, JPEG, ASCII |
| 5 | Session | Open, manage, close sessions | Data | NetBIOS, RPC, sessions |
| 4 | Transport | End-to-end delivery, ports | Segment/Datagram | TCP, UDP |
| 3 | Network | Logical addressing, routing | Packet | IP, ICMP, routers |
| 2 | Data Link | Physical addressing, framing | Frame | Ethernet, MAC, switches, ARP |
| 1 | Physical | Bits on the medium | Bits | Cables, Wi-Fi radio, hubs |

**Memory trick (top to bottom):** All People Seem To Need Data Processing.
**Bottom to top:** Please Do Not Throw Sausage Pizza Away.

### 3. Key facts
- Troubleshooting approach: start at Layer 1 (is the cable plugged in?) and move up.
- A device is described by the highest layer it works at: switch = L2, router = L3, firewall = L3 to L7.

### 4. Security issues and attacks by layer

| Layer | Example attacks |
|-------|-----------------|
| 7 | SQL injection, XSS, phishing, DNS abuse |
| 6 | SSL stripping, weak ciphers |
| 5 | Session hijacking |
| 4 | SYN flood, port scanning |
| 3 | IP spoofing, ICMP flood, route attacks |
| 2 | ARP spoofing, MAC flooding, VLAN hopping |
| 1 | Cable tapping, jamming, physical theft |

### 5. Defenses
Defense in depth: put controls at several layers (physical locks, port security, firewalls, TLS, WAF, secure coding).

### 6. Hands-on lab (real life)
1. Open Wireshark (install it from wireshark.org), capture on your Wi-Fi/Ethernet adapter, and open any website.
2. Click one packet and expand the sections in the middle pane. You will see **Frame (L1)**, **Ethernet (L2)**, **IP (L3)**, **TCP (L4)** and **HTTP/TLS (L7)**. Write down one field from each layer.
3. For each of these, name the OSI layer: a Wi-Fi card, an IP address, a port number, a web page, an encrypted connection.

### 7. Quick summary
- 7 layers, from Physical to Application.
- Encapsulation goes down, decapsulation goes up.
- Security thinking: ask "which layer is this attack at?"

---

## 3. TCP/IP Model

### 1. What it is
The TCP/IP model is the practical 4-layer model the internet actually uses. OSI is the reference, TCP/IP is the implementation.

### 2. How it works

| TCP/IP layer | OSI layers covered | Examples |
|--------------|--------------------|----------|
| Application | 5, 6, 7 | HTTP, DNS, SSH, SMTP, DHCP |
| Transport | 4 | TCP, UDP |
| Internet | 3 | IP, ICMP, ARP (often placed between 2 and 3) |
| Network Access (Link) | 1, 2 | Ethernet, Wi-Fi |

**A web request, step by step**
1. Browser creates an HTTP request (Application).
2. TCP splits it into segments and adds source/destination ports (Transport).
3. IP adds source/destination IP addresses (Internet).
4. Ethernet adds MAC addresses and sends frames on the wire (Link).
5. The receiver reverses the process.

### 3. Key facts
- OSI has 7 layers, TCP/IP has 4. Industry uses OSI terms (Layer 2, Layer 3) when talking, TCP/IP when building.
- Each layer only talks to the same layer on the other side (peer-to-peer logical communication).

### 4. Security issues and attacks
Same as the matching OSI layers. Attacks often cross layers, for example a phishing email (L7) leading to credential theft then lateral movement (L2/L3).

### 5. Defenses
Layered controls matching each layer.

### 6. Hands-on lab (real life)
1. In Wireshark, apply the filter `http || dns`. Visit a plain HTTP site (for example `http://neverssl.com`).
2. Find the DNS lookup first, then the TCP handshake, then the HTTP GET. Write down the order in your notes: this is the real flow of a web request.
3. Draw the TCP/IP stack and mark which fields you saw in each layer.

### 7. Quick summary
- 4 layers: Application, Transport, Internet, Link.
- Maps onto the 7-layer OSI model.
- Every web request touches all layers.

---

## 4. IP Addressing

### 1. What it is
An IP address is a logical address that identifies a device on a network and tells routers where to send packets.

### 2. How it works
**IPv4:** 32 bits, written as four decimal numbers (octets), for example `192.168.1.10`. About 4.3 billion addresses.
**IPv6:** 128 bits, written in hex, for example `2001:db8::1`. Solves address exhaustion.

**Classes (historical):**

| Class | First octet | Default mask |
|-------|-------------|--------------|
| A | 1-126 | 255.0.0.0 (/8) |
| B | 128-191 | 255.255.0.0 (/16) |
| C | 192-223 | 255.255.255.0 (/24) |
| D | 224-239 | Multicast |
| E | 240-255 | Reserved |

**Private ranges (not routable on the internet):**

| Range | CIDR |
|-------|------|
| 10.0.0.0 - 10.255.255.255 | 10.0.0.0/8 |
| 172.16.0.0 - 172.31.255.255 | 172.16.0.0/12 |
| 192.168.0.0 - 192.168.255.255 | 192.168.0.0/16 |

**Special addresses:**
- `127.0.0.1`: loopback (this machine).
- `169.254.x.x`: APIPA, self-assigned when DHCP fails.
- `0.0.0.0`: "any" or "unknown".
- `255.255.255.255`: limited broadcast.

**Static vs dynamic:** static is set by hand (servers, routers), dynamic is assigned by DHCP (laptops, phones).

### 3. Key facts
- Public IP = visible on the internet. Private IP = inside your LAN.
- Each interface has its own IP.
- IPv6 has no broadcast (uses multicast) and no need for NAT in design.

### 4. Security issues and attacks
- IP spoofing (faking the source address).
- Open services on public IPs get scanned within minutes.
- IPv6 is often forgotten: unmonitored IPv6 traffic can bypass IPv4-only firewalls.

### 5. Defenses
Ingress/egress filtering, firewalls for both IPv4 and IPv6, keeping an asset inventory.

### 6. Hands-on lab (real life)
1. Find your **private IP**: `ipconfig` or `ip a`.
2. Find your **public IP**: visit `https://ifconfig.me` or run `curl ifconfig.me`. Notice that it is different from your private IP.
3. Run `ping 127.0.0.1` and `ping -6 ::1` to test loopback.
4. Disconnect from the network, release and renew your IP (`ipconfig /release`, `ipconfig /renew`) and watch what happens. On Linux: `sudo dhclient -r && sudo dhclient`.
5. Classify these: `10.4.5.6`, `172.20.1.1`, `172.32.1.1`, `8.8.8.8`, `169.254.3.4` (answers: private, private, public, public, APIPA).

### 7. Quick summary
- IPv4 = 32 bits, IPv6 = 128 bits.
- Know the three private ranges by heart.
- Loopback, APIPA and broadcast are special.

---

## 5. Subnetting and CIDR

### 1. What it is
Subnetting splits one large network into smaller ones. CIDR (Classless Inter-Domain Routing) writes the network size as a prefix, for example `/24`.

### 2. How it works
A subnet mask separates the **network part** from the **host part**.

`192.168.1.0/24` means the first 24 bits are the network, the last 8 bits are for hosts.

**Formulas**
- Number of addresses = 2^(32 - prefix)
- Usable hosts = 2^(32 - prefix) - 2 (minus network and broadcast addresses)

**Cheat sheet**

| CIDR | Mask | Addresses | Usable hosts |
|------|------|-----------|--------------|
| /24 | 255.255.255.0 | 256 | 254 |
| /25 | 255.255.255.128 | 128 | 126 |
| /26 | 255.255.255.192 | 64 | 62 |
| /27 | 255.255.255.224 | 32 | 30 |
| /28 | 255.255.255.240 | 16 | 14 |
| /29 | 255.255.255.248 | 8 | 6 |
| /30 | 255.255.255.252 | 4 | 2 |

**Worked example:** split `192.168.1.0/24` into 4 subnets.
- Borrow 2 bits, so the new prefix is `/26`, with 64 addresses per subnet.

| Subnet | Network | First host | Last host | Broadcast |
|--------|---------|------------|-----------|-----------|
| 1 | 192.168.1.0 | .1 | .62 | .63 |
| 2 | 192.168.1.64 | .65 | .126 | .127 |
| 3 | 192.168.1.128 | .129 | .190 | .191 |
| 4 | 192.168.1.192 | .193 | .254 | .255 |

**Is a host on my subnet?** Compare the network part. With `192.168.1.70/26`, the network is `192.168.1.64`. So `192.168.1.100` is local and `192.168.1.130` is not.

### 3. Key facts
- A larger prefix number means a smaller network.
- The "block size" trick: 256 minus the mask's last octet value (for 255.255.255.192, block size = 64).
- Hosts in different subnets need a router to talk.

### 4. Security issues and attacks
- Attackers use subnet knowledge to map and scan internal networks.
- Poor subnet design means flat networks with no isolation.

### 5. Defenses
Use subnets to separate users, servers, guests and IoT, then control traffic between them with firewalls (see Level 4).

### 6. Hands-on lab (real life)
1. Find your mask with `ipconfig` and compute your network address, broadcast address and usable hosts by hand.
2. Use a calculator such as `ipcalc` on Linux (`sudo apt install ipcalc`, then `ipcalc 192.168.1.0/26`) and check your manual answer.
3. Design a network for a small company: 50 staff PCs, 10 servers, 20 guest devices, 5 printers. Give each group its own subnet inside `10.10.0.0/16`.
4. Practice daily for a week with 5 random problems from sites like subnettingpractice.com.

### 7. Quick summary
- Subnet mask divides network and host bits.
- Usable hosts = 2^h - 2.
- Practice until subnetting is automatic. It shows up in every interview and exam.

---

## 6. MAC Addresses and Switching

### 1. What it is
A MAC address is a 48-bit hardware address burned into a network interface, for example `00:1A:2B:3C:4D:5E`. Switches use MAC addresses to forward frames inside a LAN (Layer 2).

### 2. How it works
- First 24 bits = OUI (manufacturer), last 24 bits = device number.
- A switch keeps a **MAC address table** (CAM table) mapping MAC addresses to ports. It learns by reading the source MAC of incoming frames.
- If the destination MAC is known, the frame goes out of one port. If unknown, the switch **floods** it to all ports.
- **Broadcast MAC:** `FF:FF:FF:FF:FF:FF` (to everyone).
- **VLANs** logically split one physical switch into multiple separate networks. Tagged using 802.1Q.
- **Trunk port** carries multiple VLANs, **access port** carries one.

### 3. Key facts
- MAC is local (changes at each hop), IP is end-to-end.
- Hub: one collision domain. Switch: each port is its own collision domain. Router: separates broadcast domains.

### 4. Security issues and attacks
- **MAC flooding:** overwhelm the CAM table so the switch behaves like a hub.
- **MAC spoofing:** change your MAC to impersonate another device.
- **VLAN hopping:** abuse trunk negotiation or double tagging.
- **ARP spoofing** (see Level 2).

### 5. Defenses
Port security (limit MACs per port), disable unused ports, disable DTP on access ports, use dedicated native VLAN, DHCP snooping and Dynamic ARP Inspection.

### 6. Hands-on lab (real life)
1. Find your MAC: `ipconfig /all` or `ip link`. Look up its vendor on a site such as macvendors.com.
2. View neighbours: `arp -a`. Match each entry's vendor to a real device in your home.
3. Use Cisco **Packet Tracer** (free) to build one switch with 3 PCs and watch the MAC table fill with `show mac address-table`.
4. In Packet Tracer, create two VLANs on one switch and confirm PCs in different VLANs cannot ping each other.

### 7. Quick summary
- MAC = Layer 2 hardware address, switch = MAC table.
- VLANs isolate traffic logically.
- Port security and snooping stop most Layer 2 attacks.

---

## 7. Routing Basics

### 1. What it is
Routing is the process of forwarding packets between different networks using IP addresses. Routers make these decisions.

### 2. How it works
- A router has a **routing table**: destination network, next hop, interface, metric.
- It picks the **longest prefix match** (most specific route).
- **Default gateway:** the router a host uses for anything outside its own subnet.
- **Static routing:** manually configured. **Dynamic routing:** protocols such as RIP, OSPF (inside an organisation) and BGP (between ISPs).
- **TTL (Time to Live):** decremented at each router, packet is dropped at 0 (prevents loops). This is how `traceroute` works.

**Host decision:** is the destination on my subnet? Yes: send directly (ARP for MAC). No: send to the default gateway.

### 3. Key facts
- Hops = routers crossed.
- Switch = same network, router = between networks.
- Typical default TTL: 64 (Linux), 128 (Windows).

### 4. Security issues and attacks
- Route hijacking (BGP hijacks have redirected real internet traffic).
- Rogue routers or fake routing updates on internal networks.
- Misconfigured routes exposing internal networks.

### 5. Defenses
Authenticate routing protocols, filter routes, use ACLs, monitor routing changes.

### 6. Hands-on lab (real life)
1. View your routing table: `route print` (Windows) or `ip route` (Linux). Find your default gateway.
2. Run `tracert google.com` (Windows) or `traceroute google.com` (Linux). Count hops and note where latency jumps.
3. Run `traceroute` to a site on another continent and compare.
4. In Packet Tracer, connect two LANs with a router, add static routes and test with ping.

### 7. Quick summary
- Routers connect networks using IP and routing tables.
- Default gateway is the exit door of a subnet.
- TTL prevents loops and powers traceroute.

---

## 8. Ports and Sockets

### 1. What it is
A port is a 16-bit number (0-65535) that identifies a specific service or application on a host. An IP address plus a port is a **socket**, for example `192.168.1.10:443`.

### 2. How it works
- **Well-known ports:** 0-1023 (system services).
- **Registered ports:** 1024-49151.
- **Dynamic/ephemeral ports:** 49152-65535 (client side, chosen randomly).
- A connection is identified by 5 values: protocol, source IP, source port, destination IP, destination port.

**Must-know ports**

| Port | Service | Protocol |
|------|---------|----------|
| 20/21 | FTP | TCP |
| 22 | SSH / SFTP | TCP |
| 23 | Telnet | TCP |
| 25 | SMTP | TCP |
| 53 | DNS | UDP/TCP |
| 67/68 | DHCP | UDP |
| 80 | HTTP | TCP |
| 110 | POP3 | TCP |
| 123 | NTP | UDP |
| 143 | IMAP | TCP |
| 161/162 | SNMP | UDP |
| 389 | LDAP | TCP/UDP |
| 443 | HTTPS | TCP |
| 445 | SMB | TCP |
| 3306 | MySQL | TCP |
| 3389 | RDP | TCP |

**Port states:** open (service listening), closed (reachable, nothing listening), filtered (firewall drops or hides it).

### 3. Key facts
- Ports are how one IP can run many services.
- Services can run on non-standard ports, so do not trust the port number alone.

### 4. Security issues and attacks
- **Port scanning** reveals which services are exposed.
- Unneeded open ports increase the attack surface.
- Default ports are attacked first (22, 3389, 445).

### 5. Defenses
Close unused ports, firewall rules, change default exposure (not as a substitute for patching), monitor listening ports.

### 6. Hands-on lab (real life)
1. See your machine's listening ports: `netstat -ano` (Windows) or `ss -tulpn` (Linux). For each, identify the program.
2. Open a website, then run `netstat -an` and find the connection to port 443 and your ephemeral source port.
3. Start a simple server: `python3 -m http.server 8080`, open `http://localhost:8080`, then rerun `ss -tulpn` to see the new listening port.
4. Memorise the ports table with flashcards (Anki or paper).

### 7. Quick summary
- Port = service identifier, socket = IP + port.
- Know the common ports by heart.
- Open/closed/filtered are the three scan results.

---

## 9. NAT and PAT

### 1. What it is
NAT (Network Address Translation) rewrites IP addresses as packets pass through a router, letting many private devices share one public IP. PAT (Port Address Translation, also called NAT overload) does this by also rewriting port numbers.

### 2. How it works
**Types**
- **Static NAT:** one private IP maps to one public IP (used for servers).
- **Dynamic NAT:** private IPs map to a pool of public IPs.
- **PAT:** many private IPs share one public IP using different source ports (what your home router does).

**Example (PAT)**

| Inside (private) | Translated (public) |
|------------------|---------------------|
| 192.168.1.10:50001 | 203.0.113.5:40001 |
| 192.168.1.11:50001 | 203.0.113.5:40002 |

The router stores this in a NAT table and reverses it for the reply.

**Port forwarding** tells the router to send incoming traffic on a public port to a chosen internal host (for example a game server or home camera).

### 3. Key facts
- NAT conserves IPv4 addresses.
- NAT is not a firewall, but by default it blocks unsolicited inbound connections as a side effect.
- Outsiders see only your public IP, not individual internal hosts.

### 4. Security issues and attacks
- Port forwarding exposes internal services directly.
- UPnP can open ports automatically without you knowing.
- NAT can hide the real source in logs, making incident tracing harder.

### 5. Defenses
Disable UPnP, minimise port forwards, patch exposed services, log NAT translations.

### 6. Hands-on lab (real life)
1. Compare your private IP (`ipconfig`) with your public IP (`curl ifconfig.me`).
2. Check the same public IP from your phone on Wi-Fi: it should be identical. On mobile data it will be different.
3. Log in to your router and check whether UPnP is enabled and list any port forwarding rules. Disable what you do not need.
4. In Packet Tracer, configure PAT on a router with `ip nat inside source list 1 interface g0/1 overload` and verify with `show ip nat translations`.

### 7. Quick summary
- NAT translates private to public addresses, PAT adds port mapping.
- Port forwarding exposes internal services, so use it carefully.
- NAT is not a security control on its own.

---

## Level 1 Checklist
Tick when you can explain or do each without notes.

- [ ] Explain LAN, WAN, switch, router and firewall
- [ ] Name all 7 OSI layers and give two protocols or devices for each
- [ ] Explain encapsulation using a web request
- [ ] Map OSI to the TCP/IP model
- [ ] Classify any IPv4 address as public, private or special
- [ ] Subnet a /24 into smaller networks by hand
- [ ] Explain how a switch builds its MAC table
- [ ] Read a routing table and explain the default gateway
- [ ] Recite 15+ common ports and their services
- [ ] Explain PAT with an example
- [ ] Identify one Layer 1 to 7 field in a Wireshark capture

## Practice Questions
1. At which OSI layer does a router work, and what address does it use?
2. What is the difference between a hub and a switch, and why does it matter for security?
3. How many usable hosts are in a /27? What is its subnet mask?
4. A host has IP `192.168.10.200/26`. What are its network and broadcast addresses?
5. What do the 5 values of a connection tuple identify?
6. Why is NAT not a replacement for a firewall?
7. What happens when TTL reaches 0?
8. Which port ranges are well-known, registered and ephemeral?

**Answers:** (1) Layer 3, IP address. (2) A hub repeats to all ports (easy sniffing), a switch forwards to the specific port. (3) 30 hosts, 255.255.255.224. (4) Network 192.168.10.192, broadcast 192.168.10.255. (5) One unique connection: protocol, source IP/port, destination IP/port. (6) It translates addresses but does not inspect or filter by policy. (7) The router drops the packet and sends an ICMP Time Exceeded message. (8) 0-1023, 1024-49151, 49152-65535.

## Real-Life Practice Platforms for This Level
- **Cisco Packet Tracer** (free with a Cisco Networking Academy account): build and test networks
- **TryHackMe:** "Pre Security" path (networking modules), "Intro to Networking"
- **Cisco NetAcad:** "Networking Basics" course
- **Subnetting practice:** subnettingpractice.com
