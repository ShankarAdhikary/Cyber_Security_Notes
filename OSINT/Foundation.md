# OSINT — Level 1: Foundations
> Before touching a single tool, you need to understand what OSINT is, how investigators think,
> the legal lines you cannot cross, and how to protect your own identity while researching.
> Everything built in later levels sits on this foundation.

---

## 1. What is OSINT?

### 1.1 Definition

**OSINT (Open Source Intelligence)** is the process of collecting, analyzing, and correlating
information that is **publicly available** — legally accessible without any hacking, unauthorized
access, or covert methods.

The word "Open Source" does not mean open-source software. It means **open to the public** —
information that anyone can access if they know where to look.

### 1.2 The Intelligence Cycle

OSINT is not just searching Google. It follows a structured intelligence cycle:

```
┌─────────────────────────────────────────────────────┐
│                 THE INTELLIGENCE CYCLE              │
│                                                     │
│   1. PLANNING        →   Define the question        │
│          ↓                                          │
│   2. COLLECTION      →   Gather raw data            │
│          ↓                                          │
│   3. PROCESSING      →   Clean and organize data    │
│          ↓                                          │
│   4. ANALYSIS        →   Connect dots, find meaning │
│          ↓                                          │
│   5. DISSEMINATION   →   Report the findings        │
│          ↓                                          │
│   6. FEEDBACK        →   Refine and repeat          │
└─────────────────────────────────────────────────────┘
```

Most beginners jump straight to Step 2 (Collection) and never do Step 1 (Planning).
That is why their investigations go in circles. **Always define your question first.**

### 1.3 What Counts as "Open Source"?

| Source Type | Examples |
|---|---|
| **Public web** | Websites, blogs, news articles, forums |
| **Social media** | Twitter/X, Instagram, Facebook, LinkedIn, Reddit, TikTok |
| **Government records** | Company registries, court records, land records, election data |
| **Academic** | Research papers, theses, conference proceedings |
| **Commercial databases** | Business registries, patent databases |
| **Media** | Newspapers, TV broadcasts, podcasts |
| **Dark web** (public areas) | Public Tor forums, Ahmia-indexed pages |
| **Code repositories** | GitHub, GitLab, Bitbucket (public repos) |
| **Archives** | Wayback Machine, cached pages |
| **Leaked data** | Breach databases available publicly |

### 1.4 What OSINT is NOT

| Misconception | Reality |
|---|---|
| OSINT = Hacking | OSINT never touches a target system. You observe only. |
| OSINT = Stalking | Legal OSINT has a legitimate purpose (security, journalism, research) |
| Finding something = allowed to use it | Found data does not grant permission to access systems with it |
| More tools = better results | One good question + two tools beats ten tools with no direction |

### 1.5 Who Uses OSINT?

```
Security Professionals   →  Pentesters, bug bounty hunters, red teams
Journalists              →  Investigative reporting, verifying sources
Law Enforcement          →  Criminal investigations, missing persons
Intelligence Agencies    →  National security, geopolitical analysis
Corporates               →  Competitive intelligence, due diligence
Researchers              →  Academic investigation
Lawyers                  →  Case evidence gathering
HR / Recruiters          →  Background verification
Fraud Investigators      →  Insurance, financial fraud
You (learning security)  →  All of the above, ethically
```

---

## 2. The OSINT Mindset

This is the most important section in Level 1. Tools change. Mindset does not.

### 2.1 Think Like an Investigator, Not a Tool Runner

A bad approach:
> "Let me run Sherlock, then run theHarvester, then run Subfinder."

A good approach:
> "What am I trying to find out? What is the most direct path to that answer?
>  What does each result tell me? Where does it point next?"

**The tool is a hammer. You still need to know where the nail is.**

### 2.2 The Pivot Mindset

Every piece of data you find is a **pivot point** — a doorway to more data.

```
Example pivot chain:

Company name
    │
    ▼
LinkedIn job postings → tech stack they use
    │
    ▼
Tech stack (Atlassian, AWS, Okta) → narrow attack surface
    │
    ▼
Employee names from LinkedIn
    │
    ▼
Email format from Hunter.io → firstname.lastname@company.com
    │
    ▼
Email in breach database → password hash
    │
    ▼
Username from email prefix → search social platforms
    │
    ▼
Social profiles → real name, photo, phone, location
```

Every answer raises new questions. Follow the thread.

