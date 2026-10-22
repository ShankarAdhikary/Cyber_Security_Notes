# Networking for Cyber Security: How to Use These Notes

These notes cover the networking knowledge needed for cyber security, from basic concepts to network attacks and analysis. This file explains how the notes are organized and how to study them.

---

## 1. Structure of the Notes

**1 Level = 1 Markdown file.** Each level has one `.md` file containing all of its topics, in order.

```
networking-for-cybersecurity/
├── README.md                          <- this file (how to use the notes)
├── Level-1-Networking-Fundamentals.md
├── Level-2-Core-Protocols.md
├── Level-3-Other-Security-Protocols.md
├── Level-4-Network-Defense.md
├── Level-5-Practical-Tools.md
└── Level-6-Attacks-and-Advanced.md
```

---

## 2. Study Sequence

Follow the levels in order. Each level builds on the one before it.

| Level | File | Focus | Topics |
|-------|------|-------|--------|
| 1 | `Level-1-Networking-Fundamentals.md` | Core concepts: models, addressing, switching, routing | 9 |
| 2 | `Level-2-Core-Protocols.md` | IP, ICMP, TCP, UDP, ARP, DHCP, DNS, HTTP, HTTPS/TLS, SSH | 9 |
| 3 | `Level-3-Other-Security-Protocols.md` | FTP, Telnet, email, SMB, RDP, LDAP/Kerberos, SNMP, VPN, wireless | 9 |
| 4 | `Level-4-Network-Defense.md` | Firewalls, IDS/IPS, proxies, segmentation, hardening | 6 |
| 5 | `Level-5-Practical-Tools.md` | Wireshark, Nmap, tcpdump, netcat, dig, traceroute | 7 |
| 6 | `Level-6-Attacks-and-Advanced.md` | Recon, sniffing, spoofing, DoS, tunneling, pentest workflow, forensics | 8 |

**Why this order?**
- Levels 1-2 give you the foundation. Without them, nothing later makes sense.
- Level 3 widens your protocol knowledge to what you meet in real environments.
- Level 4 teaches how networks are protected, so you understand what attacks are up against.
- Level 5 gives you the tools to see and test everything you learned.
- Level 6 brings it together with attacks, methodology and forensics.

**Tip:** Do not skip to Level 6. Attacks only make sense once you know the protocol being attacked.

---

## 3. Structure of Every Topic

Every topic inside every level file uses the same template:

```
## Topic Name

### 1. What it is
Short definition in simple words.

### 2. How it works
Step-by-step explanation, with a diagram or packet example.

### 3. Key facts
Port numbers, OSI layer, important terms, quick-reference table.

### 4. Security issues and attacks
Known weaknesses and how attackers abuse them.

### 5. Defenses
How to detect, prevent or reduce the risk.

### 6. Hands-on lab
A practical exercise (Wireshark, Nmap, or a command to run).

### 7. Quick summary
3-5 bullet points to revise before an exam or interview.
```

Each level file starts with:
- A short **overview** of the level
- A **table of contents** linking to every topic
- **Prerequisites** (what you should already know)

And ends with:
- A **level checklist** of what you must be able to do before moving on
- **Practice questions**

---

## 4. How to Study Each Topic

1. **Read** the "What it is" and "How it works" sections.
2. **Do the lab.** Do not skip it; hands-on practice is what makes the knowledge stay.
3. **Read** the security and defense sections, and think from both the attacker's and the defender's side.
4. **Write the quick summary in your own words** to check you understood it.
5. **Tick the level checklist** only when you can explain each item without looking at the notes.

---

## 5. Recommended Tools and Setup

Set these up early, because you will use them from Level 2 onward:

| Tool | Purpose | Used from |
|------|---------|-----------|
| Wireshark | Packet capture and analysis | Level 2 |
| Nmap | Scanning and enumeration | Level 3 onward |
| Kali Linux or Parrot OS (VM) | Security toolkit | Level 5 |
| VirtualBox or VMware | Safe lab environment | Level 1 |
| TryHackMe / Hack The Box | Guided practice | All levels |

---

## 6. Lab Safety and Legal Rules

- Only scan or attack systems that **you own or have written permission to test**.
- Use your own virtual machines, or legal platforms such as TryHackMe, Hack The Box and other CTF sites.
- Unauthorized scanning or access is illegal in most countries, including under India's IT Act.

---

## 7. Progress Tracker

Tick each level when you finish its checklist.

- [ ] Level 1: Networking Fundamentals
- [ ] Level 2: Core Protocols
- [ ] Level 3: Other Security Protocols
- [ ] Level 4: Network Defense
- [ ] Level 5: Practical Tools
- [ ] Level 6: Attacks and Advanced Topics

---

## 8. Naming and Formatting Conventions

- File names: `Level-<number>-<Title-With-Hyphens>.md`
- One `#` heading for the level title, `##` for each topic, `###` for template sections
- Use tables for port numbers and comparisons
- Use code blocks for commands and packet captures
- Keep explanations simple; add your own examples and screenshots from labs

---

## 9. Contribution and Updates

As you learn, improve these notes:
- Add screenshots from your own Wireshark captures
- Add commands you actually tested
- Note mistakes you made and how you fixed them

Notes you wrote and tested yourself are worth far more than copied text.
