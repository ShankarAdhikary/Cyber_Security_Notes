# Level 7: Advanced Networking Topics

## Overview
Levels 1 to 6 gave you the full networking base for cyber security. This level adds the advanced topics that modern security work depends on: IPv6, cryptography and PKI, network authentication and Zero Trust, cloud networking, routing protocols (OSPF and BGP), modern web transport (HTTP/2, HTTP/3, QUIC), IoT and industrial (OT) protocols, logging and detection engineering, and anonymity networks.

**Prerequisites:** Levels 1 to 6. In particular, Level 2 (TLS, SSH, DNS), Level 3 (VPN, Kerberos, syslog), Level 4 (firewalls, segmentation) and Level 6 (MITM, tunnelling, forensics).

**Lab setup:** Linux VMs (Kali/Ubuntu), Wireshark, a free-tier cloud account (AWS, Azure or GCP) with a **billing alert set**, GNS3 or Packet Tracer for routing, and optionally Wazuh or Security Onion for detection practice. Keep attack labs on an isolated host-only network. **Never test against real industrial systems, real routers on the internet, or cloud accounts you do not own.**

## Table of Contents
1. [IPv6 in Depth](#1-ipv6-in-depth)
2. [Cryptography and PKI Basics](#2-cryptography-and-pki-basics)
3. [Network Authentication, Access Control and Zero Trust](#3-network-authentication-access-control-and-zero-trust)
4. [Cloud Networking](#4-cloud-networking)
5. [Routing Protocols: OSPF and BGP](#5-routing-protocols-ospf-and-bgp)
6. [HTTP/2, HTTP/3 and QUIC](#6-http2-http3-and-quic)
7. [IoT and OT Protocols](#7-iot-and-ot-protocols)
8. [Logging, SIEM and Detection Engineering](#8-logging-siem-and-detection-engineering)
9. [Anonymity Networks](#9-anonymity-networks)
10. [Level 7 Checklist](#level-7-checklist)
11. [Practice Questions](#practice-questions)
12. [What to Learn Next](#what-to-learn-next)

---

## 1. IPv6 in Depth

### 1. What it is
IPv6 is the successor to IPv4. It uses 128-bit addresses, removes broadcast, drops ARP in favour of NDP (Neighbor Discovery Protocol), and lets hosts configure themselves automatically (SLAAC). Almost every modern OS has IPv6 **enabled by default**, so even "IPv4-only" networks often carry IPv6 traffic that nobody monitors.

### 2. How it works
**Address types**

| Type | Prefix | Notes |
|------|--------|-------|
| Loopback | `::1` | Like 127.0.0.1 |
| Link-local | `fe80::/10` | Auto-assigned on every interface, works only on the local link |
| Unique local (ULA) | `fc00::/7` (used as `fd00::/8`) | Private-like addresses |
| Global unicast | `2000::/3` | Public, routable addresses |
| Multicast | `ff00::/8` | Replaces broadcast, for example `ff02::1` (all nodes), `ff02::2` (all routers) |

**Writing addresses:** eight groups of 16 bits in hex. Leading zeros can be dropped and one run of zero groups can be replaced by `::`. Example: `2001:0db8:0000:0000:0000:0000:0000:0001` becomes `2001:db8::1`.

**Structure of a typical address:** a 64-bit **network prefix** (given by the router or ISP) plus a 64-bit **interface ID**. A standard LAN subnet is a `/64`.

**NDP (Neighbor Discovery Protocol)** runs on ICMPv6 and does the jobs of ARP, ICMP router discovery and more:

| ICMPv6 type | Message | Purpose |
|-------------|---------|---------|
| 133 | Router Solicitation (RS) | Host asks "any routers here?" |
| 134 | Router Advertisement (RA) | Router announces prefix, gateway, flags |
| 135 | Neighbor Solicitation (NS) | Like an ARP request, also used for DAD |
| 136 | Neighbor Advertisement (NA) | Like an ARP reply |
| 137 | Redirect | Router points a host to a better next hop |

**SLAAC (Stateless Address Autoconfiguration)**
1. Host creates a link-local address (`fe80::...`).
2. **DAD (Duplicate Address Detection):** it sends an NS for its own tentative address. No reply means it is unique.
3. Host sends an RS (or waits for periodic RAs).
4. The router's **RA** carries the prefix. The host builds a global address from prefix + interface ID.
5. Interface ID can come from the MAC (EUI-64, leaks the MAC address), from a stable random value, or from temporary privacy addresses (RFC 4941).

**DHCPv6:** can give addresses (stateful) or just DNS and other settings (stateless). Uses UDP 546 (client) and 547 (server). The RA flags **M** (managed) and **O** (other config) tell hosts whether to use DHCPv6.

**Other points:** IPv6 headers are fixed size with optional **extension headers** (hop-by-hop, routing, fragment). Routers no longer fragment packets (only the sender does). IPsec support is part of the design. NAT is not needed for address conservation, though firewalls are still required.

**Transition mechanisms:** dual stack (both IPv4 and IPv6), tunnelling (6to4, Teredo, 6in4, ISATAP), translation (NAT64/DNS64).

### 3. Key facts
- ICMPv6 is **required** for IPv6 to work. Blocking all ICMPv6 breaks the network.
- No broadcast, no ARP. Multicast and NDP replace them.
- Hosts normally have **several IPv6 addresses** on one interface (link-local, global, temporary).
- A `/64` has 2^64 addresses, so blind scanning of a subnet is impractical. Attackers instead use multicast pings (`ff02::1`), neighbour caches, DNS, logs and predictable addressing.
- Tools differ: `ping -6`, `ip -6`, `traceroute6`, `nmap -6`.

### 4. Security issues and attacks
- **Rogue Router Advertisement:** an attacker sends fake RAs so hosts adopt the attacker as default gateway (MITM or DoS). It is the IPv6 equivalent of a rogue DHCP server.
- **NDP spoofing / poisoning:** fake NA messages, like ARP spoofing.
- **RA flooding:** a flood of RAs can overload hosts, especially some Windows versions in the past.
- **DAD denial of service:** an attacker answers every DAD probe so a host can never claim an address.
- **mitm6-style DHCPv6 attacks:** on IPv4-only networks that still have IPv6 enabled, a rogue DHCPv6 server can make victims use the attacker as their DNS server, enabling credential relay attacks in Windows domains.
- **Unmonitored dual stack:** IPv6 traffic bypasses IPv4-only firewalls, IDS and logging.
- **Extension header and fragmentation tricks** to evade filters.
- **Tunnels (Teredo, 6to4)** that bypass perimeter controls.
- **Privacy leaks:** EUI-64 addresses reveal the MAC address and can be used to track a device across networks.

### 5. Defenses
- **Decide deliberately:** either run IPv6 properly (firewall, monitoring, logging) or disable it on hosts that do not need it (and still watch for rogue RA/DHCPv6).
- **RA Guard** and **DHCPv6 Guard** on switches (only trusted ports may send RAs or DHCPv6 replies).
- **ND inspection / IPv6 Source Guard** on managed switches (the IPv6 counterpart of Dynamic ARP Inspection).
- **Firewall rules for IPv6** that mirror your IPv4 policy (`ip6tables`/`nft`, Windows Firewall). Allow the required ICMPv6 types only.
- **Block or monitor tunnelling protocols** (protocol 41, UDP 3544 for Teredo).
- Use **temporary/privacy addresses** and avoid EUI-64.
- Monitor with tools such as **NDPMon** and `ramond`. Include IPv6 in logging and asset inventory.
- Scan **your own** networks over IPv6 too.

### 6. Hands-on lab (real life)
1. **Inspect your own host:** `ip -6 addr` (Linux) or `ipconfig` (Windows). Identify your link-local, global and temporary addresses. Which one is used for outgoing connections?
2. **Test real IPv6:** visit `https://test-ipv6.com` and `curl -6 ifconfig.me`. Does your ISP give you IPv6?
3. **Find neighbours:** `ping -6 -c 3 ff02::1%eth0` (all nodes) then `ip -6 neigh`. Compare with `arp -a` for IPv4.
4. **Capture NDP:** in Wireshark filter `icmpv6`. Bring an interface down and up, then identify the RS, RA, NS (DAD) and NA packets. Open an RA and read the prefix, lifetime, and M/O flags.
5. **Build SLAAC in a lab:** on a router VM install `radvd` and configure it to advertise `2001:db8:1::/64` on the internal network. On a client VM watch it configure itself with SLAAC (`ip -6 addr`, `ip -6 route`).
6. **Rogue RA (isolated lab only):** start a second `radvd` (or the `ra6`/`fake_router6` tools from the THC-IPv6 toolkit) on an attacker VM on the host-only network. Observe in the victim's `ip -6 route` that a second default gateway appears. Then explain which switch feature (RA Guard) stops it. Do not do this on any shared network.
7. **Scan:** `nmap -6 -sn fe80::/64 -e eth0` is not practical, so try `nmap -6 -p 22,80,443 <target-IPv6>` and the NSE script `targets-ipv6-multicast-echo` on your own lab.
8. **Firewall parity:** with `ip6tables`, write default-deny rules for IPv6 that match your IPv4 `iptables` policy from Level 4. Verify with `nmap -6` from another VM.
9. **Check your exposure:** if your home network has IPv6, scan your **own** global IPv6 address from an outside host or a trusted online IPv6 port scanner, and confirm that your router firewall protects the devices.
10. **Packet Tracer:** configure IPv6 addressing and static routes between two LANs and verify with `ping` and `show ipv6 route`.

### 7. Quick summary
- IPv6 = 128-bit addresses, NDP replaces ARP, SLAAC lets hosts self-configure.
- Rogue RAs, NDP spoofing and unmonitored dual stack are the main risks.
- Either secure and monitor IPv6, or disable it deliberately. Never ignore it.

---

## 2. Cryptography and PKI Basics

### 1. What it is
Cryptography is the set of techniques that protect data's **confidentiality** (nobody else can read it), **integrity** (it was not changed), **authentication** (it comes from who it claims) and **non-repudiation** (the sender cannot deny sending). **PKI (Public Key Infrastructure)** is the system of certificates and authorities that lets strangers trust each other's public keys. TLS, SSH, VPNs, email security, code signing and password storage all depend on these ideas.

### 2. How it works
**Encoding vs hashing vs encryption**

| | Reversible? | Needs a key? | Purpose | Examples |
|---|-------------|--------------|---------|----------|
| Encoding | Yes | No | Format conversion | Base64, URL encoding |
| Hashing | No (one-way) | No | Integrity, fingerprints, password storage | SHA-256, bcrypt |
| Encryption | Yes (with key) | Yes | Confidentiality | AES, RSA |

**Hashing**
- A hash function turns any input into a fixed-size digest. Good hashes are **deterministic**, **fast to compute**, **infeasible to reverse**, and **collision resistant** (hard to find two inputs with the same output). A tiny change gives a completely different output (avalanche effect).
- Safe for integrity today: **SHA-256, SHA-384, SHA-512, SHA-3, BLAKE2/3**. **Broken for security:** MD5 and SHA-1.
- **HMAC** combines a hash with a secret key to prove both integrity and authenticity of a message.
- **Password storage:** never plain hashes. Use a slow, salted algorithm: **Argon2, bcrypt, scrypt or PBKDF2**. A **salt** is a random per-user value that defeats rainbow tables. A **pepper** is a secret value kept outside the database.

**Symmetric encryption (one shared key)**
- Fast, used for bulk data. Examples: **AES** (128/256-bit), **ChaCha20**.
- **Modes matter:** use authenticated modes such as **AES-GCM** or ChaCha20-Poly1305. **ECB** mode leaks patterns (the famous "ECB penguin") and should never be used. CBC needs careful padding and integrity handling.
- Problem: how do two parties share the key safely? That is why asymmetric crypto exists.

**Asymmetric encryption (public/private key pair)**
- **RSA** (2048-bit minimum, 3072+ preferred), **ECC** (elliptic curves, such as P-256 and Curve25519, smaller keys with the same strength).
- What the public key encrypts only the private key can decrypt. Slower, so it is used for **key exchange and signatures**, not for bulk data.
- **Diffie-Hellman (DH/ECDH)** lets two parties agree on a shared secret over an open channel. **Ephemeral** versions (DHE/ECDHE) give **forward secrecy**: stealing a server's long-term key later does not reveal past recorded sessions.
- **Hybrid approach (what TLS does):** asymmetric crypto for authentication and key exchange, then symmetric crypto (AES-GCM) for the data.

**Digital signatures**
1. Sender hashes the message.
2. Sender signs the hash with the **private key** (RSA, ECDSA, Ed25519).
3. Receiver verifies the signature with the sender's **public key** and compares the hash.
Gives integrity, authentication and non-repudiation. Used in certificates, software updates, signed emails, Git commits, DNSSEC.

**PKI and certificates**
- An **X.509 certificate** binds an identity (domain name or person) to a public key, signed by a **Certificate Authority (CA)**.
- Fields to know: Subject, Subject Alternative Names (SAN), Issuer, validity dates, public key, key usage, signature algorithm, serial number.
- **Chain of trust:** leaf (server) certificate, signed by an intermediate CA, signed by a **root CA** that is pre-installed in your OS/browser trust store.
- **Life cycle:** generate a key pair, create a **CSR** (Certificate Signing Request), the CA validates you and issues the certificate, you install it, renew it, and **revoke** it if the key is compromised.
- **Revocation:** CRL (a list of revoked certificates), OCSP (online status query), OCSP stapling (server includes a signed status).
- **Certificate Transparency (CT):** public append-only logs of all issued certificates, letting domain owners detect mis-issuance (see crt.sh).
- **Validation levels:** DV (domain control only), OV, EV. Let's Encrypt provides free automated DV certificates (ACME protocol).
- **Alternative trust model:** SSH uses host keys and `authorized_keys` (trust on first use), PGP uses a web of trust.

**Where it shows up in what you already learned**
- **TLS (Level 2):** certificate validation, ECDHE key exchange, AES-GCM data encryption.
- **SSH (Level 2):** host keys, public key authentication.
- **VPNs (Level 3):** IKE, WireGuard keys, certificate-based OpenVPN.
- **Kerberos, WPA3-SAE, DNSSEC, email (DKIM, S/MIME)** all rely on the same primitives.

### 3. Key facts
- **Never invent your own cryptography.** Use well-reviewed libraries and standard algorithms.
- Security depends on the **key**, not on secrecy of the algorithm (Kerckhoffs's principle).
- Key length is not everything: randomness, key storage and protocol design matter at least as much.
- Quantum computers threaten RSA/ECC in future. NIST has standardised post-quantum algorithms (ML-KEM, ML-DSA, SLH-DSA), and "harvest now, decrypt later" is a reason to start planning migration.
- Rule of thumb: **hash** to check integrity, **encrypt** to hide, **sign** to prove origin, **HMAC** to check integrity with a shared secret.

### 4. Security issues and attacks
- **Weak or obsolete algorithms:** MD5, SHA-1, DES, 3DES, RC4, 1024-bit RSA, export-grade ciphers.
- **Password cracking:** brute force, dictionary, rainbow tables (defeated by salts), GPU cracking of fast hashes.
- **Poor randomness:** predictable keys or nonces (for example reused nonces in GCM are catastrophic).
- **Key theft and mismanagement:** private keys in Git repos, in container images, or world-readable on servers.
- **Padding oracle and downgrade attacks** (POODLE, BEAST, DROWN, FREAK, Logjam).
- **Implementation bugs** (Heartbleed).
- **Rogue or compromised CAs and mis-issued certificates** (DigiNotar 2011), certificates stolen from vendors.
- **Users ignoring certificate warnings**, and malware installing a rogue root CA for MITM.
- **Hash length-extension attacks** when someone builds a MAC as `hash(secret || message)` instead of using HMAC.

### 5. Defenses
Use TLS 1.2+ (prefer 1.3) with ECDHE and AEAD ciphers, store passwords with Argon2/bcrypt plus salt, store keys in an HSM or a secrets manager (never in source control), rotate keys and certificates, automate renewal, monitor Certificate Transparency for your domains, pin or restrict CAs where appropriate (CAA DNS records), enforce HSTS, and keep crypto libraries patched.

### 6. Hands-on lab (real life)
All commands work on Linux/macOS (OpenSSL is also available for Windows).
1. **Hashing basics:** `echo -n "hello" | sha256sum`, then `echo -n "hellp" | sha256sum`. Compare the two outputs (avalanche effect). Also try `md5sum` and discuss why it is no longer safe.
2. **File integrity:** hash a file, change one byte, hash again. Download a Linux ISO and verify its published SHA-256 checksum (`sha256sum -c`).
3. **HMAC:** `echo -n "message" | openssl dgst -sha256 -hmac "secretkey"`.
4. **Password hashing demo:** generate a bcrypt/salted hash (`python3 -c "import bcrypt; print(bcrypt.hashpw(b'pass', bcrypt.gensalt()))"`, after `pip install bcrypt`). Run it twice and notice the different output because of the salt.
5. **Crack your own test hashes (lab only):** create 3 MD5 hashes of simple passwords, then crack them with `john` or `hashcat` and a small wordlist. Repeat with bcrypt and compare how long it takes. This shows why slow hashes matter.
6. **Symmetric encryption:** `openssl enc -aes-256-cbc -pbkdf2 -salt -in secret.txt -out secret.enc` then decrypt with `-d`. Try encrypting with a wrong password.
7. **Asymmetric keys:** `openssl genrsa -out private.pem 2048` then `openssl rsa -in private.pem -pubout -out public.pem`. Encrypt a small file with the public key and decrypt with the private key. Also try `ssh-keygen -t ed25519`.
8. **Sign and verify:** `openssl dgst -sha256 -sign private.pem -out file.sig file.txt` and `openssl dgst -sha256 -verify public.pem -signature file.sig file.txt`. Change one character in the file and verify again.
9. **Build your own mini CA:** create a root CA key and certificate (`openssl req -x509 -new -nodes -key ca.key -sha256 -days 365 -out ca.crt`), create a server key and CSR, sign the CSR with your CA, then run `openssl verify -CAfile ca.crt server.crt`. Serve a page with Nginx using that certificate and import `ca.crt` into your lab browser to see the padlock turn green. This is the best way to understand chains of trust.
10. **Inspect a real certificate:** `openssl s_client -connect example.com:443 -servername example.com </dev/null | openssl x509 -noout -text`. Identify the issuer, SANs, validity, key type and signature algorithm. Then search the domain on **crt.sh**.
11. **Check forward secrecy:** in Wireshark look at the TLS Server Hello cipher suite. Is it ECDHE? Test a server's setup with `nmap --script ssl-enum-ciphers -p 443 <your-lab-host>`.
12. **CAA record:** `dig example.com CAA` to see which CAs are allowed to issue certificates for a domain.
13. **Practice:** TryHackMe "Cryptography" and "Public Key Cryptography Basics" rooms, **CryptoHack** (free challenges), and the Cryptopals challenges for deeper practice.

### 7. Quick summary
- Hash for integrity, symmetric for speed, asymmetric for key exchange and signatures.
- PKI turns public keys into trusted identities through certificates and CAs.
- Most real-world failures come from weak algorithms, bad key handling and bad passwords, not from broken maths.

---

## 3. Network Authentication, Access Control and Zero Trust

### 1. What it is
Controlling **who and what** may join and use the network. This includes port-based authentication (**802.1X**), central authentication servers (**RADIUS**, TACACS+), **NAC (Network Access Control)** that checks the device as well as the user, and **Zero Trust**, a security model that stops treating the internal network as automatically trusted.

### 2. How it works
**AAA:** Authentication (who are you?), Authorization (what may you do?), Accounting (what did you do?).

**802.1X port-based access control**
Three roles:
- **Supplicant:** the client device (laptop, phone, printer).
- **Authenticator:** the switch port or Wi-Fi access point. It blocks all traffic except authentication until the client is approved.
- **Authentication server:** usually a **RADIUS** server (FreeRADIUS, Microsoft NPS, Cisco ISE, Aruba ClearPass), often linked to Active Directory/LDAP.

Flow (simplified):
1. Device connects. The port allows only **EAPOL** (EAP over LAN) frames.
2. The authenticator asks for identity and relays **EAP** messages to the RADIUS server inside RADIUS packets.
3. The server and client run an EAP method.
4. On success the server sends **Access-Accept**, optionally with attributes (for example assign VLAN 20, apply an ACL). The port opens.
5. On failure: **Access-Reject**, or placement into a restricted guest/remediation VLAN.

**Common EAP methods**

| Method | How it authenticates | Notes |
|--------|---------------------|-------|
| **EAP-TLS** | Client and server certificates | Strongest, needs PKI on every device |
| **PEAP (MSCHAPv2)** | Server certificate + username/password inside a TLS tunnel | Very common, risk if clients do not validate the server certificate |
| **EAP-TTLS** | Server certificate + inner method | Similar to PEAP |
| EAP-FAST, EAP-MD5 | Various | EAP-MD5 is weak |

**RADIUS basics:** UDP 1812 (authentication) and 1813 (accounting) (older 1645/1646). Client and server share a **shared secret**. Only the password attribute is encrypted by the legacy RADIUS protocol, so RADIUS traffic should travel on trusted segments or inside IPsec/TLS (RadSec). **TACACS+** (TCP 49) is the usual choice for **device administration** (logins to routers and switches) because it encrypts the whole payload and separates authentication from authorization.

**WPA2/WPA3-Enterprise** is simply 802.1X applied to Wi-Fi.

**NAC (Network Access Control)**
Adds device-aware decisions on top of 802.1X:
- **Profiling:** what kind of device is this (printer, phone, laptop)?
- **Posture assessment:** is the OS patched, antivirus running, disk encrypted?
- **Policy:** place the device into the right VLAN or ACL (employee, contractor, guest, quarantine).
- **MAB (MAC Authentication Bypass):** for devices that cannot do 802.1X (printers, cameras), the switch uses the MAC address as the identity. Convenient but weak.
- **Dynamic VLAN assignment** and **downloadable ACLs** applied by RADIUS attributes.

**Zero Trust**
- Principle: **"never trust, always verify"**. Being on the internal network does not grant trust. NIST SP 800-207 describes the architecture.
- **Core ideas:**
  1. Verify **identity** (strong MFA) for every user and service.
  2. Verify **device health** (managed, patched, compliant).
  3. Grant **least privilege**, per application or resource, not per network.
  4. **Micro-segmentation:** small zones and per-workload policies to stop lateral movement.
  5. **Continuous verification** and risk-based decisions (location, behaviour, time).
  6. **Assume breach:** log everything and limit the blast radius.
- **Components (NIST model):** Policy Engine, Policy Administrator and Policy Enforcement Points that sit in front of resources.
- **ZTNA (Zero Trust Network Access)** replaces broad VPN access with per-application access brokered through an identity-aware proxy (examples: Cloudflare Access, Zscaler, Google BeyondCorp, Tailscale-style overlays).
- **Zero Trust is a strategy, not a product.** It combines identity, device management, segmentation, encryption and monitoring.

### 3. Key facts
- 802.1X protects the **edge** (the port or SSID). Zero Trust extends the idea to every connection.
- Certificate-based authentication (EAP-TLS) resists phishing and password attacks much better than passwords.
- The shared secret between switch and RADIUS server must be long and unique per device.
- Printers and IoT often cannot do 802.1X, so segment them heavily.

### 4. Security issues and attacks
- **Rogue or evil twin access points** that impersonate a PEAP network and collect inner credentials when clients do not verify the server certificate.
- **MAB bypass:** spoofing the MAC of an authorised printer or phone.
- **802.1X bypass** using a transparent bridge or hub placed between an authenticated device and the switch.
- **Weak RADIUS shared secrets**, which can be cracked offline from a captured packet.
- **RADIUS and EAP-MD5 downgrade** and use of unencrypted RADIUS over untrusted links.
- **Credential theft** (password spraying against the directory, MFA fatigue attacks).
- **Misconfigured NAC policies** that leave a "temporary allow-all" in place.
- **Zero Trust failures:** a single over-privileged service account, stale device inventories, or a ZTNA broker that becomes a single point of failure.

### 5. Defenses
Use EAP-TLS where possible, enforce **server certificate validation** on clients (deploy the trusted CA by policy), long unique RADIUS secrets, RadSec or IPsec for RADIUS transport, port security plus 802.1X, MACsec where available, strong MFA (phishing-resistant FIDO2 where possible), dedicated guest and quarantine VLANs, regular review of NAC rules, device posture checks, micro-segmentation with default deny, and central logging of every authentication event.

### 6. Hands-on lab (real life)
1. **Install FreeRADIUS** on a Linux VM: `sudo apt install freeradius freeradius-utils`. Edit `/etc/freeradius/3.0/users` and add a test user (`alice Cleartext-Password := "Passw0rd!"`). Run in debug mode: `sudo freeradius -X`.
2. **Test it:** `radtest alice 'Passw0rd!' 127.0.0.1 0 testing123`. Read the Access-Accept in the debug output. Try a wrong password and watch the Access-Reject.
3. **Capture RADIUS:** in Wireshark filter `radius` and inspect the attributes. Notice the User-Password is hidden but everything else is visible.
4. **Add a VLAN attribute:** in `users`, give Alice `Tunnel-Type = VLAN, Tunnel-Medium-Type = IEEE-802, Tunnel-Private-Group-Id = "20"` and verify the reply contains it.
5. **Packet Tracer / GNS3:** configure a switch for **802.1X** (or AAA with RADIUS for device logins) pointing to your FreeRADIUS server. Connect a PC that supports 802.1X and see the port go from blocked to authorised.
6. **Optional Wi-Fi Enterprise lab:** use `hostapd` with your FreeRADIUS server and a USB Wi-Fi adapter, or the FreeRADIUS "eapol_test" utility to test EAP methods without hardware. Create certificates with the scripts in `/etc/freeradius/3.0/certs`.
7. **Observe the certificate-validation risk:** on a test client, disable "validate server certificate" for PEAP in your lab and discuss what an evil twin could capture. Re-enable validation. Do not try this on real networks.
8. **MAB awareness:** list the devices in your home or lab that could never run 802.1X (printer, TV, IoT) and decide which VLAN each should live in.
9. **Zero Trust design exercise:** take a fictional company (200 staff, 3 internal web apps, a database, remote workers, contractors). Draw how users reach each app under a VPN-based model, then under a ZTNA model. List which identity, device and policy checks happen at each step.
10. **Try ZTNA free tier:** set up Cloudflare Access (free for small teams) or Tailscale on your own devices to put a lab web app behind identity-based access and review the access logs.
11. **Read:** NIST SP 800-207 (Zero Trust Architecture) and CISA's Zero Trust Maturity Model.

### 7. Quick summary
- 802.1X lets only authenticated devices use a port or SSID. RADIUS is the server behind it.
- NAC adds device checks and dynamic policy. MAB is convenient but weak.
- Zero Trust = verify identity and device for every request, least privilege, micro-segmentation, assume breach.

---

## 4. Cloud Networking

### 1. What it is
In the cloud, the "network" is software-defined: you create virtual networks, subnets, routes and firewalls with APIs and consoles instead of cables and switches. The three big providers use slightly different names but follow the same ideas:

| Concept | AWS | Azure | GCP |
|---------|-----|-------|-----|
| Virtual network | **VPC** | **VNet** | **VPC** (global) |
| Subnet | Subnet (per AZ) | Subnet | Subnet (regional) |
| Instance firewall | **Security Group** (stateful, allow rules only) | **NSG** (Network Security Group) | **VPC firewall rules** (stateful) |
| Subnet firewall | **Network ACL** (stateless) | NSG on subnet | Hierarchical firewall policies |
| Internet access | Internet Gateway, NAT Gateway | Public IP, NAT Gateway | External IP, Cloud NAT |
| Private connectivity | VPC Peering, Transit Gateway, PrivateLink | VNet Peering, Private Link | VPC Peering, Private Service Connect |
| Hybrid link | Site-to-Site VPN, Direct Connect | VPN Gateway, ExpressRoute | Cloud VPN, Interconnect |
| Flow logging | VPC Flow Logs | NSG Flow Logs / VNet flow logs | VPC Flow Logs |
| Threat detection | GuardDuty | Defender for Cloud | Security Command Center |

### 2. How it works
**Typical VPC layout (AWS wording)**
```
Internet
   |
[Internet Gateway]
   |
VPC 10.0.0.0/16
 +-- Public subnet 10.0.1.0/24   (route to IGW)   -> load balancer, bastion/jump host
 +-- Private subnet 10.0.2.0/24  (route to NAT)   -> app servers
 +-- Isolated subnet 10.0.3.0/24 (no internet)    -> databases
```
- A subnet is "public" because its **route table** sends `0.0.0.0/0` to an Internet Gateway **and** its instances have public IPs. A "private" subnet reaches the internet only through a **NAT gateway**.
- **Security groups** attach to instances/network interfaces, are **stateful**, have **allow rules only**, and can reference other security groups (for example "database allows port 3306 from the app security group").
- **Network ACLs** attach to subnets, are **stateless** (you must allow return traffic) and support allow and deny rules.
- **VPC peering** and **transit gateways** connect VPCs. Peering is not transitive by default.
- **VPC endpoints / Private Link** reach cloud services (S3, databases) without crossing the public internet.
- **Metadata service** (`169.254.169.254`) gives instances their credentials and configuration. Version 2 (IMDSv2) requires a session token to reduce SSRF abuse.
- **Shared responsibility model:** the provider secures the underlying infrastructure. **You** secure your configuration, identities, data and network rules. Most cloud breaches are configuration mistakes.
- **Infrastructure as Code (IaC):** Terraform, CloudFormation, Bicep. Network setups are defined in code and reviewed like software.

### 3. Key facts
- Everything is an API call, so **IAM (identity and access management)** is as important as the network rules. A leaked access key can bypass every network control.
- Default deny inbound is the default for security groups. Danger comes from rules you add.
- Cloud networks are **flat by default inside a VPC** unless you segment with subnets and security groups.
- Logs are **not always on**. Flow logs, audit logs (CloudTrail, Azure Activity Log, GCP Audit Logs) must be enabled and stored safely.
- Costs: NAT gateways, data transfer, load balancers and public IPs cost money. Set billing alerts before you experiment.

### 4. Security issues and attacks (common misconfigurations)
- **Management ports open to the world:** SSH (22) or RDP (3389) allowed from `0.0.0.0/0`.
- **Public databases and services:** MySQL, PostgreSQL, MongoDB, Redis, Elasticsearch exposed to the internet without authentication.
- **Public storage:** S3 buckets, Azure Blob containers and GCS buckets set to public, or listable by anyone.
- **Overly permissive security groups/firewall rules:** "allow all" between tiers, wide CIDR ranges.
- **SSRF to the metadata service:** a vulnerable web app is tricked into fetching `169.254.169.254` and leaks cloud credentials (IMDSv1 is especially risky).
- **Overly broad IAM roles and long-lived access keys** committed to Git repositories.
- **Unencrypted data and traffic**, missing TLS between internal services.
- **No logging or monitoring**, so breaches go unnoticed.
- **Default VPC/network use** and forgotten test resources ("shadow IT").
- **Exposed container/Kubernetes dashboards and APIs.**
- **Dangling DNS records pointing to deleted cloud resources** (subdomain takeover).
- **Cross-tenant and peering mistakes:** accidentally routing sensitive networks together.

### 5. Defenses
- **Least privilege:** open only required ports to only required sources. Prefer security-group-to-security-group references over wide CIDR ranges.
- **No direct admin access from the internet:** use a VPN, a bastion with MFA, or managed access (AWS Systems Manager Session Manager, Azure Bastion, GCP IAP).
- **Private by default:** databases and internal services in private subnets with no public IP. Use private endpoints.
- **Block public storage** at the account level (S3 Block Public Access, Azure and GCP equivalents).
- **Enforce IMDSv2**, restrict instance roles, and validate outbound URLs in applications.
- **Turn on logging everywhere:** flow logs, audit logs, DNS logs. Send them to a separate protected account.
- **Use managed detection:** GuardDuty, Defender for Cloud, Security Command Center.
- **Scan configurations continuously** (CSPM tools such as Prowler, ScoutSuite, Wiz, Prisma) and scan IaC before deployment (Checkov, tfsec, Terrascan).
- **Segment** with separate VPCs/accounts/subscriptions for production, development and shared services.
- **Protect secrets** with a secrets manager and rotate keys. Never commit keys.
- **Tag and inventory** resources, and delete unused ones.

### 6. Hands-on lab (real life)
**Use a free-tier account you own. First set a billing alarm or budget alert, and delete everything when done.**
1. **Build a 3-tier network (AWS example, Azure and GCP are similar):** create a VPC `10.0.0.0/16`, a public subnet and a private subnet, an Internet Gateway, route tables, and (optionally, since it costs money) a NAT gateway. Draw the diagram in your notes.
2. **Security groups:** launch one small instance in the public subnet. Allow SSH only from **your own IP** (`/32`), not `0.0.0.0/0`. Launch a second instance in the private subnet that accepts SSH only from the first instance's security group. Test connectivity both ways.
3. **Prove the difference:** temporarily open port 22 to `0.0.0.0/0` on a throwaway instance and watch `/var/log/auth.log` for brute-force attempts from the internet within minutes or hours. Then close it. This shows why exposed management ports are dangerous.
4. **Network ACL practice:** add a stateless NACL rule set and notice that you must allow return (ephemeral) ports.
5. **Enable VPC Flow Logs** to CloudWatch or S3 and read a few records. Find the `REJECT` entries from internet scanners.
6. **Use the CLI:** `aws ec2 describe-security-groups --query "SecurityGroups[?IpPermissions[?IpRanges[?CidrIp=='0.0.0.0/0']]].GroupId"` (or the Azure `az network nsg list` and GCP `gcloud compute firewall-rules list` equivalents) to find rules open to the world.
7. **Run a scanner on your own account:** install **Prowler** (`pip install prowler`) or **ScoutSuite** and review the findings. Fix three.
8. **Check storage exposure:** create an S3 bucket, confirm Block Public Access is on, and try to access an object anonymously from a browser to confirm it is blocked.
9. **Metadata service:** on a lab instance run `curl http://169.254.169.254/latest/meta-data/` (IMDSv1) versus using the IMDSv2 token flow. Then enforce IMDSv2 (`aws ec2 modify-instance-metadata-options --http-tokens required`).
10. **IaC scan:** write a small Terraform file with an intentionally open security group and scan it with **Checkov**. Fix it until it passes.
11. **Legal cloud hacking practice:** **flaws.cloud** and **flaws2.cloud** (free), **CloudGoat** and **TerraGoat** (deploy into your own sandbox account), and TryHackMe cloud rooms.
12. **Clean up:** terminate instances, delete NAT gateways, elastic IPs, buckets and VPCs to avoid charges.

### 7. Quick summary
- Cloud networks are software-defined: VPC/VNet, subnets, route tables, security groups or NSGs.
- Most cloud breaches are misconfigurations: open ports, public storage, over-broad IAM, exposed metadata.
- Turn on logs, keep services private, scan configurations continuously.

---

## 5. Routing Protocols: OSPF and BGP

### 1. What it is
Routing protocols let routers share information about networks and choose paths automatically.
- **OSPF (Open Shortest Path First)** is an **interior gateway protocol (IGP)** used **inside** one organisation or autonomous system.
- **BGP (Border Gateway Protocol)** is the **exterior gateway protocol** that glues the internet together **between** autonomous systems (ISPs, cloud providers, large organisations). BGP is the "postal routing system" of the internet.

### 2. How it works
**Autonomous System (AS):** a network under one administrative policy with a unique **ASN** (for example AS15169 is Google).

**OSPF (link-state)**
- Every router learns the **whole topology** of its area by exchanging **LSAs (Link-State Advertisements)**, then runs **Dijkstra's shortest path first** algorithm to build routes. Metric = **cost** (based on bandwidth).
- **Areas:** the network is split into areas, all connected to **Area 0 (the backbone)**, to limit the amount of information each router handles.
- **Neighbour formation:** routers send **Hello** packets (multicast `224.0.0.5` for all OSPF routers, `224.0.0.6` for DR/BDR). They must match on area ID, hello/dead timers, subnet and authentication to become neighbours. Neighbour states: Down, Init, 2-Way, ExStart, Exchange, Loading, Full.
- **DR/BDR (Designated Router / Backup):** on multi-access networks (Ethernet) one router collects and distributes updates to reduce chatter.
- Runs directly over IP (protocol number **89**), not TCP or UDP.
- Authentication: plain text (weak), MD5, or **HMAC-SHA** (OSPFv2 with SHA, or IPsec for OSPFv3).

**BGP (path-vector)**
- Runs over **TCP port 179** between configured **peers**.
- **eBGP** (between different ASes) and **iBGP** (inside one AS).
- Routers announce **prefixes** (for example `203.0.113.0/24`) together with the **AS_PATH**, the list of ASes the announcement has passed through. AS_PATH also prevents loops (a router rejects an announcement that already contains its own ASN).
- **Best-path selection** uses attributes such as highest **LOCAL_PREF**, shortest **AS_PATH**, lowest **MED**, and others. Policy (business relationships: customer, provider, peer) matters more than shortest distance.
- Routing follows the **longest prefix match**: a more specific prefix (`/24`) wins over a less specific one (`/16`).
- The global routing table has roughly a million IPv4 prefixes.
- BGP was designed on **trust**: by default a router believes what its neighbours announce.

**BGP hijacks and leaks**
- **Prefix (origin) hijack:** an AS announces a prefix it does not own. Traffic for that prefix flows toward the attacker (or toward a black hole).
- **More-specific hijack (sub-prefix):** the attacker announces smaller prefixes (for example two `/24`s inside the victim's `/23`). Because of longest-prefix-match, the whole internet prefers the attacker's routes, even though the victim's route is still being announced.
- **AS-path manipulation / forged origin:** the attacker adds the legitimate ASN at the end of the path so that the origin looks right.
- **Route leak:** an AS accidentally re-announces routes it learned from one provider or peer to another, attracting traffic it cannot handle (often a misconfiguration, not an attack).
- **Famous incidents:**
  - **2008, Pakistan Telecom and YouTube:** Pakistan tried to block YouTube locally by announcing a more-specific route. It leaked to the world and took YouTube offline for a couple of hours.
  - **2018, Amazon Route 53 / MyEtherWallet:** attackers hijacked address space used by DNS servers to redirect users of a cryptocurrency wallet to a fake site.
  - **2021, Facebook outage:** not a hijack, but a configuration change withdrew its BGP routes (and DNS servers became unreachable), showing how critical BGP is.
- **Why it matters for security:** hijacks enable traffic interception, censorship, denial of service, and attacks on DNS and certificate issuance.

**Defences built into the ecosystem**
- **RPKI (Resource Public Key Infrastructure):** the owner of an IP block publishes a signed **ROA (Route Origin Authorization)** stating which ASN may originate the prefix and the maximum prefix length. Networks perform **ROV (Route Origin Validation)** and reject invalid routes (an origin hijack or a too-specific announcement).
- **IRR filtering, prefix limits and max-prefix settings**, **bogon filtering**.
- **MANRS (Mutually Agreed Norms for Routing Security)** best-practice programme.
- **BGPsec/ASPA** (path validation efforts, still being deployed).
- **Monitoring services:** RIPE RIS, RouteViews, BGPStream, bgp.he.net, Cloudflare Radar, ThousandEyes.

### 3. Key facts
- OSPF finds the best path **inside** a network by cost. BGP chooses paths **between** networks by **policy**.
- BGP is the reason the internet can be rerouted by a single misconfiguration or malicious announcement.
- An ROA only protects the **origin**. It does not (alone) validate the whole path.
- Within enterprises, routing protocol authentication is often left off, which is a risk.

### 4. Security issues and attacks
- **Rogue router / adjacency forging (OSPF):** an attacker who can send OSPF packets on a segment can form an adjacency (if authentication is off) and inject false LSAs to redirect traffic (MITM or black hole).
- **OSPF LSA flooding or sequence-number attacks** causing instability.
- **BGP hijacks and route leaks** (see above).
- **BGP session attacks:** TCP reset or spoofing against unauthenticated sessions. Mitigated by TCP-AO/MD5 and TTL security (GTSM).
- **Route flapping and table-exhaustion attacks** (too many prefixes).
- **Black holing abuse**, for example misused RTBH communities.
- **Misconfigured filters** causing accidental leaks.

### 5. Defenses
- **OSPF:** enable authentication (HMAC-SHA), mark user-facing interfaces **passive** so they send no Hellos, filter or restrict who can become neighbours, protect router management, monitor adjacencies and LSA changes.
- **BGP:** publish **ROAs** for your prefixes and **perform ROV** on received routes, apply prefix filters and **max-prefix limits** on every neighbour, filter bogons and your own prefixes from outside, use **TCP-AO** or MD5 on sessions and **GTSM** (TTL 255), follow **MANRS**, and monitor for unexpected announcements of your prefixes.
- Treat router control planes as critical infrastructure: restrict management access, log changes and use secure configuration baselines.

### 6. Hands-on lab (real life)
1. **OSPF in Packet Tracer or GNS3:** build 3 routers in a triangle with LANs behind them. Configure OSPF (`router ospf 1`, `network ... area 0`). Verify with `show ip ospf neighbor`, `show ip route ospf`, `show ip ospf database`. Shut one link and watch the path change.
2. **Break and fix:** set a mismatched hello timer or area ID on one router and see why the adjacency does not form (`debug ip ospf adj` in a lab). Then configure **OSPF authentication** (`ip ospf authentication message-digest`) and see what happens when keys differ.
3. **Capture OSPF:** in Wireshark filter `ospf` and inspect a Hello and an LSA update.
4. **Use real routing software:** install **FRRouting (FRR)** on Linux VMs or use **Containerlab** to build a small network and run OSPF and BGP between Linux routers.
5. **BGP lab with two or three ASes:** configure eBGP between AS 65001 and AS 65002 (private ASNs, `neighbor x.x.x.x remote-as ...`, `network ...`). Verify with `show ip bgp`, `show ip bgp summary`. Look at the AS_PATH of learned routes.
6. **Simulate a hijack in your lab only:** in your own virtual AS topology, announce a more-specific prefix from the "attacker" AS and watch the routing tables of other ASes switch to it (longest prefix match). Then add a **prefix filter** (or simulate RPKI validation) to stop it. This isolated simulation shows the mechanism without touching the real internet.
7. **Explore the real internet:** look up your ISP's ASN on **bgp.he.net** or **RIPEstat**. Which networks does it peer with? Which prefixes does it announce?
8. **Check RPKI adoption:** visit **isbgpsafeyet.com** to see whether your ISP filters invalid routes, and look up your own or a famous prefix's ROA on a public RPKI validator or RIPEstat.
9. **Case study write-up:** pick the 2008 Pakistan/YouTube incident or the 2018 Route 53 hijack, and write 10 lines: what was announced, why it spread, how it was fixed, which control would have prevented it.
10. **Read:** MANRS documentation and Cloudflare's learning pages on BGP and RPKI.

### 7. Quick summary
- OSPF is the link-state IGP inside a network. BGP is the path-vector protocol between ASes.
- BGP trusts announcements by default. Hijacks and leaks exploit that, especially more-specific prefixes.
- RPKI/ROV, prefix filters, max-prefix limits and monitoring are the main defences. Enable authentication on OSPF.

---

## 6. HTTP/2, HTTP/3 and QUIC

### 1. What it is
Modern web traffic no longer uses plain text HTTP/1.1 only:
- **HTTP/2** (2015) keeps HTTP's meaning (methods, headers, status codes) but changes how it is sent: binary framing, multiplexing, header compression. Still runs over **TCP**, almost always with TLS.
- **HTTP/3** (2022) maps HTTP onto **QUIC**, a transport protocol built on **UDP** that integrates TLS 1.3, reducing latency and fixing TCP's head-of-line blocking.

### 2. How it works
**HTTP/1.1 vs HTTP/2 vs HTTP/3**

| Feature | HTTP/1.1 | HTTP/2 | HTTP/3 |
|---------|----------|--------|--------|
| Format | Text | Binary frames | Binary frames |
| Transport | TCP | TCP | **QUIC over UDP** |
| Encryption | Optional (HTTPS) | Browsers require TLS | **Always (TLS 1.3 built in)** |
| Multiplexing | No (many connections) | Yes (streams on one connection) | Yes (independent streams) |
| Header compression | None | HPACK | QPACK |
| Port | 80 / 443 | 443 | **UDP 443** |

**HTTP/2 details**
- One TCP connection carries many **streams** (requests and responses) interleaved as **frames** (HEADERS, DATA, SETTINGS, RST_STREAM, GOAWAY...).
- **HPACK** compresses repeated headers.
- Negotiated during the TLS handshake using **ALPN** (`h2`).
- Problem: **TCP head-of-line blocking.** One lost packet stalls all streams because TCP delivers data in order.

**QUIC and HTTP/3 details**
- QUIC runs over UDP, implements its own reliability, congestion control, stream multiplexing and **encryption (TLS 1.3 handshake inside QUIC)**.
- **Faster setup:** 1 round trip for new connections and **0-RTT** for resumed ones (risk: replayable data).
- **Independent streams:** a lost packet only delays its own stream.
- **Connection migration:** a **Connection ID** lets a session survive an IP change (Wi-Fi to mobile).
- **Mostly encrypted headers:** unlike TCP, even transport details are hidden from the network.
- **Discovery:** browsers usually start with HTTP/2 over TCP, learn from an `Alt-Svc: h3=":443"` header (or an HTTPS DNS record) that HTTP/3 is available, then switch.
- **Fallback:** if UDP 443 is blocked, browsers fall back to HTTP/2 or HTTP/1.1.

**Impact on inspection (why this matters to security work)**
- Firewalls, proxies and IDS/IPS built to inspect **TCP 443** may not see **UDP 443** QUIC traffic at all, creating a blind spot or bypass.
- In QUIC, nearly everything after the first packets is encrypted. The **Initial packet** is protected with keys derivable from the connection's visible ID, so some devices can still read the **SNI** (server name) from it, but this is getting harder with **Encrypted Client Hello (ECH)**.
- Wireshark can decode QUIC and HTTP/3 only if you supply TLS secrets (via `SSLKEYLOGFILE`).
- Organisations that inspect web traffic often **block UDP 443** at the perimeter to force browsers back to TCP (HTTP/2) so existing inspection still works, or they move inspection to endpoints and secure web gateways that support QUIC.

### 3. Key facts
- The **semantics are the same** (methods, status codes, cookies, headers), so web vulnerabilities like XSS and SQL injection are unchanged. What changes is **transport and inspection**.
- HTTP/2 and HTTP/3 are binary, so you cannot hand-type requests with netcat. Use `curl`, `nghttp`, or browser tools.
- QUIC is encrypted by default, so the "blocking UDP/443" decision is a real operational choice.

### 4. Security issues and attacks
- **HTTP/2 Rapid Reset (CVE-2023-44487, 2023):** attackers open and cancel huge numbers of streams very quickly, causing record-setting DDoS attacks.
- **HTTP/2 request smuggling / desync via protocol downgrade:** when a front-end proxy speaks HTTP/2 to clients but converts requests to HTTP/1.1 for the backend, mismatched length or header handling (H2.CL, H2.TE) can let attackers smuggle requests past security controls.
- **HPACK/QPACK abuse:** header-compression "bombs" and memory exhaustion in poorly implemented servers.
- **Other HTTP/2 resource attacks:** settings floods, ping floods, slow reads, stream-dependency abuse (several were disclosed together as the "HTTP/2 Netflix vulnerabilities" in 2019).
- **QUIC amplification and reflection** (UDP-based, address validation helps), connection-ID tracking, **0-RTT replay attacks** on non-idempotent requests.
- **Visibility gaps:** malware and exfiltration that use HTTP/3 over UDP 443 to slip past TCP-only inspection and logging.
- **Downgrade confusion:** security rules applied differently on different protocol versions.

### 5. Defenses
Keep web servers and proxies patched (Rapid Reset fixes), apply connection, stream and rate limits, use a CDN/WAF that handles HTTP/2 and HTTP/3 DDoS, **avoid HTTP/2 to HTTP/1.1 downgrading** or normalise and validate strictly at the proxy, disable HTTP/2 features you do not need, restrict 0-RTT to safe requests, decide your **UDP/443 policy** (allow with proper logging or block to force TCP), log HTTP version in access logs, update IDS/NDR/secure gateway products to support QUIC, and use endpoint/DNS telemetry to cover gaps.

### 6. Hands-on lab (real life)
1. **See the protocol in your browser:** open DevTools, Network tab, right-click the column header, enable **Protocol**. Browse several big sites and note `h2` versus `h3`. Reload a site to see it switch to `h3` after the `Alt-Svc` discovery.
2. **curl:** `curl -sI --http1.1 https://example.com`, `curl -sI --http2 https://example.com`, and if your curl supports it, `curl -sI --http3 https://cloudflare.com`. Use `curl -sI https://cloudflare.com | grep -i alt-svc`.
3. **Check support:** use an online HTTP/3 checker for a site, or `nghttp -nv https://nghttp2.org` to see the HTTP/2 frames (SETTINGS, HEADERS, DATA).
4. **Wireshark:** capture while loading a site. Filter `http2` (for TCP+TLS you will need keys), `tcp.port==443` versus `udp.port==443 || quic`. See that QUIC packets appear as UDP, and note how little is readable.
5. **Decrypt your own browser traffic:** set `SSLKEYLOGFILE=~/tls.keys` before launching Chrome/Firefox from the terminal, then point Wireshark to the file (**Edit, Preferences, Protocols, TLS, (Pre)-Master-Secret log filename**). Now filter `http2` or `http3` and read the headers.
6. **Force the fallback:** block QUIC on your Linux lab firewall (`sudo iptables -A OUTPUT -p udp --dport 443 -j REJECT`), reload a site and watch the browser fall back to `h2` over TCP. Remove the rule afterwards.
7. **Set up your own server:** configure Nginx with `listen 443 ssl http2;` using your lab certificate (see Topic 2), then confirm with `curl --http2 -kI https://<lab-IP>`. Optionally try Caddy, which enables HTTP/3 with little configuration.
8. **Reverse proxy awareness:** put Nginx or HAProxy in front of an HTTP/1.1 backend and read the documentation on how it converts HTTP/2 requests, then list the risks of downgrade desync.
9. **Rate limiting:** add `limit_req` and `http2_max_concurrent_streams` to your Nginx lab config and test with a loop of `curl` requests.
10. **Read the case study:** summarise the 2023 HTTP/2 Rapid Reset disclosures (Google, Cloudflare, AWS) in 8 lines: attack, scale, and fix.
11. **Further reading:** RFC 9113 (HTTP/2), RFC 9114 (HTTP/3), RFC 9000 (QUIC), and PortSwigger's "HTTP request smuggling" and "HTTP/2 downgrade" research.

### 7. Quick summary
- HTTP/2 multiplexes streams over one TCP connection. HTTP/3 uses QUIC over UDP with built-in TLS 1.3.
- Web attacks are the same, but **inspection changes**: QUIC on UDP/443 can bypass TCP-based controls.
- Watch Rapid Reset, HTTP/2 downgrade smuggling and your UDP/443 policy.

---

## 7. IoT and OT Protocols

### 1. What it is
- **IoT (Internet of Things):** connected consumer and business devices (cameras, smart plugs, sensors, wearables, routers).
- **OT (Operational Technology):** systems that monitor and control **physical processes**: factories, power grids, water treatment, pipelines, buildings. Includes **ICS/SCADA** (Industrial Control Systems / Supervisory Control and Data Acquisition), **PLCs** (Programmable Logic Controllers) and **HMIs** (Human-Machine Interfaces).
These systems often use lightweight, old or purpose-built protocols with **little or no security**, and a compromise can have **physical** consequences.

### 2. How it works
**MQTT (Message Queuing Telemetry Transport)**
- Lightweight **publish/subscribe** protocol over TCP, very common in IoT. Port **1883** (plain) and **8883** (TLS).
- A central **broker** (Mosquitto, EMQX, HiveMQ, cloud IoT hubs) routes messages. Clients **publish** to **topics** (for example `home/livingroom/temperature`) and **subscribe** to topics.
- Wildcards: `+` (one level) and `#` (everything below). Subscribing to `#` receives every message.
- **QoS levels:** 0 (at most once), 1 (at least once), 2 (exactly once). Features: **retained messages** (stored last value), **Last Will and Testament** (message sent if a client disconnects unexpectedly), optional username/password and client certificates.

**Modbus**
- A 1979 protocol for talking to PLCs and sensors. **Modbus/TCP** uses **port 502**; Modbus RTU uses serial lines.
- **Master/client** requests, **slave/server (PLC)** responds. Each device has a **Unit ID**. Data lives in **coils** (single bits, read/write), **discrete inputs** (read-only bits), **input registers** (read-only 16-bit) and **holding registers** (read/write 16-bit).
- Common **function codes:** 01 Read Coils, 02 Read Discrete Inputs, 03 Read Holding Registers, 04 Read Input Registers, 05 Write Single Coil, 06 Write Single Register, 15/16 write multiple.
- **No authentication, no encryption, no integrity.** Anyone who can reach port 502 can read and write values.

**Other protocols you will meet**

| Protocol | Port | Used for |
|----------|------|----------|
| DNP3 | TCP/UDP 20000 | Electric and water utilities |
| Siemens S7comm | TCP 102 | Siemens PLCs |
| EtherNet/IP (CIP) | TCP 44818, UDP 2222 | Rockwell/Allen-Bradley |
| OPC UA | TCP 4840 | Modern, secure-by-design industrial data exchange |
| BACnet | UDP 47808 | Building automation (HVAC) |
| CoAP | UDP 5683 | Constrained IoT devices (like HTTP for IoT) |
| Zigbee, Z-Wave, BLE, LoRaWAN | Radio | Short-range/low-power device networks |

**The Purdue Model (reference architecture for ICS networks)**

| Level | Name | Examples |
|-------|------|----------|
| 5 / 4 | Enterprise / Business | Corporate IT, email, ERP |
| 3.5 | **IT/OT DMZ** | Jump hosts, historians (replicas), patch servers |
| 3 | Operations | Manufacturing operations, engineering workstations |
| 2 | Supervisory control | HMI, SCADA servers |
| 1 | Basic control | PLCs, RTUs |
| 0 | Physical process | Sensors, motors, valves |

**Key differences between IT and OT security**
- **Priorities:** IT focuses on **Confidentiality** first. OT focuses on **Availability and Safety** first (a plant must not stop, people must not be hurt).
- **Lifespans:** OT equipment lasts 15 to 30 years, patching is rare and needs planned downtime.
- **Scanning can break things:** old PLCs can crash from an aggressive Nmap scan.
- **Standards:** **IEC 62443** (industrial security), **NIST SP 800-82** (ICS security guide), NERC CIP (electric utilities).

### 3. Key facts
- Most legacy industrial protocols were designed for isolated, trusted networks. Connectivity to IT and the internet exposed them.
- Search engines like Shodan regularly find thousands of exposed Modbus, S7 and BACnet devices, as well as open MQTT brokers.
- IoT devices often ship with default credentials, outdated firmware and no update mechanism.
- **Passive monitoring** (listening on mirrored traffic) is preferred in OT. Active scanning is used cautiously.

### 4. Security issues and attacks
- **Unauthenticated MQTT brokers:** anyone can subscribe to `#` and see all data, or publish commands to devices.
- **Cleartext MQTT/CoAP** with credentials in the clear. Weak or default passwords.
- **Modbus/DNP3/S7 command injection:** an attacker with network access can read process values or write coils and registers to change the process (open valves, stop motors).
- **Replay and man-in-the-middle** of unauthenticated industrial messages.
- **Reconnaissance:** scanning for open industrial ports and devices on the internet.
- **IoT botnets:** **Mirai** infected cameras and routers using default Telnet credentials and launched record DDoS attacks (see Level 6).
- **Firmware flaws and hard-coded credentials**, insecure update channels.
- **Flat IT/OT networks** where a phishing compromise on the corporate side reaches PLCs.
- **Landmark incidents:** **Stuxnet (2010)** targeted Siemens PLCs controlling centrifuges, **Ukraine power grid attacks (2015, 2016)** caused blackouts through compromised SCADA access, **TRITON/TRISIS (2017)** targeted safety instrumented systems.
- **Remote access risk:** vendor VPNs and remote desktop tools used for maintenance.

### 5. Defenses
- **Segmentation:** strict **IT/OT separation** with an industrial DMZ, firewalls between Purdue levels, and **one-way gateways (data diodes)** for the most critical links.
- **Never expose OT protocols to the internet.** Control remote access with MFA, jump hosts, session recording and time-limited vendor access.
- **Protocol-aware firewalls and allowlists:** allow only specific function codes, source hosts and register ranges (for example read-only Modbus from the historian).
- **Passive network monitoring** with OT-aware tools (Zeek with ICS parsers such as ICSNPP, Dragos, Claroty, Nozomi) to build asset inventory and detect new devices or unusual commands.
- **For MQTT:** require authentication, use **TLS (8883)**, per-client credentials or certificates, topic-level ACLs, disable anonymous access, restrict broker exposure.
- **For IoT:** change default passwords, update firmware, disable unused services (Telnet, UPnP), put devices on an isolated VLAN, and prefer vendors that provide updates.
- **Use secure alternatives** where possible: OPC UA with security enabled, Modbus/TCP Security (TLS), IEC 62351 for power-system protocols.
- **Patch with a plan:** test, schedule downtime, use virtual patching and compensating controls where patching is not possible.
- **Backups and incident plans** specific to OT, including safe manual operation procedures.
- **Physical security** of PLCs and engineering workstations, and USB control.

### 6. Hands-on lab (real life)
**Use simulators and your own VMs only. Never scan or send commands to real industrial equipment or someone else's IoT devices.**
1. **MQTT broker:** `sudo apt install mosquitto mosquitto-clients`. Start the broker. In terminal 1: `mosquitto_sub -h localhost -t 'home/#' -v`. In terminal 2: `mosquitto_pub -h localhost -t home/livingroom/temp -m 22.5`. Watch it arrive.
2. **Capture MQTT:** in Wireshark filter `mqtt` and read the CONNECT, PUBLISH and SUBSCRIBE packets. Find the topic and payload in clear text.
3. **Show the danger of `#`:** subscribe to `#` from a second client and see every topic. Publish a "command" message to a topic and discuss what a real device would do.
4. **Secure the broker:** create a password file (`mosquitto_passwd -c /etc/mosquitto/passwd alice`), set `allow_anonymous false` and `password_file ...` in the config, add an ACL file restricting topics, and confirm anonymous clients are rejected.
5. **Add TLS:** generate certificates (your lab CA from Topic 2), enable a TLS listener on 8883, connect with `mosquitto_sub --cafile ca.crt -p 8883 ...`, and capture again to confirm the payload is now encrypted.
6. **Scan your own broker:** `nmap -sV -p 1883,8883 <lab-IP>` and `nmap --script mqtt-subscribe -p 1883 <lab-IP>` (own lab only).
7. **Modbus simulator:** run a Modbus/TCP server simulator on a lab VM (for example `pymodbus` server example, `diagslave`, or ModbusPal) on port 502 with some coils and registers.
8. **Talk Modbus:** from another VM use `mbpoll` or a Python `pymodbus` client to read holding registers and coils (function codes 03 and 01), then write a coil (05). Capture with Wireshark filter `mbtcp` (or `tcp.port==502`) and decode the function code, unit ID and values. Notice there is no login of any kind.
9. **Detect and restrict:** write a firewall rule so only one "HMI" VM can reach port 502 on the PLC simulator. Add a Suricata or Zeek rule that alerts on any Modbus **write** function code (05, 06, 15, 16) from an unexpected host, then test it.
10. **Scan carefully:** run `nmap -sV -p 502 --script modbus-discover <simulator-IP>` only against your simulator, and note why you would not do this to a real PLC.
11. **Draw a Purdue-model diagram** for a fictional water treatment plant and mark where firewalls, the DMZ, and remote access should sit.
12. **View-only exposure research:** read articles and reports on how many Modbus/MQTT devices Shodan lists as exposed, without connecting to any.
13. **Practice platforms:** TryHackMe IoT and ICS rooms, **OWASP IoTGoat** (a deliberately vulnerable firmware), **Damn Vulnerable ICS/IoT** style simulators, SANS ICS free resources, and ICS CTFs (for example in hacking conferences).
14. **Read:** NIST SP 800-82 and CISA's industrial control systems advisories.

### 7. Quick summary
- MQTT is pub/sub over a broker. Modbus is an old, unauthenticated protocol for PLCs on port 502.
- OT prioritises safety and availability. Scanning and patching must be careful.
- Segment IT from OT (Purdue model, industrial DMZ), restrict function codes, secure brokers with auth and TLS, monitor passively.

---

## 8. Logging, SIEM and Detection Engineering

### 1. What it is
- **Logging:** recording events from systems, applications and the network.
- **SIEM (Security Information and Event Management):** a platform that **collects, normalises, stores, searches and correlates** logs from many sources and raises alerts. Examples: Splunk, Elastic Security, Microsoft Sentinel, IBM QRadar, Wazuh, Graylog.
- **Detection engineering:** the discipline of designing, building, testing and tuning **detection rules** so that real attacks are found with few false alarms.
- **SOC (Security Operations Center)** analysts then **triage** the alerts.

### 2. How it works
**Log sources (what to collect)**

| Area | Examples | Why |
|------|----------|-----|
| Network | Firewall, IDS/IPS, NetFlow, DNS, DHCP, proxy, VPN, Zeek | Who talked to whom, DNS lookups, blocked traffic |
| Authentication | Windows Security log, Linux `auth.log`, Active Directory, SSO/IdP, MFA, RADIUS | Logins, failures, privilege use |
| Endpoint | **Sysmon**, EDR, antivirus, `auditd` | Processes, files, registry, network connections |
| Servers/apps | Web server, database, application logs | Attacks against applications |
| Email | Mail gateway, Microsoft 365/Google Workspace audit | Phishing, mailbox rules |
| Cloud | CloudTrail, Azure Activity/Entra ID logs, GCP Audit Logs, flow logs | API calls and configuration changes |

**Key Windows event IDs to know:** 4624 (successful logon), 4625 (failed logon), 4648 (explicit credential logon), 4672 (special privileges), 4688 (process creation), 4698 (scheduled task created), 4720 (account created), 4768/4769 (Kerberos ticket requests, useful for Kerberoasting), 4776 (NTLM authentication), **1102/104 (audit/event log cleared)**. **Sysmon** event 1 (process creation), 3 (network connection), 11 (file created).

**Pipeline**
```
Sources -> Collect (agents, syslog, Beats) -> Parse and normalise (ECS, CIM)
        -> Store and index -> Search and correlate -> Detect/Alert -> Triage -> Respond
```
- **Normalisation:** mapping different log formats to common field names (for example `source.ip`, `user.name`) so one rule works across products. Standards: Elastic Common Schema (ECS), Splunk CIM, OCSF.
- **Time sync:** wrong timestamps break correlation (see NTP, Level 3).
- **Retention and integrity:** keep logs long enough (often 90 days hot, 1 year or more archived), protect them from tampering, and forward them off the originating device.

**Types of detection rules**
- **Signature/IOC match:** known bad IP, domain, hash, user-agent (threat intelligence feeds).
- **Threshold:** more than N failed logins from one source in 5 minutes.
- **Correlation/sequence:** event A followed by event B (failed logins followed by a success, then a new admin account).
- **Behavioural/anomaly:** deviation from a baseline (a user logging in from a new country, a server sending unusually large outbound data, rare process spawning).
- **Statistical/ML-assisted** detections, with careful tuning.

**Sigma: a vendor-neutral rule format**
You write a rule once in YAML and convert it to Splunk, Elastic, Sentinel and others with `sigma-cli`. Example (simplified):
```yaml
title: Windows Security Event Log Cleared
id: 11111111-2222-3333-4444-555555555555
status: experimental
description: Detects clearing of the Windows Security event log, a common attacker cover-up step
logsource:
  product: windows
  service: security
detection:
  selection:
    EventID: 1102
  condition: selection
level: high
tags:
  - attack.defense_evasion
  - attack.t1070.001
falsepositives:
  - Authorised administrative maintenance
```
Related rule languages: **Suricata/Snort** (network), **YARA** (files/memory).

**Detection engineering lifecycle**
1. **Pick a threat/behaviour** (a MITRE ATT&CK technique, a recent incident, a finding from a pentest).
2. **Identify the data source** needed and confirm the logs actually exist (the most common failure).
3. **Write the detection** (Sigma, SPL, KQL, Suricata).
4. **Test it:** generate the behaviour safely with **Atomic Red Team** or CALDERA in a lab and confirm the rule fires. Also test against normal activity to measure false positives.
5. **Tune:** add exclusions, thresholds and context. Document known false positives.
6. **Deploy** with a runbook (what to check when it fires).
7. **Measure and improve:** true positive rate, false positive rate, **MTTD** (mean time to detect), **MTTR** (mean time to respond), ATT&CK coverage. Retire useless rules.
- **Detection as code:** keep rules in Git, review changes, run automated tests.
- **Pyramid of Pain:** detections based on hashes and IPs are easy for attackers to dodge. Detections based on **tools and behaviours (TTPs)** hurt them most.

**Alert triage (the analyst's workflow)**
1. **Read the alert:** what rule fired, what severity, what asset, what user, what time.
2. **Gather context:** asset criticality, user role, recent activity of that host/user, related alerts (use the "5 Ws": who, what, when, where, why).
3. **Enrich:** WHOIS, passive DNS, VirusTotal, AbuseIPDB, internal threat intelligence, geolocation, Tor exit list (see Topic 9).
4. **Decide:** **True Positive** (real malicious activity), **Benign True Positive** (rule fired correctly but activity was authorised, such as a pentest), **False Positive** (rule fired wrongly), or **Needs more information**.
5. **Act:** close with notes, tune the rule, or **escalate** to incident response (contain the host, reset credentials, block indicators).
6. **Document** the reasoning. Good notes make handovers and audits easy.
- Beware **alert fatigue:** too many low-value alerts cause real ones to be missed.

### 3. Key facts
- You can only detect what you **log**. Visibility comes before detection.
- Logs from **DNS, authentication and endpoint process creation** give the highest security value for the effort.
- A SIEM is only as good as its **data quality, parsing and tuning**.
- Attackers try to blind you: clearing logs, stopping agents, disabling auditing, timestomping.

### 4. Security issues and attacks
- **Log tampering and deletion** (event log clearing, editing local log files).
- **Disabling or evading logging/agents** (stopping Sysmon, blocking SIEM traffic, modifying audit policy).
- **Log injection and forging** (unsanitised input written into logs, spoofed syslog).
- **Log flooding** to hide malicious events or exhaust storage.
- **Coverage gaps:** unmonitored segments, cloud accounts, IPv6, encrypted traffic, shadow IT.
- **Sensitive data in logs:** passwords and tokens written by applications, privacy risks.
- **SIEM compromise:** a SIEM holds the keys to the kingdom (credentials, asset data), so protect it as a high-value target.
- **Misconfigured retention** that deletes evidence before an investigation starts.

### 5. Defenses
Forward logs in near real time to a separate, access-controlled system, enable write-once or immutable storage and integrity checks, alert on log-clearing and agent-stopped events, monitor for **silent sources** (a server that stops sending logs is itself an alert), standardise time with NTP, collect the high-value sources first (identity, DNS, endpoints, firewalls, cloud audit logs), enable Sysmon with a well-known configuration (for example from SwiftOnSecurity or Olaf Hartong), keep SIEM access to least privilege with MFA, sanitise application logging, and review detection coverage against MITRE ATT&CK.

### 6. Hands-on lab (real life)
1. **Look at raw logs first:** on Linux run `sudo journalctl -u ssh --since "1 hour ago"` and `sudo tail -f /var/log/auth.log`. On Windows open **Event Viewer, Windows Logs, Security**. Find a 4624 and a 4625 event after you log in and fail once. Note the fields.
2. **Build a mini SIEM:** install **Wazuh** (all-in-one in a VM, or the Docker version) or use **Security Onion**, **Graylog** or the **Elastic stack**. Enrol a Linux VM and a Windows VM as agents.
3. **Generate events on purpose (lab only):** attempt 10 failed SSH logins to your own VM (from another lab VM), then one success. Find them in the SIEM dashboard. Run `net user labuser Passw0rd! /add` on the Windows VM and find event 4720.
4. **Install Sysmon** on the Windows VM with the SwiftOnSecurity config, run a few commands (`whoami`, `powershell -c "Get-Process"`) and search for Sysmon Event ID 1 process creation records, including the **parent process** and **command line**.
5. **Write a threshold alert:** "more than 5 failed SSH logins from one IP in 5 minutes". Trigger it, confirm it fires, and note what a false positive could look like (a user mistyping their password).
6. **Write a Sigma rule:** use the example above (log cleared, 1102) or write one for failed logons. Install `sigma-cli` (`pip install sigma-cli`) and convert it for your SIEM (`sigma convert -t splunk -p <pipeline> rule.yml` or the Elasticsearch backend). Test it by clearing the Security log in your lab VM (`wevtutil cl Security`).
7. **Test with Atomic Red Team:** in your Windows lab VM run one harmless Atomic test for a technique (for example T1087 account discovery or T1059.001 PowerShell), and check whether your SIEM detects it. If not, find which log source or rule is missing.
8. **Add enrichment:** write a small Python script or SIEM lookup that checks a source IP against the Tor exit list and a threat-intelligence feed (see Topic 9).
9. **Practice triage:** take 5 alerts from your lab (or from a public dataset). For each, write a short triage note: what fired, context, enrichment, verdict (TP / benign TP / FP), action. Use a template and keep it in Git.
10. **Silent-source test:** stop the agent on one VM and create an alert for "no logs received from host X in 15 minutes".
11. **Public datasets and challenges:** Splunk **Boss of the SOC (BOTS)** datasets, **Security Datasets (OTRF)**, **EVTX-ATTACK-SAMPLES**, **CyberDefenders**, **Blue Team Labs Online**, TryHackMe SOC Level 1 path and "Splunk" and "ELK" rooms.
12. **Map coverage:** list 10 ATT&CK techniques from this course (T1046 network scanning, T1110 brute force, T1021 remote services, T1071.004 DNS C2, T1048 exfiltration, T1070 log clearing...) and mark which ones you can detect with your lab logs.

### 7. Quick summary
- Logging gives visibility, a SIEM collects and correlates, detection engineering writes and tunes the rules.
- Start with identity, DNS, endpoint and firewall logs. Write rules from ATT&CK behaviours (Sigma), then test with Atomic Red Team.
- Triage = context, enrichment, verdict, action, documentation. Watch for log tampering and silent sources.

---

## 9. Anonymity Networks

### 1. What it is
Anonymity networks and proxy services hide **who is connecting to what** by routing traffic through intermediate systems. They protect privacy and help people in censored or dangerous environments, such as journalists, activists and ordinary users. They are also abused by criminals, so security teams and investigators must understand them. Main types: **Tor**, **VPNs and proxies**, and others such as **I2P**.

### 2. How it works
**Tor (The Onion Router)**
- Volunteer-run network of thousands of **relays**. Your traffic is wrapped in **multiple layers of encryption** (like an onion) and sent through **three relays**:
  1. **Guard (entry) relay:** knows your real IP but not your destination.
  2. **Middle relay:** knows neither you nor the destination.
  3. **Exit relay:** sees the destination and the traffic (if it is not HTTPS), but does not know who you are.
- Each relay peels off one layer. No single relay knows both the source and the destination. A **circuit** is the path through the three relays and changes regularly.
- **Directory authorities** publish the list of relays. **Bridges** are unlisted entry relays for places where Tor is blocked, and **pluggable transports** (obfs4, Snowflake, meek) disguise Tor traffic as something else.
- **Onion services (`.onion` sites):** both client and server connect through Tor and meet at **rendezvous points**, so the server's real IP is hidden too and traffic never leaves the Tor network. Used for privacy-preserving websites, secure messaging, SecureDrop whistleblower systems, and also by illegal services.
- Tor Browser adds protections against browser fingerprinting and tracking.

**VPNs and proxies**
- **Commercial VPN:** encrypts traffic to the provider. Websites see the VPN's IP, but **the provider can see your traffic metadata** and you must trust it (see Level 3).
- **HTTP/SOCKS proxies:** relay traffic with little or no encryption. **Open proxies** are often abused or malicious.
- **Residential proxies:** networks of real home devices rented as exit points, sometimes created with the owners' knowledge and sometimes through malware or shady apps, making attacker traffic look like normal home users.
- **Proxy chains** and multi-hop VPNs add layers but also complexity.
- **Bulletproof hosting and fast-flux DNS** are infrastructure services that attackers use to resist takedown.

**I2P:** another anonymity network with "garlic routing", mostly used for in-network services.

**What anonymity does and does not give you**
- It hides your IP from the destination. It does **not** make you invisible: **browser fingerprinting, logged-in accounts, malware, human mistakes (opsec failures), timing correlation and compromised endpoints** can all reveal identity.
- **Traffic correlation:** an adversary who can observe both the entry and the exit sides of the network may match traffic patterns by timing and volume.
- **Exit relays can see unencrypted traffic** and can tamper with it, so use HTTPS or onion services.

**How investigators and defenders deal with it**
- **Known infrastructure lists:** the Tor Project publishes the **list of exit node IPs** (`https://check.torproject.org/torbulkexitlist`) and relay lists. Threat-intel vendors maintain lists of VPN, proxy and residential-proxy ranges.
- **Enrichment in the SIEM:** flag logins, payments or account signups from Tor/VPN/proxy IPs and raise the risk score (see Topic 8).
- **Policy decisions:** block Tor exits for sensitive services, require MFA, rate-limit or add friction. Do not assume every Tor user is malicious.
- **Detect Tor use inside your network:** connections to known guard relays, Tor's characteristic TLS patterns, and traffic to bridge/pluggable-transport endpoints are signs, although bridges and obfuscation make detection imperfect.
- **Endpoint and account evidence:** investigations usually succeed through **other evidence** (endpoint logs, device identifiers, payment trails, cookies/sessions, malware artefacts, email and OSINT links, leaked or reused handles, and mistakes) rather than "breaking" Tor's cryptography.
- **Legal process and cooperation:** hosting providers, registrars, payment processors and platforms can respond to lawful requests. Law enforcement operations have compromised servers or exploited operational mistakes, and some operations used network investigative techniques under legal authority.
- **Tracing malicious infrastructure:** pivot on certificates, hosting providers, domain registration, JA3/JA4 fingerprints, favicons and shared infrastructure to link services.
- **Ethics and law:** respect privacy and legal boundaries. Do not access illegal marketplaces or content. Investigations should run under proper authority, in your own jurisdiction's rules.

### 3. Key facts
- Tor's strength is **layered encryption and distributed trust**. Its weaknesses are usually at the **edges and in users' behaviour**.
- A VPN moves trust from your ISP to your VPN provider. Tor distributes trust across relays. Neither is a magic cloak.
- Anonymity tools have many legitimate uses (privacy, censorship circumvention, safe reporting), so blanket blocking has costs.
- For defenders, Tor and proxy use is a **risk signal**, not proof of malicious intent.

### 4. Security issues and attacks
- **Malware command-and-control** hidden behind Tor, proxies or residential proxy networks. Credential-stuffing and fraud launched from residential proxies to appear legitimate.
- **Illegal marketplaces and leak sites** hosted as onion services, and ransomware groups using Tor for negotiation portals.
- **Attacks on users of anonymity networks:** malicious exit relays that modify traffic, SSL stripping, fake download sites, deanonymisation by browser exploits or by user error.
- **Traffic correlation and relay-level attacks** by powerful adversaries.
- **Open proxy and VPN abuse** to hide origin of scans, brute force and DDoS.
- **Data exfiltration over Tor** from compromised hosts.
- **Shadow IT risk** when employees install Tor/VPN clients that bypass corporate controls.

### 5. Defenses
For organisations: maintain and update lists of Tor exit nodes and known anonymising proxies, add them to SIEM enrichment and risk scoring, restrict outbound traffic with egress filtering and a mandatory proxy so unapproved anonymisers cannot be used from servers, monitor DNS and TLS for Tor/bridge indicators, enforce **MFA** and device checks (the best answer to anonymised credential attacks), rate-limit and use bot detection, and monitor for impossible-travel or residential-proxy patterns. For users who need privacy: use the Tor Browser as designed, keep it updated, prefer HTTPS/onion services, avoid logging in to personal accounts, do not open downloaded files while online, and do not install extra browser add-ons.

### 6. Hands-on lab (real life)
**Stay legal. Do not visit illegal marketplaces or content. Do not run a Tor exit relay without understanding the legal and operational responsibilities.**
1. **Try Tor Browser:** download it only from `torproject.org`, open `https://check.torproject.org`, and note that your visible IP is different. Open the circuit display (the padlock/onion icon) and read the three relays and their countries. Press "New Tor Circuit for this Site" and see the path change.
2. **Compare:** check your IP on `ifconfig.me` normally, through Tor, and through a VPN. Record the three results and what each reveals (use `ipleak.net` or `dnsleaktest.com` for DNS leak checks).
3. **Tor on the command line (Linux):** `sudo apt install tor`, start it, then `curl --socks5-hostname 127.0.0.1:9050 https://check.torproject.org/api/ip`. The `--socks5-hostname` option sends DNS through Tor as well. Try the same request **without** it and discuss the DNS leak risk.
4. **Capture Tor traffic:** run Wireshark while using Tor. Identify the connection to the guard relay on a TLS port (often 443 or 9001). Note that you cannot read the content, and that the DNS lookup does not leak when you use Tor correctly.
5. **Build a Tor exit list lookup for defenders:** download the exit list: `curl -s https://check.torproject.org/torbulkexitlist -o tor_exits.txt`. Write a small script (grep or Python) that reads a sample log file (your lab `auth.log` or a web access log) and flags any source IP found in the list. Add this as an enrichment step in your Topic 8 SIEM or triage notes.
6. **Check an IP:** use AbuseIPDB, VirusTotal and Shodan (view only) on a Tor exit IP from the list and on a normal IP. Note what each source tells you.
7. **Create an onion service (own lab only):** run a simple local web server (`python3 -m http.server 8080`), add to `/etc/tor/torrc`: `HiddenServiceDir /var/lib/tor/lab_service/` and `HiddenServicePort 80 127.0.0.1:8080`, restart Tor, read the generated `.onion` hostname, and visit it from Tor Browser. Observe that you did not need any public IP or port forwarding. Then shut it down.
8. **Detection exercise:** write a Suricata/Zeek-style detection idea for "internal host connects to many different IPs on port 9001/443 matching Tor relay lists", and list how you would verify a hit (check the relay list, process on the host, user context).
9. **Proxy awareness:** set up a Squid proxy (Level 4) and an SSH SOCKS proxy (`ssh -D 1080`). Compare what the destination site sees, what the proxy logs record, and what the network capture shows.
10. **Case studies (reading only):** summarise how investigators have identified people using anonymity tools in public case reports (the common themes are operational security mistakes, reused identities, server misconfigurations and traditional investigation methods). Write 8 lines on lessons for defenders.
11. **Read:** the Tor Project's design documents and "How Tor works" page, EFF's Surveillance Self-Defense guide, and MITRE ATT&CK technique **T1090 (Proxy)** and **T1090.003 (Multi-hop Proxy)**.

### 7. Quick summary
- Tor routes traffic through three relays with layered encryption. Onion services hide both sides.
- Anonymity is strong at the network layer but weak at the edges: fingerprinting, accounts, malware and human mistakes.
- Defenders flag Tor/VPN/proxy IPs as a risk signal, enforce MFA and egress controls, and rely on endpoint and other evidence in investigations.

---

## Level 7 Checklist
Tick when you can explain or do each without notes.

- [ ] Write IPv6 addresses in short form and identify link-local, ULA, global and multicast addresses
- [ ] Describe NDP, SLAAC and DAD and name the ICMPv6 message types
- [ ] Explain rogue RA attacks and how RA Guard stops them
- [ ] Explain hashing, HMAC, symmetric, asymmetric and digital signatures, and when to use each
- [ ] Explain forward secrecy and why ECDHE matters
- [ ] Build a small CA, issue a certificate and verify the chain with OpenSSL
- [ ] Describe the 802.1X flow (supplicant, authenticator, RADIUS) and name three EAP methods
- [ ] Explain NAC, MAB and why MAB is weak
- [ ] Explain Zero Trust principles and the difference between VPN and ZTNA access
- [ ] Draw a VPC with public and private subnets and explain security groups vs network ACLs
- [ ] List 6 common cloud network misconfigurations and their fixes
- [ ] Explain OSPF neighbour formation and how BGP chooses and announces routes
- [ ] Explain a BGP prefix hijack, why more-specific prefixes win, and how RPKI helps
- [ ] Compare HTTP/1.1, HTTP/2 and HTTP/3 and explain why QUIC changes network inspection
- [ ] Explain MQTT topics and wildcards, and Modbus function codes and why Modbus is insecure
- [ ] Describe the Purdue model and why OT security priorities differ from IT
- [ ] List key log sources and Windows event IDs, and describe the detection engineering lifecycle
- [ ] Write a basic Sigma rule and triage an alert with a clear verdict
- [ ] Explain how Tor works (guard, middle, exit, onion services) and how defenders handle Tor traffic

## Practice Questions
1. Why is blocking all ICMPv6 a mistake?
2. What is the IPv6 equivalent of ARP spoofing, and which switch feature defends against it?
3. Why are MD5 and SHA-1 unsuitable for security, and what should you use to store passwords?
4. What is the difference between signing a message and encrypting it?
5. In a PEAP Wi-Fi network, why must clients validate the server certificate?
6. What does "never trust, always verify" mean in practice for an internal web application?
7. How does a security group differ from a network ACL in AWS?
8. How can a vulnerable web application lead to stolen cloud credentials via the metadata service?
9. Why does announcing a /24 inside someone's /23 hijack their traffic, and what does RPKI ROV do about it?
10. Why can HTTP/3 traffic bypass a TCP-only inspection device, and what are two ways to handle it?
11. Why is an unauthenticated Modbus device dangerous, and which Modbus function codes would you alert on?
12. Name three log sources you would collect first and why.
13. What is the difference between a True Positive and a Benign True Positive?
14. Can an investigator simply "decrypt" Tor to find a user? What usually reveals identity instead?

**Answers:** (1) NDP, SLAAC, DAD and path MTU discovery all use ICMPv6, so the network breaks. (2) NDP spoofing or rogue RA. RA Guard, ND inspection and IPv6 Source Guard. (3) They are fast and have known collisions, so they can be attacked. Use Argon2, bcrypt, scrypt or PBKDF2 with a unique salt. (4) Signing proves origin and integrity using the private key and does not hide content. Encryption hides content and does not by itself prove who sent it. (5) Otherwise an evil twin can present its own certificate and capture the inner username and password hash. (6) Each request is authenticated, device health is checked and access is limited to that app regardless of network location, and everything is logged. (7) Security groups are stateful, attached to instances and allow-only. NACLs are stateless, attached to subnets and have allow and deny rules. (8) SSRF makes the server fetch `169.254.169.254` and return role credentials. IMDSv2 and restricted roles reduce the risk. (9) Routers choose the longest matching prefix, so the more specific /24 wins. ROV rejects routes whose origin or prefix length does not match a signed ROA. (10) HTTP/3 uses QUIC over UDP/443, which TCP-only tools do not see and which is encrypted. Block UDP/443 to force fallback, or use inspection tools and endpoint telemetry that support QUIC. (11) Anyone who reaches port 502 can read and write values with no authentication. Alert on write codes 05, 06, 15 and 16 from unexpected hosts. (12) Identity/authentication logs, DNS logs and endpoint process-creation (Sysmon/EDR) logs, because they show most attacker behaviour. (13) A True Positive is real malicious activity. A Benign True Positive is real detected activity that was authorised, such as a pentest. (14) No. Identity is usually revealed through endpoint evidence, opsec mistakes, account or payment links, malware artefacts, server misconfiguration and legal process.

## What to Learn Next
1. **Linux Architecture** (file system, permissions, processes, bash scripting), the next group in your teacher's roadmap
2. **Core Web Concepts** and **Common Web Security Flaws** (XSS, SQL injection, CSRF, SSRF, RCE, broken authentication)
3. **Advanced Web Exploitation** (XXE and beyond)
4. **Cyber Threat Intelligence and Incident Response**, which build directly on Topics 8 and 9
5. **Capstone project:** build a lab with a DMZ, firewall, IDS, SIEM, a Zero Trust-style access layer and a small cloud component. Attack it, detect it, and write the incident report.

## Real-Life Practice Platforms for This Level
- **IPv6:** Hurricane Electric IPv6 Certification and tunnel broker (free), test-ipv6.com, THC-IPv6 toolkit (lab use), Packet Tracer IPv6 labs
- **Cryptography:** CryptoHack, Cryptopals, TryHackMe "Cryptography" modules, OpenSSL man pages and the "OpenSSL Cookbook" (free)
- **802.1X and Zero Trust:** FreeRADIUS documentation, NIST SP 800-207, CISA Zero Trust Maturity Model, Cloudflare Access and Tailscale free tiers
- **Cloud:** flaws.cloud, flaws2.cloud, CloudGoat, TerraGoat, AWS/Azure/GCP free tiers (with billing alerts), Prowler, ScoutSuite, Checkov
- **Routing:** GNS3, Packet Tracer, Containerlab, FRRouting, bgp.he.net, RIPEstat, isbgpsafeyet.com, MANRS
- **HTTP/2, HTTP/3 and QUIC:** browser DevTools, `curl`/`nghttp`, Wireshark, PortSwigger Web Security Academy (HTTP request smuggling labs), the HTTP RFCs
- **IoT and OT:** Mosquitto, pymodbus/diagslave/ModbusPal simulators, OWASP IoTGoat, SANS ICS resources, NIST SP 800-82, CISA ICS advisories
- **Logging and detection:** Wazuh, Security Onion, Elastic, Graylog, Sigma rules repository, Atomic Red Team, Splunk BOTS, CyberDefenders, Blue Team Labs Online, TryHackMe SOC paths
- **Anonymity:** Tor Project documentation, EFF Surveillance Self-Defense, AbuseIPDB, VirusTotal, Shodan (view only), MITRE ATT&CK T1090