### 2.3 Separate Collection from Analysis

A common mistake is analyzing while collecting — you find something interesting,
start thinking about what it means, lose track of what else you were gathering,
and end up with an incomplete picture.

**Collect first. Analyze after.**

```
Session structure:
  Hour 1 → Pure collection. Screenshot everything. Note sources. Don't conclude yet.
  Hour 2 → Review what you found. Identify gaps. Ask new questions.
  Hour 3 → Targeted collection to fill gaps.
  Hour 4 → Analysis and reporting.
```

### 2.4 The Two-Source Rule

**Never report a finding you can only verify from a single source.**

If only one website says something, it might be wrong, outdated, or fabricated.
Treat single-source findings as leads, not conclusions.

```
One source   →  Lead / hypothesis
Two sources  →  Probable
Three sources →  Confirmed (for OSINT purposes)
```

### 2.5 Archive Everything Immediately

The internet changes. Pages get deleted. Accounts get locked. Evidence disappears.

**As soon as you find something valuable:**
1. Take a screenshot with timestamp visible
2. Archive the URL at `archive.ph` or `web.archive.org/save/`
3. Note the tool / search that found it
4. Save to your documentation system

If you don't archive it now, it may not exist in an hour.

### 2.6 Confirm, Don't Confirm Bias

The human brain finds patterns even where none exist. When you form a hypothesis
early, you unconsciously look for evidence that supports it and ignore evidence
against it. This is called **confirmation bias** and it ruins investigations.

```
Wrong approach:  "I think this person is X. Let me find evidence of that."
Right approach:  "Here is what I found. What does it actually support?"
```

Always actively look for evidence that CONTRADICTS your hypothesis. If your conclusion
survives that test, you can trust it. If it doesn't, revise.

---

## 3. The OSINT Framework

**URL:** https://osintframework.com

The OSINT Framework is a visual, interactive map of tools organized by category.
It was created by Justin Nordine and is updated by the community.

### 3.1 How to Use It

1. Open osintframework.com in a browser
2. Click any category (Username, Email, Domain Name, IP Address, Social Networks, etc.)
3. Subcategories expand with links to relevant tools
4. Use it as a **reference** when you know what you need to find but don't know which tool

### 3.2 Main Categories

```
OSINT Framework
│
├── Username
├── Email Address
├── Domain Name
├── IP Address
├── Social Networks
│   ├── Facebook
│   ├── Twitter
│   ├── LinkedIn
│   └── Instagram
├── Instant Messaging
├── People Search
├── Dating Sites
├── Telephone Numbers
├── Images / Videos / Docs
├── Maps / Geospatial
├── Archives
├── Language Translation
├── Metadata
├── Mobile Emulation
├── Threat Intelligence
└── Dark Web
```

### 3.3 Key Point

The framework lists hundreds of tools. You do not need to learn all of them.
**Focus on the 10–15 tools relevant to your investigation type. Depth beats breadth.**

---

## 4. Legal & Ethical Boundaries

Understanding the legal framework is not optional — it determines what you can
and cannot do. Crossing the line is not a mistake; it is a crime.

### 4.1 What is Always Legal

```
✅ Searching Google, Bing, DuckDuckGo
✅ Viewing public social media profiles
✅ Using Shodan, Censys, crt.sh (querying indexed public data)
✅ Accessing public government records
✅ Reading public GitHub repositories
✅ Checking WHOIS records
✅ Downloading publicly accessible files
✅ Reading archived pages on Wayback Machine
✅ Checking breach databases for your own email
```

### 4.2 What is Always Illegal

```
❌ Using found credentials to log into accounts you do not own
❌ Accessing private accounts, systems, or files without authorization
❌ Port scanning or active recon of systems you do not own
❌ Scraping platforms in ways that violate their Terms of Service (may be civil)
❌ Downloading and distributing breach data (receiving stolen data)
❌ Creating fake profiles to deceive someone into revealing private info
❌ Accessing someone's private messages, emails, or cloud storage
❌ Stalking, harassment, or surveillance of private individuals
```

### 4.3 Grey Areas

```
⚠️  "Forgot password" technique (viewing partial info platforms reveal)
     → Generally accepted in security research; not illegal in itself

⚠️  Sock puppet accounts
     → Creating fake accounts for research purposes
     → Permitted in security contexts if not used to deceive/defraud

⚠️  Aggregating public data into profiles of private individuals
     → Legal in most countries but violates GDPR in EU for private citizens

⚠️  Scraping public social media
     → Platforms ban it in ToS; some jurisdictions have ruled it legal (hiQ v. LinkedIn)
```

### 4.4 Country-Specific Laws

| Country | Key Law | Relevance to OSINT |
|---|---|---|
| **India** | IT Act 2000, Section 66 | Unauthorized computer access; Section 66C = identity theft |
| **India** | DPDP Act 2023 | Personal data protection; limits collecting and processing personal data |
| **USA** | CFAA (Computer Fraud and Abuse Act) | Unauthorized access to computers |
| **USA** | SCA (Stored Communications Act) | Accessing stored electronic communications |
| **EU** | GDPR | Strict limits on collecting/processing personal data of EU citizens |
| **UK** | Computer Misuse Act 1990 | Unauthorized access to computer systems |

> **India-specific note:** Section 66 of the IT Act 2000 makes unauthorized access a
> criminal offence with up to 3 years imprisonment. "Finding a door open" is not a
> defense. You need explicit authorization to access any system.

### 4.5 The Authorization Principle

A simple test for any OSINT action:

```
Ask yourself: "Do I have explicit permission to collect this information
               about this specific person or organization?"

If YES  → Proceed
If NO   → Is the data genuinely public and am I only observing?
  If YES  → Proceed with caution; do not access systems or private data
  If NO   → Stop. You are in illegal territory.
```

### 4.6 Bug Bounty Specific Rules

If you are doing OSINT for bug bounty:

```
1. Read the scope page completely before doing ANYTHING
2. Note the exact domains and IP ranges that are in scope
3. Note what is explicitly out of scope (employees, third parties, subprocessors)
4. Passive recon (Shodan, Google, crt.sh) is almost always permitted
5. Active scanning requires explicit permission in the scope
6. Never test on production user data
7. Report → wait for response → do not disclose publicly until resolved
```

---

## 5. Operational Security (OpSec) for OSINT

**OpSec** means protecting your own identity and intent while conducting an investigation.
If the target can see that someone is investigating them, they may delete evidence,
lock down accounts, or alert others.

### 5.1 Why OpSec Matters

```
Scenario: You visit a LinkedIn profile while logged into your account.
Result:   The target sees "Someone at [Your Company] viewed your profile."
          → They know they are being investigated.
          → They lock down their profiles and delete information.

Scenario: You visit the target's website from your home IP.
Result:   Their server logs record your IP address.
          → In a legal or corporate context, this creates a record.
          → In some contexts, it could constitute "contact" with the target.
```

### 5.2 The OpSec Stack

Build this before any serious investigation:

```
Layer 1 — Network (hide your IP)
    ├── VPN: Mullvad, ProtonVPN (no-logs, paid)
    └── Tor: tor-browser for maximum anonymity (slow but effective)

Layer 2 — Browser (isolated, clean)
    ├── Dedicated Firefox profile (NOT your daily browser)
    ├── No extensions that reveal identity (no password managers)
    ├── uBlock Origin (block tracking pixels and analytics)
    └── Firefox privacy settings: disable WebRTC, enable resist fingerprinting

Layer 3 — Accounts (not linked to you)
    ├── Sock puppet accounts on platforms you need to search
    ├── Created from a clean IP with a burner email
    └── Never accessed from your real accounts or devices

Layer 4 — Device (isolated environment)
    ├── VM (Virtual Machine) — recommended: Kali Linux or Ubuntu in VirtualBox
    └── Everything inside the VM stays inside the VM (snapshots for clean state)

Layer 5 — Identity (never cross-contaminate)
    ├── Never log into personal accounts during an investigation session
    ├── Never search your own name or details in the same session
    └── Use different passwords and emails for research accounts
```

### 5.3 Setting Up a Research Browser Profile (Firefox)

```
Step 1: Open Firefox → Menu → Manage Profiles → Create New Profile
        Name it: "OSINT-Research"

Step 2: Install only these extensions:
        - uBlock Origin (ad/tracker blocking)
        - Canvas Blocker (prevents fingerprinting)

Step 3: Change these settings (about:config):
        - privacy.resistFingerprinting → true
        - media.peerconnection.enabled → false  (disables WebRTC IP leak)
        - geo.enabled → false  (disables location sharing)

Step 4: Settings → Privacy & Security:
        - Enhanced Tracking Protection → Strict
        - Delete cookies and site data when Firefox is closed → ON
        - Do Not Track → Always

Step 5: Use this profile ONLY for OSINT. Never log into personal accounts.
```

### 5.4 Setting Up a VM (Virtual Machine)

A VM is the cleanest way to isolate your research environment.

```
Tools:
  VirtualBox (free) — virtualbox.org
  VMware Workstation Player (free for personal use)

Recommended OS for OSINT:
  Kali Linux — comes with many tools pre-installed
  Ubuntu/Debian — lighter; install tools manually
  TAILS OS — amnesiac OS; leaves no trace on the host machine

Setup steps:
  1. Download VirtualBox from virtualbox.org
  2. Download Kali Linux ISO from kali.org/get-kali
  3. Create new VM → 4GB+ RAM, 50GB+ disk → attach Kali ISO
  4. Install Kali → take a "clean" snapshot before installing any tools
  5. Do all research inside the VM
  6. Revert to snapshot to wipe any traces after sensitive sessions
```

### 5.5 Creating a Sock Puppet Account

A **sock puppet** is a research identity — a fake persona used to access platforms
without revealing your real identity.

```
Step 1: Generate a fake identity
        - Name: use a name generator (fakenamegenerator.com)
        - Face: thispersondoesnotexist.com (AI-generated face, no real person)
        - Backstory: age, city, interests — keep it simple and consistent

Step 2: Create a burner email
        - ProtonMail or Tutanota (no phone required)
        - Create over VPN or Tor
        - Never access from your real IP

Step 3: Create platform accounts
        - Use the burner email
        - Use VPN (different from your main VPN if possible)
        - Age the account: post generic content for 1–2 weeks before researching

Step 4: Never cross-contaminate
        - Never follow your real accounts
        - Never use your real phone number for verification
        - Use a disposable number (TextNow, Google Voice) if SMS is needed

Legal reminder: Sock puppets for research purposes are accepted in security contexts.
               Using them to deceive, defraud, or harass people is illegal.
```

### 5.6 Archiving During Research

Every important finding must be archived the moment you find it.

```
Tools:
  archive.ph      → Save a live URL to a permanent archive
  web.archive.org/save/<URL>  → Wayback Machine on-demand save
  Hunchly (Chrome extension)  → Auto-captures every page you visit
  Flameshot (Linux)           → Screenshot with annotation
  ShareX (Windows)            → Screenshot + auto-upload

Naming convention for screenshots:
  YYYY-MM-DD_HHMM_Source_Finding.png
  Example: 2026-10-04_1130_LinkedIn_TargetName_JobHistory.png

Documentation template (one entry per finding):
  Date/Time:    2026-10-04 11:30 IST
  Source:       LinkedIn
  URL:          https://linkedin.com/in/targetname
  Tool Used:    Manual browser (OSINT Firefox profile)
  Finding:      Subject works at Company X as Senior DevOps Engineer since 2023
  Archived:     https://archive.ph/XXXXX
  Pivot leads:  Search GitHub for targetname, check Hunter.io for email format
```

---

## 6. Documentation Systems

A well-documented investigation is a professional one. Findings that are not documented
did not happen — you cannot act on, share, or verify what exists only in your head.

### 6.1 CherryTree

**What it is:** A hierarchical, tree-structured note-taking application. Widely used by
penetration testers and OSINT investigators.

```
Install:
  sudo apt install cherrytree       (Debian/Kali)
  or download from giuspen.com/cherrytree

Structure for an investigation:
  📁 Target: Company / Person Name
  ├── 📄 Overview (who, what, why, scope)
  ├── 📁 Infrastructure
  │   ├── 📄 Domains & Subdomains
  │   ├── 📄 IP Addresses & ASN
  │   └── 📄 Shodan Results
  ├── 📁 People
  │   ├── 📄 Employees
  │   └── 📄 Social Media
  ├── 📁 Code & Leaks
  ├── 📄 Timeline
  └── 📄 Summary & Findings
```

### 6.2 Obsidian

**What it is:** A modern markdown note-taking tool with a **graph view** that shows
relationships between notes — ideal for OSINT because you can visually map connections.

```
Download: obsidian.md (free, local storage)

Setup for OSINT:
  Create a vault per investigation
  Each entity (person, domain, IP, company) → one note
  Link between notes: [[John Smith]] works at [[Target Corp]]
  Graph view shows you the relationship map visually

Plugin: Dataview → query your notes like a database
Plugin: Kanban → track investigation status
```

### 6.3 Maltego (Visual Link Analysis)

**What it is:** A professional-grade link analysis tool. Creates visual entity graphs
connecting people, domains, emails, IPs, organizations, and more.

```
Download: maltego.com
Community Edition: free (limited transforms, results capped at 12 per transform)
Professional: paid

Use it when:
  - You have many related entities and need to see relationships
  - You want to run automated transforms (OSINT queries via built-in integrations)
  - You need a deliverable graph for a report

Example: Add a domain entity → Run transforms → 
         See linked IPs, emails, subdomains, MX records all connected visually
```

---

## 7. Key Concepts Glossary

| Term | Definition |
|---|---|
| **OSINT** | Open Source Intelligence — intelligence from public sources |
| **Pivot** | Using one data point to find another, related data point |
| **Sock Puppet** | A fake online identity used for research purposes |
| **OpSec** | Operational Security — protecting your identity and methods |
| **EXIF** | Metadata embedded in image files (GPS, camera, date) |
| **ASN** | Autonomous System Number — identifies a network block |
| **CT Log** | Certificate Transparency Log — public record of all TLS certs issued |
| **Passive Recon** | Gathering information without touching the target's systems |
| **Active Recon** | Directly interacting with target systems (requires authorization) |
| **Dork** | A specialized search query using advanced operators |
| **GHDB** | Google Hacking Database — library of proven dork queries |
| **Banner** | The response a service sends when connected to — contains version info |
| **Footprint** | The digital trail left by your investigation activities |
| **Intelligence** | Raw data that has been analyzed and given meaning |
| **Attribution** | Linking an action, account, or infrastructure to a specific person/org |
| **OPSEC failure** | Accidentally revealing your identity or methods to the target |

---

## 8. Level 1 Practice Tasks

Complete these before moving to Level 2. They require no tools — only a browser.

### Task 1 — Build Your Research Browser Profile
- Set up a dedicated Firefox profile as described in Section 5.3
- Verify WebRTC is disabled (test at browserleaks.com/webrtc)
- Verify no personal accounts are accessible in that profile

### Task 2 — Explore the OSINT Framework
- Visit osintframework.com
- Click through at least 5 categories
- Write down 3 tools you had not heard of before

### Task 3 — Understand the Intelligence Cycle
- Pick any current news investigation (a fraud case, a missing person, a corporate scandal)
- Write a one-page breakdown of it using the intelligence cycle:
  - What was the question? (Planning)
  - What sources were used? (Collection)
  - How was the data processed? (Processing)
  - What conclusions were drawn? (Analysis)
  - How was it published? (Dissemination)

### Task 4 — Audit Your Own Digital Footprint
- Google your own full name
- Google your email address
- Google your phone number
- Check your email at haveibeenpwned.com
- Write down everything you find — this is exactly what an investigator sees about you

### Task 5 — Read One Bellingcat Investigation
- Visit bellingcat.com and read one published investigation
- Note every OSINT technique they used (Google Maps, satellite imagery, social media, etc.)
- Identify where each technique fits in the map you have

---

## 9. Level 1 Summary

```
✅ OSINT = legally gathering publicly available information
✅ Intelligence cycle: Plan → Collect → Process → Analyze → Disseminate
✅ Mindset is more important than tools
✅ Pivot constantly — every data point opens new doors
✅ Separate collection from analysis
✅ Archive everything immediately
✅ Never access systems; observe only
✅ Know the laws in your jurisdiction
✅ Build your OpSec stack before any investigation
✅ Document everything in a structured system

When you are done here → Proceed to Level 2: Person OSINT
```

---

## 10. Resources for Level 1

| Resource | Type | URL |
|---|---|---|
| OSINT Framework | Reference | osintframework.com |
| Bellingcat How-To | Articles | bellingcat.com/resources/how-tos |
| OSINT Curious | Community | osintcurio.us |
| Michael Bazzell — OSINT Techniques Book | Book | inteltechniques.com |
| Trace Labs | CTF Practice | tracelabs.org |
| TraceLabs Workflow | Guide | tracelabs.org/getinvolved |
| Firefox Privacy Settings | Guide | privacyguide.io |
| VirtualBox | Tool | virtualbox.org |
| Kali Linux | OS | kali.org |
| Hunchly | Tool | hunch.ly |
