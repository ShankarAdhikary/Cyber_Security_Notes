# OSINT — Level 10: Advanced OSINT Techniques
> Level 10 is where individual skills fuse into a single investigative
> discipline. Advanced OSINT is not about knowing more tools — it is
> about pivoting faster, correlating deeper, operating more covertly,
> thinking several steps ahead, and countering OSINT being run against
> you. This level covers the techniques that separate a competent
> practitioner from an expert investigator.

---

## 1. The Pivot Mindset — Advanced Correlation

### 1.1 The Six Degrees of Infrastructure

Every digital asset is connected to every other asset through at most
six pivot steps. The expert finds the shortest path.

```
Starting point: a company name
    │
    ▼ Reverse WHOIS → all domains
    │
    ▼ CT logs → all subdomains + SANs
    │
    ▼ SSL cert fingerprint → other servers using same cert
    │
    ▼ Favicon hash → related servers not linked to the domain
    │
    ▼ ASN → full IP range
    │
    ▼ Shodan host data → services, banners, open ports
    │
    ▼ Banner strings → other servers with identical banners
    │
    ▼ Cloud bucket naming → S3/GCS assets
    │
    ▼ GitHub org → leaked secrets → internal hostnames
    │
    ▼ Internal hostname → back to DNS → new subnet
```

### 1.2 Pivot Types Reference

```
From a DOMAIN:
  → WHOIS email         →  Reverse WHOIS → more domains
  → NS record           →  Other domains on same nameserver
  → MX record IP        →  Mail infrastructure
  → CT SANs             →  All certs covering this domain
  → SSL org field       →  All certs issued to this org
  → Favicon hash        →  All servers running same app
  → Google Analytics ID →  All sites sharing the same GA ID
  → AdSense ID          →  All sites sharing the same AdSense account

From an IP:
  → Reverse DNS         →  Hostname → domain → recon chain
  → ASN                 →  Full IP range
  → Co-hosted domains   →  Bing ip: / Shodan hostname:
  → SSL certs served    →  Domains on this IP
  → Banner              →  Product version → CVE search
  → BGP community       →  Network policy + upstream

From an EMAIL:
  → Domain part         →  Organisation domain
  → Username part       →  Social media search (Sherlock)
  → Breach databases    →  Password pattern, phone, username
  → GHunt (if Gmail)    →  Google account → Maps reviews → YouTube
  → Gravatar hash       →  Profile picture → reverse image search
  → Hunter.io           →  Colleagues at same domain

From a PERSON:
  → Full name           →  LinkedIn, court records, MCA director
  → Photo               →  Reverse image → other profiles
  → Username            →  WhatsMyName → all platforms
  → Phone               →  Truecaller, WhatsApp profile
  → Employer            →  Org OSINT chain
  → Location tags       →  GeoINT → home/work location
  → Writing style       →  Stylometric attribution

From a CERTIFICATE:
  → Serial number       →  All domains on the same cert (SANs)
  → Fingerprint         →  All servers using identical cert
  → Issuer              →  CA being used → cost/maturity signal
  → Valid from date     →  When was this deployed?
  → Org field           →  All certs issued to this org name

From a FAVICON:
  → MurmurHash3         →  All servers running same application
  → App identification  →  Technology fingerprint
  → Shodan search       →  Global exposure of that app
```

---

## 2. Shared Attribute Pivoting

### 2.1 Google Analytics / Tag Manager ID Pivoting

Every Google Analytics property has a unique tracking ID (UA-XXXXXXX-X
or G-XXXXXXXXXX). Sites sharing the same ID are operated by the same entity.

```bash
# Step 1: Find the GA ID in page source
curl -s https://example.com | grep -oE "(UA-[0-9]+-[0-9]+|G-[A-Z0-9]+)"

# Step 2: Search for other sites using the same ID

# BuiltWith:
# builtwith.com/relationships/tag/UA-12345678-1
# → Lists all sites using that GA ID

# SpyOnWeb:
# spyonweb.com/UA-12345678-1
# → Shows all sites sharing this analytics ID

# Publicwww:
# publicwww.com/websites/UA-12345678-1/
# → Searches HTML source across the web for this ID
# Free: 10 searches/day

# DNSlytics:
curl "https://dnslytics.com/reverse-analytics?q=UA-12345678-1"

# RelatedWebsites:
# relatedwebsites.io → enter any domain → see related sites by shared IDs
```

### 2.2 AdSense / Publisher ID Pivoting

```bash
# Find AdSense publisher ID in page source
curl -s https://example.com | grep -oE "pub-[0-9]{16}"

# Search other sites with same AdSense ID
# SpyOnWeb: spyonweb.com/pub-1234567890123456
# PublicWWW: publicwww.com/websites/pub-1234567890123456/

# Intelligence value:
# Sites sharing AdSense IDs = monetized by the same person/org
# Links seemingly unrelated sites to a common operator
```

### 2.3 Nameserver Pivoting

```bash
# Find nameservers for a domain
dig NS example.com +short
# ns1.examplehost.com
# ns2.examplehost.com

# Find all OTHER domains using the same nameservers
# Bing: search "ns1.examplehost.com" (shows domains linking to it)
# SecurityTrails API:
curl "https://api.securitytrails.com/v1/search/list?apikey=KEY" \
  -H "Content-Type: application/json" \
  -d '{"filter":{"apex_domain":"example.com","ns":["ns1.examplehost.com"]}}'

# ViewDNS Reverse NS lookup:
# viewdns.info/reversens/?ns=ns1.examplehost.com
# → Returns all domains using those nameservers

# Intelligence value:
# All domains on the same custom nameservers = same operator
# Especially useful when nameservers are not a public provider
# (e.g., not Cloudflare/Route53 but a custom ns1.targetcorp.com)
```

### 2.4 SSL Certificate Serial / Fingerprint Pivoting

```bash
# Get cert fingerprint from a live site
echo | openssl s_client -connect example.com:443 2>/dev/null \
  | openssl x509 -noout -fingerprint -sha256

# Search Shodan for all hosts using the same cert fingerprint
shodan search "ssl.cert.fingerprint:AA:BB:CC:DD:..."

# Get cert serial number
echo | openssl s_client -connect example.com:443 2>/dev/null \
  | openssl x509 -noout -serial

# Search crt.sh for same serial (finds all domains on that cert)
curl "https://crt.sh/?serial=SERIALNUMBER&output=json"

# Intelligence value:
# Multi-domain certificates expose all domains the operator runs
# A self-signed cert with the same fingerprint on multiple IPs
#   → same server or same infrastructure operator
```

### 2.5 IP Co-Hosting Pivot

```bash
# Find all domains hosted on the same IP address

# Bing ip: operator
# bing.com/search?q=ip:93.184.216.34

# Shodan (shows hostnames on an IP)
shodan host 93.184.216.34 | grep "Hostnames"

# SecurityTrails
curl "https://api.securitytrails.com/v1/ips/list?apikey=KEY" \
  -H "Content-Type: application/json" \
  -d '{"filter":{"ipv4":"93.184.216.34"}}'

# ViewDNS Reverse IP
# viewdns.info/reverseip/?host=93.184.216.34

# HackerTarget
curl "https://api.hackertarget.com/reverseiplookup/?q=93.184.216.34"

# Intelligence value:
# Shared hosting → lower-value targets on the same IP
# Dedicated IP with multiple hostnames → same operator's assets
# CDN IP → do not pivot (Cloudflare IP hosts millions of sites)
#   First check: is this a CDN IP? → if yes, skip IP pivot
```

---

## 3. Sock Puppet Advanced Operations

### 3.1 Building a Credible Persona

A sock puppet for serious research needs depth — an inconsistent or thin
persona is easily detected and blocked.

```
Identity Construction:

1. Generate base identity
   → Name: fakenamegenerator.com (consistent name + backstory)
   → Face: thispersondoesnotexist.com (AI-generated, unique each refresh)
   → Age: 25-35 (most active social demographic)
   → Location: a real city, but not yours

2. Build a consistent backstory
   → Occupation: something generic (teacher, freelancer, small business owner)
   → Interests: 2-3 hobbies relevant to the investigation context
   → Keep it simple — inconsistency kills sock puppets

3. Create infrastructure
   → Burner email: ProtonMail or Tutanota (from clean IP/Tor)
   → Phone (if needed): Google Voice, TextNow, or a burner SIM
   → Device: separate browser profile or VM (never your real machine)
   → VPN: different provider from your main VPN
   → Payments: prepaid card (bought with cash) if any paid service needed

4. Age the account
   → Create accounts 2-4 weeks before you need them
   → Post generic, non-investigative content
   → Follow unrelated public accounts
   → Build a follower base (follow back communities, etc.)
   → Engage with content that matches the persona's interests

5. Compartmentalization rules
   → NEVER follow your own real accounts from the puppet
   → NEVER log in from your home IP
   → NEVER use the same device as your real accounts
   → NEVER cross the streams (real identity + puppet in same session)
```

### 3.2 Sock Puppet Detection Avoidance

```
What platforms use to detect sock puppets:
  → IP address and ISP matching
  → Device fingerprint (browser, OS, screen resolution, fonts)
  → Behavioral patterns (posting times, writing style)
  → Network graph analysis (who you follow/interact with)
  → Account age and activity history
  → Phone number reuse

Countermeasures:
  IP:          Use a different VPN exit node per persona
  Fingerprint: Firefox + CanvasBlocker extension; different screen res in VM
  Behavior:    Post at times consistent with persona's time zone
  Network:     Do not follow/interact with other puppets or your real accounts
  Age:         Create accounts long before you need them
  Phone:       Each persona gets a different number (TextNow gives free US numbers)

Browser fingerprinting check:
  browserleaks.com — test your current fingerprint uniqueness
  coveryourtracks.eff.org — EFF browser fingerprint test
```

---

## 4. Stylometry — Writing Style Analysis

### 4.1 What Stylometry Is

Stylometry is the statistical analysis of writing style to attribute
anonymous text to a known author. It is used to:

```
→ Link anonymous social media accounts to known identities
→ Attribute darkweb forum posts to a known suspect
→ Compare two sets of writing to confirm or deny same author
→ Identify ghost-written content
→ Academic plagiarism detection
```

### 4.2 Observable Style Markers

```
Lexical features:
  ├── Average word length
  ├── Vocabulary richness (type-token ratio)
  ├── Function word frequencies (the, of, and, to, a...)
  └── Rare word usage

Syntactic features:
  ├── Average sentence length
  ├── Punctuation patterns
  ├── Capitalization habits
  └── Grammar patterns

Character-level features:
  ├── Spelling errors (consistent typos = fingerprint)
  ├── Abbreviations (gonna, wanna, u, ur, bc)
  ├── Emoji and emoticon patterns
  └── Double spacing after periods

Content features:
  ├── Recurring phrases or idioms
  ├── Domain-specific vocabulary
  └── Cultural references (implies background/education)
```

### 4.3 Tools

```
JGAAP (Java Graphical Authorship Attribution Program):
  URL: github.com/evllabs/JGAAP
  Desktop Java application
  Import: known writing samples + anonymous text
  Run: multiple attribution algorithms
  Output: confidence percentage for each candidate author

Stylometrica: stylometrica.eu
  Web-based stylometric analysis
  Upload text → compare against other samples

NLTK (Python Natural Language Toolkit):
  pip3 install nltk
  # Build custom feature extractors
  # Used for bulk analysis of large corpora

Manual pattern analysis:
  Collect 500+ words from the known author and the anonymous text
  Compare:
    → Unique phrases (exact wording reuse)
    → Error patterns (same misspelling in both)
    → Punctuation habits (serial comma? em dash?)
    → Sentence structure preferences (short punchy? long complex?)
    → Vocabulary overlap (word choice for the same concepts)
```

### 4.4 Practical Example

```
Scenario: An anonymous Twitter account @anon_researcher posts about
          a specific company. You suspect it is an employee named John Doe.

Step 1: Collect writing samples
  → @anon_researcher: 200 tweets (copy to text file)
  → John Doe's LinkedIn posts + public GitHub commits (copy to text file)

Step 2: Look for unique patterns
  → Both use "tbh" (to be honest) frequently
  → Both say "quite literally" as an intensifier
  → Both consistently miss the Oxford comma
  → Both spell "definately" (wrong — should be "definitely")
  → Both write "per se" correctly in technical contexts

Step 3: JGAAP analysis
  → Import known (Doe) and unknown (anon) corpora
  → Run Burrows's Delta algorithm
  → Result: 91% confidence same author

Step 4: Corroborate with other OSINT
  → @anon_researcher's timezone matches Doe's location
  → Posting times align with after-hours for that timezone
  → Topics discussed match Doe's specific role at the company
```

---

## 5. Breach Data Analysis — Advanced

### 5.1 Working with Large Breach Datasets (Legally)

```
Legal note: Downloading, storing, and distributing breach data is
illegal in most jurisdictions regardless of how it was obtained.

Legal uses of breach intelligence:
  ✅ Querying via breach search services (HIBP, DeHashed) — you query, don't store
  ✅ Using breach notifications about YOUR OWN data
  ✅ Authorized corporate security work (checking employee exposure)
  ✅ Academic research under institutional ethics approval

What breach data reveals (from authorized search results):
  → Password patterns for credential guessing → ANALYSIS ONLY, NOT TESTING
  → Associated usernames → pivot to social media search
  → IP addresses at registration → geolocation intelligence
  → Phone numbers → pivot to Truecaller / Phoneinfoga
  → Security questions/answers → often contain real personal info
  → Physical addresses → confirm or discover location
```

### 5.2 Password Pattern Analysis

```
Plaintext password analysis (from authorized breach reports):
Purpose: understand patterns, NOT to test against live systems

Common patterns:
  "CompanyName2024!"   → organization + year + symbol
  "John1990@"          → name + birth year + symbol
  "Mumbai#123"         → city + symbol + sequence
  "Qwerty123@"         → keyboard walk + number + symbol
  "P@ssword1"          → password with leet speak

What patterns tell you:
  → Name in password → confirms real name or alias
  → Year in password → birth year or year of account creation
  → City in password → likely location at time of account creation
  → Company name → confirms employer or affiliations
  → Same base word across multiple breaches → predictable mutation pattern

Password mutation prediction:
  Breach 2020: "Mumbai2020!"
  Predicted 2026: "Mumbai2026!" or "Mumbai26!"
  → Document this as analysis — never test
```

### 5.3 Credential Stuffing Context (for Defenders)

```
When doing authorized security assessments:
  1. Check client's email domains against HIBP API
  2. Report: how many employees have breached credentials?
  3. Which services were breached (social media, shopping sites, etc.)?
  4. Recommend: mandatory password reset + MFA for affected accounts

HIBP API (for authorized use):
  curl "https://haveibeenpwned.com/api/v3/breachedaccount/target@company.com" \
    -H "hibp-api-key: YOUR_KEY"

  # Returns: list of breach names for that email
  # Does NOT return passwords

  # Bulk check (Enterprise):
  curl "https://haveibeenpwned.com/api/v3/breachedaccount/{email}" \
    -H "hibp-api-key: YOUR_KEY"
```

---

## 6. Infrastructure Pivoting via Unique Identifiers

### 6.1 JA3/JA3S TLS Fingerprinting

```
JA3 is a method to fingerprint TLS client hellos. JA3S fingerprints
the server's TLS hello response. Different software has different JA3S values.

Why it matters:
  → C2 frameworks (Cobalt Strike, Metasploit) have known JA3S fingerprints
  → Finding all servers with the same JA3S = finding all instances of that C2
  → Useful for identifying malicious infrastructure

Shodan search by JA3S:
  shodan search 'ssl.jarm:"JARM_HASH_VALUE"'

JARM (Salesforce):
  → JARM is an active TLS fingerprinting tool
  → It sends 10 probes and records the server's responses
  → Result: a unique fingerprint for each TLS implementation
  pip3 install jarm-fingerprinter
  jarm_fingerprinter example.com 443

# Find all Cobalt Strike servers:
  shodan search 'ssl.jarm:"2ad2ad0002ad2ad00042d42d0000002059c5c6548571d8a7c89e14d2c2d2d"'
```

### 6.2 JARM for Malicious Infrastructure Hunting

```bash
# Step 1: Get JARM fingerprint of a known malicious server
# (from threat intelligence reports — Cobalt Strike has published JARMs)

# Cobalt Strike default JARM:
# 07d14d16d21d21d00042d43d000000aa99ce74e2c6d013c745aa52b5cc042d

# Step 2: Hunt for all instances on Shodan
shodan search 'ssl.jarm:"07d14d16d21d21d00042d43d000000aa99ce74e2c6d013c745aa52b5cc042d"'

# Step 3: Filter to find any overlap with your target's IP ranges
shodan search 'ssl.jarm:"HASH" net:104.16.0.0/16'

# Intelligence value:
# Your target's IP range hosts a Cobalt Strike C2 server
# → Either they were compromised OR they are a threat actor
```

### 6.3 HTTP Response Header Fingerprinting

```bash
# Collect HTTP response headers from a server
curl -I https://example.com

# Unique header combinations fingerprint a specific web framework or app
# Example — a custom header reveals a proprietary system:
# X-Powered-By: AcmeCorp-Intranet/2.3.1
# → Search Shodan for this exact header → find all instances globally

shodan search '"X-Powered-By: AcmeCorp-Intranet"'

# Server header quirks:
# Server: Apache/2.4.51 (Debian)  → standard
# Server: Apache  → deliberately obscured (security-conscious)
# Server: (missing entirely) → deliberately hidden (more security-conscious)
# Server: nginx/1.21.6 + custom headers → nginx with custom config

# Track custom error pages:
# A unique 404 error page design is a fingerprint
# Reverse image search the 404 page → find all sites with same design
# → May reveal sites operated by the same developer/hosting company
```

---

## 7. Dark Web OSINT (Advanced)

### 7.1 When to Go to the Dark Web

```
The dark web is NOT the primary source for most OSINT.
Go there only when:
  → Investigating threat actors operating on Tor
  → Looking for leaked data posted on dark web forums
  → Tracking ransomware groups and their victims
  → Investigating darknet market activity

Surface web tools first (cover 80% of needs):
  → Ahmia.fi (Tor search, no Tor needed)
  → IntelligenceX (indexes dark web without Tor)
  → DarkSearch.io (dark web search)
  → Ransomware.live (tracks ransomware leak sites)
```

### 7.2 Safe Dark Web Research Setup

```
Required setup for Tor-based research:
  1. Dedicated VM (Tails OS preferred — amnesiac, leaves no traces)
     OR: Whonix OS (routes all traffic through Tor by design)
  2. Never use your real machine
  3. Never open files downloaded from dark web on your real machine
     (PDFs, Word docs can beacon back with your real IP)
  4. Never take screenshots with your name/details visible
  5. Never log in to any personal accounts
  6. Never enable JavaScript (Settings → Privacy → strictest mode)
  7. Use Tor Browser only — never your regular browser with Tor proxy

Tails OS:
  Download: tails.boum.org
  Boot from USB → leaves zero traces on the host machine
  All traffic routes through Tor automatically
```

### 7.3 Dark Web Monitoring Tools

```
Ransomware.live:    ransomware.live
  → Aggregates victim lists from known ransomware groups' leak sites
  → Updated in near-real-time
  → Search by: company name, country, sector
  → No Tor needed — surface web tracker

  API:
  curl "https://api.ransomware.live/victims"
  curl "https://api.ransomware.live/victims/search?q=targetcorp"

DarkFeed:           darkfeed.io (commercial)
  → Real-time dark web monitoring

Flare:              flare.io (commercial)
  → Monitors dark web forums, paste sites, Telegram for brand mentions

Manual dark web search (via Tor):
  Ahmia:      ahmia.fi (also works as: msydqstlz2kzerdg.onion)
  Torch:      xmh57jrknzkhv6y3ls3ubitzfqnkrwxhopf5ayieonly2jyniy5g2qxyd.onion
  Haystak:    haystak5njsmn2hqkewecpaxetahtwhsbsa64jom2k22z5afxhnpxfid.onion
  DuckDuckGo: duckduckgogg42ts2pqs62a62gam2i3fdm7gyittiqcqnbmtmh5ub4id.onion
```

### 7.4 Telegram OSINT

```
Telegram is increasingly the communication platform for threat actors,
activists, journalists, criminal groups, and leakers.

Searching without an account:
  tgscan.me         → search Telegram channels/groups by keyword
  lyzem.com         → Telegram search engine
  telegago.com      → Telegram search
  telegramchannels.me → channel directory

With a Telegram account:
  telegram.me/s/<channel_name>  → view public channel without app
  In Telegram app: search by keyword → finds public groups and channels

IntelligenceX (intelx.io):
  → Indexes Telegram messages from public channels
  → Search by: username, keyword, phone number

What to look for:
  → Leaked data dumps shared in channels
  → Threat actor communications
  → Coordination channels for groups
  → Usernames → pivot to other platforms

Phone number → Telegram account:
  Add the phone number to your contacts (in Telegram)
  → If they have Telegram → their profile appears (username, picture)
  → Username → search across all platforms
```

---

## 8. OSINT Countering (Defensive)

### 8.1 Auditing Your Own Footprint

```
Run OSINT on yourself quarterly. What investigators find = what attackers see.

Personal footprint audit:
  □ Google your full name (with and without quotes)
  □ Google your email addresses
  □ Google your phone numbers
  □ Check your email on HIBP and DeHashed
  □ Search username on Sherlock
  □ Run reverse image search on your profile photos
  □ Check LinkedIn for information accuracy
  □ Review all social media privacy settings
  □ Search your home address on people-search sites
  □ Check court records for your name
  □ Review what your domain WHOIS reveals

Organizational footprint audit:
  □ Run subdomain enumeration on your domains
  □ Check crt.sh for all issued certificates
  □ Search GitHub for your org name + sensitive keywords
  □ Check Shodan for exposed services on your IP ranges
  □ Run metagoofil on your domain for public documents
  □ Check paste sites: site:pastebin.com "yourdomain.com"
  □ Review all MX/SPF/DMARC records
  □ Check for S3 buckets with your company name
  □ Search for your brand in ransomware.live
```

### 8.2 Reducing Your OSINT Surface

```
Domain & Infrastructure:
  → Enable WHOIS privacy (registrar privacy protection)
  → Use Cloudflare or similar CDN to mask origin IP
  → Remove version numbers from server headers
    nginx: server_tokens off;
    Apache: ServerTokens Prod; ServerSignature Off;
  → Use generic nameservers (Cloudflare, Route53) not custom ones
  → Remove metadata from all public documents before publishing
    exiftool -all= document.pdf
  → Disable directory listing on web servers
  → Review robots.txt — don't list sensitive paths (they advertise them)
  → Enable DMARC with p=reject to prevent domain spoofing
  → Set GitHub repos to private by default

Personal:
  → Opt out of people-search sites (takes time but is possible):
    spokeo.com/optout
    whitepages.com/suppression_requests
    intelius.com/optout
    beenverified.com/app/optout/
  → Use a PO Box or registered address service instead of home address
  → Use WHOIS privacy for all personally registered domains
  → Use email aliases (SimpleLogin, AnonAddy) for online accounts
  → Keep social media profiles minimal and private
  → Avoid geotagging photos before posting
  → Remove EXIF from photos: exiftool -all= photo.jpg
  → Use a VPN for general browsing (reduces IP-based geolocation)

Social Engineering Hardening:
  → Train employees not to disclose tech stack in public (LinkedIn, Twitter)
  → Sanitize job postings — "experience with X" reveals your exact tooling
  → Use generic email patterns (avoid firstname.lastname@)
  → Restrict who can see employee list on LinkedIn company page
```

### 8.3 OSINT Deception (Advanced)

```
Honey tokens:
  → Fake AWS credentials in a fake .env file on GitHub
  → When accessed → alert fires → attacker is detected
  → Canary tokens: canarytokens.org (free)
    Types: URL, email, Word doc, PDF, AWS key, DNS name
    When the token is triggered → you receive an alert with the attacker's IP

  Example:
  1. Generate a Canary Token AWS key at canarytokens.org
  2. Commit it to a public GitHub repo in a file called config.py
  3. When an attacker finds it and tries to use it → alert fires
  4. Alert contains: attacker's IP, time, browser fingerprint

Honeypot subdomains:
  → Create a subdomain like dev-internal.example.com
  → No real service → just a logging endpoint
  → Any HTTP request to it = an attacker is probing your infrastructure
  → Cloudflare Workers can log these and send Slack alerts

Fake documents with tracking:
  → Create a PDF with an embedded tracking pixel or hyperlink
  → Seed it in a location where attackers might find it (a "leaked" document)
  → When opened → tracking pixel fires → reveals opener's IP and device
  → Tools: canarytokens.org (Word/PDF tokens)
```

---

## 9. Advanced Infrastructure Intelligence

### 9.1 BGP Hijacking Detection

```
BGP hijacking is when an AS incorrectly announces ownership of IP prefixes
it doesn't own, causing traffic to be rerouted.

Detection tools:
  BGPMon:    bgpmon.net
  RIPE RIS:  ris.ripe.net
  Cloudflare Radar: radar.cloudflare.com (BGP tab)

Check if your target's prefixes are being hijacked:
  curl "https://stat.ripe.net/data/bgp-state/data.json?resource=104.16.0.0/16"
  → Returns current BGP state for a prefix

Historical BGP data:
  BGPlay: stat.ripe.net/widget/bgplay
  → Visualizes how a prefix's routing has changed over time
```

### 9.2 Cloud Misconfiguration Patterns

```
AWS common misconfigurations to check:

S3 public buckets:
  curl -I https://bucket.s3.amazonaws.com
  aws s3 ls s3://bucket-name --no-sign-request

EC2 metadata exposure (via SSRF):
  http://169.254.169.254/latest/meta-data/
  http://169.254.169.254/latest/meta-data/iam/security-credentials/

Lambda function URLs exposed:
  https://abc123.lambda-url.us-east-1.on.aws/
  → Check if authentication is required

AWS Cognito misconfiguration:
  → User pools sometimes allow self-registration
  → Can grant access to authenticated-only resources

GCP misconfigurations:
  Google Cloud Storage:
    curl https://storage.googleapis.com/bucket-name
  Firebase open DBs:
    curl https://app.firebaseio.com/.json
  GCP metadata:
    http://metadata.google.internal/computeMetadata/v1/

Azure misconfigurations:
  Blob storage public containers:
    curl "https://account.blob.core.windows.net/container?restype=container&comp=list"
  Azure AD application secrets in repos:
    Search GitHub: "client_secret" "tenant_id" site:github.com
```

### 9.3 API Key Entropy Analysis

```python
#!/usr/bin/env python3
"""
Detect high-entropy strings (likely API keys/secrets) in text
"""
import math
import re
from typing import List, Tuple

def calculate_entropy(string: str) -> float:
    """Calculate Shannon entropy of a string."""
    if not string:
        return 0.0
    freq = {}
    for char in string:
        freq[char] = freq.get(char, 0) + 1
    entropy = 0.0
    for count in freq.values():
        prob = count / len(string)
        entropy -= prob * math.log2(prob)
    return entropy

def find_high_entropy_strings(text: str,
                              min_length: int = 20,
                              min_entropy: float = 4.0) -> List[Tuple[str, float]]:
    """Find potential secrets based on entropy analysis."""
    # Match strings that look like tokens/keys
    pattern = r'[A-Za-z0-9+/=_\-]{' + str(min_length) + r',}'
    candidates = re.findall(pattern, text)

    results = []
    for candidate in candidates:
        entropy = calculate_entropy(candidate)
        if entropy >= min_entropy:
            results.append((candidate, round(entropy, 2)))

    return sorted(results, key=lambda x: x[1], reverse=True)

# Known patterns for specific key types
KEY_PATTERNS = {
    "AWS Access Key":     r'AKIA[0-9A-Z]{16}',
    "AWS Secret Key":     r'[0-9a-zA-Z/+]{40}',
    "GitHub Token":       r'ghp_[0-9a-zA-Z]{36}',
    "GitHub App Token":   r'(ghu|ghs|ghr)_[0-9a-zA-Z]{36}',
    "GitLab Token":       r'glpat-[0-9a-zA-Z_-]{20}',
    "Slack Token":        r'xox[baprs]-[0-9a-zA-Z-]+',
    "Stripe Key":         r'(sk|pk)_(live|test)_[0-9a-zA-Z]{24}',
    "Sendgrid Key":       r'SG\.[0-9a-zA-Z]{22}\.[0-9a-zA-Z]{43}',
    "Twilio SID":         r'AC[0-9a-f]{32}',
    "JWT Token":          r'eyJ[A-Za-z0-9_-]+\.eyJ[A-Za-z0-9_-]+\.[A-Za-z0-9_-]+',
    "Generic API Key":    r'api[_-]?key[_-]?[=:]\s*["\']?([A-Za-z0-9_\-]{20,})',
    "Generic Secret":     r'secret[_-]?[=:]\s*["\']?([A-Za-z0-9_\-]{20,})',
    "Generic Password":   r'password[_-]?[=:]\s*["\']?([^\s"\']{8,})',
    "Generic Token":      r'token[_-]?[=:]\s*["\']?([A-Za-z0-9_\-\.]{20,})',
}

def scan_for_secrets(text: str) -> List[dict]:
    """Scan text for known secret patterns."""
    findings = []
    for key_type, pattern in KEY_PATTERNS.items():
        matches = re.findall(pattern, text, re.IGNORECASE)
        for match in matches:
            val = match if isinstance(match, str) else match[0]
            findings.append({
                "type": key_type,
                "value": val[:20] + "..." if len(val) > 20 else val,
                "entropy": calculate_entropy(val)
            })
    return findings

if __name__ == "__main__":
    import sys
    with open(sys.argv[1]) as f:
        content = f.read()

    print("=== Pattern-Based Findings ===")
    for finding in scan_for_secrets(content):
        print(f"[{finding['type']}] {finding['value']} (entropy: {finding['entropy']:.2f})")

    print("\n=== High Entropy Strings ===")
    for string, entropy in find_high_entropy_strings(content):
        print(f"[{entropy}] {string[:50]}")
```

---

## 10. OSINT for Threat Intelligence

### 10.1 Tracking Threat Actors

```
Threat actor OSINT combines:
  → Malware analysis (hashes, C2 IPs, domains)
  → Infrastructure analysis (IP ranges, hosting providers, cert patterns)
  → OPSEC failures (reused tools, usernames, languages)
  → Forum/dark web activity (vendor profiles, posts)
  → Attribution reports (government + vendor reports)

Key resources:
  MITRE ATT&CK:  attack.mitre.org
    → Database of TTPs (Tactics, Techniques, Procedures) by threat group
    → Search by technique → find which groups use it
    → Search by group → find their known infrastructure patterns

  MalpediaI:     malpedia.caad.fkie.fraunhofer.de
    → Malware family database
    → Each family linked to known actors

  Threat Intel Reports:
    → Mandiant/Google Threat Intelligence
    → CrowdStrike Intelligence
    → Microsoft MSTIC
    → Recorded Future (free intel reports)
    → Secureworks CTU Reports
    → All publish free IOC reports → download and correlate with your targets
```

### 10.2 IOC (Indicator of Compromise) Enrichment Pipeline

```bash
# Given a list of IOCs (IPs, domains, hashes) — enrich each one

# Install: pip3 install ioc-finder

# Enrich a domain IOC:
DOMAIN="malicious.example.com"

# VirusTotal
curl "https://www.virustotal.com/api/v3/domains/$DOMAIN" \
  -H "x-apikey: $VT_KEY" | python3 -m json.tool

# OTX
curl "https://otx.alienvault.com/api/v1/indicators/domain/$DOMAIN/general" \
  -H "X-OTX-API-KEY: $OTX_KEY"

# URLhaus
curl -X POST https://urlhaus-api.abuse.ch/v1/host/ -d "host=$DOMAIN"

# Shodan
shodan search "hostname:$DOMAIN" --fields ip_str,port,product

# Passive DNS (SecurityTrails)
curl "https://api.securitytrails.com/v1/domain/$DOMAIN" \
  -H "apikey: $ST_KEY"

# Combine all results into a single enrichment record
python3 << 'EOF'
import json, subprocess, os

def enrich_domain(domain):
    result = {"domain": domain, "sources": {}}

    # VirusTotal
    import requests
    vt = requests.get(
        f"https://www.virustotal.com/api/v3/domains/{domain}",
        headers={"x-apikey": os.environ.get("VT_KEY", "")}
    )
    if vt.status_code == 200:
        data = vt.json()
        stats = data.get("data",{}).get("attributes",{}).get("last_analysis_stats",{})
        result["sources"]["virustotal"] = {
            "malicious": stats.get("malicious", 0),
            "suspicious": stats.get("suspicious", 0),
        }

    return result

print(json.dumps(enrich_domain("example.com"), indent=2))
EOF
```

---

## 11. OSINT CTF Techniques

### 11.1 CTF-Specific Approaches

```
OSINT CTFs (Capture The Flag) test your ability to extract a specific
piece of information — a flag — from limited starting data.

Common challenge types:
  → Geolocate an image (find coordinates or a location name)
  → Find a person's home address from social media
  → Identify a vehicle from partial plate + location
  → Find when a photo was taken (timestamp)
  → Trace an online account to a real identity
  → Find what company was involved in an event

CTF mindset vs. real investigation:
  → CTF: one correct answer, time pressure, everything needed is findable
  → Real: multiple valid answers, incomplete data, ambiguity is normal

General CTF strategy:
  1. Extract EVERYTHING from the given material first
     (EXIF, visible text, background details, metadata)
  2. Build a hypothesis from the most distinctive feature
  3. Test it immediately — don't overthink
  4. If stuck, pivot to the next most distinctive feature
  5. Use reverse image search FIRST for image challenges
     (saves time — sometimes the location is directly findable)
  6. Google distinctive text visible in images
     (a sign, a shop name, a license plate format)
```

### 11.2 Top OSINT CTF Platforms

```
TraceLabs:         tracelabs.org
  → Missing persons OSINT (real cases — findings go to investigators)
  → Most impactful OSINT CTF
  → Team-based; judges score findings by quality

OSINT Dojo:        osintdojo.com
  → Beginner-friendly challenges

Gralhix / Sofia Santos: gralhix.com
  → High-quality exercises with solutions published

Bellingcat:        bellingcat.com/resources/quizzes/
  → Real-world investigative challenges

CyberDefenders:    cyberdefenders.org
  → Some OSINT challenges in Blue Team CTF format

Google CTF (misc): ctftime.org
  → Various CTFs include OSINT categories

Hack The Box:      hackthebox.com
  → Some OSINT challenges in their challenge lab
```

### 11.3 TraceLabs — Real-World Missing Persons OSINT

```
TraceLabs runs CTF competitions where participants search for
publicly available information on real missing persons cases.
Findings are submitted to law enforcement investigators.

How it works:
  1. Register at tracelabs.org
  2. Join a CTF event (several per year)
  3. Receive a list of missing persons (publicly available info)
  4. Research their last known location, contacts, online presence
  5. Submit findings through the TraceLabs platform
  6. Judges score based on quality and relevance of findings
  7. All findings go to the actual investigators

Why it matters:
  → Real practice with real data
  → Direct contribution to missing persons investigations
  → Highest standard of ethical OSINT
  → Strong community of practitioners
```

---

## 12. Building Your OSINT Knowledge Graph

### 12.1 Obsidian for Investigation Mapping

```
Obsidian (obsidian.md) is a markdown-based note-taking tool with
a graph view that shows connections between notes. Ideal for OSINT.

Structure for an investigation vault:
  /Entities/
    /Persons/
      John_Doe.md
      Jane_Doe.md
    /Domains/
      example.com.md
      targetcorp.net.md
    /IPs/
      93.184.216.34.md
    /Companies/
      TargetCorp.md
    /Emails/
      john.doe@example.com.md
  /Findings/
    Finding_ExposedAdmin_2026-10-15.md
    Finding_LeakedCredentials_2026-10-14.md
  /Timeline.md
  /Summary.md

Linking entities:
  In John_Doe.md:
    "John Doe works at [[TargetCorp]] and uses email [[john.doe@example.com]]"
    "His LinkedIn shows: [[example.com]] and [[targetcorp.net]]"

  In Obsidian graph view:
    → Every [[link]] creates a visible connection
    → Graph shows the full relationship web
    → Click any node → navigate to that entity's notes
    → Useful plugins: Dataview (query your notes), Kanban (track status)
```

### 12.2 OSINT Knowledge Base Template

```markdown
# Entity: [Name / Domain / IP]

## Type
Person / Company / Domain / IP / Email / Phone / etc.

## Status
Active investigation / Confirmed / Closed / Monitoring

## Summary
One paragraph of what this entity is and why it matters.

## Known Identifiers
- Domain: example.com
- IP: 93.184.216.34
- Email: john.doe@example.com
- Phone: +91-XXXXXXXXXX
- Username: johndoe1990
- ASN: AS15133

## Connections
- Works at: [[TargetCorp]]
- Associated with: [[example.com]]
- Known associate: [[Jane_Doe]]

## Findings Log
| Date | Tool | Finding | Source | Archived |
|---|---|---|---|---|
| 2026-10-14 | crt.sh | New subdomain dev.example.com | | archive.ph/XXX |
| 2026-10-13 | GitHub | AWS key in repo | | archive.ph/YYY |

## Open Questions
- [ ] Confirm home address
- [ ] Find phone number
- [ ] Verify employment at TargetCorp

## Sources
- [LinkedIn Profile](https://linkedin.com/in/johndoe) (archived: archive.ph/XXX)
- [WHOIS Record](https://whois.domaintools.com/example.com) (archived: archive.ph/YYY)
```

---

## 13. The Complete OSINT Professional Workflow

### 13.1 From Request to Report

```
Phase 0: Authorization & Scoping (BEFORE ANYTHING)
  □ Written authorization received and saved
  □ Scope defined: which domains, IPs, persons are in scope
  □ Out-of-scope items noted
  □ OpSec setup: VPN active, research browser profile open, VM ready
  □ Documentation system ready (CherryTree / Obsidian)
  □ Start time and authorization reference logged

Phase 1: Passive Infrastructure (Levels 4, 6)
  □ WHOIS (domain + primary IP)
  □ ASN + IP range discovery
  □ CT log subdomain enumeration (crt.sh)
  □ Passive subdomain enum (Subfinder + Amass)
  □ DNS full record extraction + analysis
  □ Shodan + Censys + FOFA search
  □ GitHub secret search
  □ Google dork sweep
  □ Document metadata (metagoofil)

Phase 2: People (Level 2)
  □ LinkedIn employee enumeration
  □ Email format identification (Hunter.io)
  □ Username search on key employees (Sherlock)
  □ Breach check on found emails (HIBP)
  □ Social media profiles

Phase 3: Business Context (Level 7)
  □ Company registry (MCA21 / Companies House / EDGAR)
  □ Director mapping
  □ Financial filings
  □ Job posting tech stack extraction
  □ Court records check
  □ Adverse media search

Phase 4: Active Recon (Level 8 — if authorized)
  □ DNS brute-force (puredns)
  □ Port scanning (Masscan → Nmap)
  □ Web probe (httpx)
  □ Directory discovery (ffuf)
  □ Nuclei scan
  □ API endpoint discovery

Phase 5: Correlation & Pivoting (Level 10)
  □ GA/AdSense ID pivot
  □ Favicon hash search
  □ SSL cert SAN pivot
  □ Co-hosted domain discovery
  □ Cross-reference all findings for connections
  □ Build entity relationship graph (Maltego / Obsidian)

Phase 6: Reporting
  □ All findings documented with: tool, command, output, screenshot
  □ All findings triaged by severity
  □ Each finding has: impact statement + remediation
  □ Timeline of investigation documented
  □ All evidence archived at archive.ph
  □ Report drafted in client format
  □ Review: no false positives, no exaggerated severity
  □ Submit via responsible disclosure or agreed channel
```

---

## 14. Level 10 Practice Tasks

### Task 1 — Pivot Chain Challenge
- Start with any public company's main domain
- Execute a full pivot chain using ONLY tools covered in this course:
  WHOIS email → reverse WHOIS → new domains → CT logs on each →
  favicon hash search → co-hosted domains → ASN → IP range → Shodan
- Map the entire chain in Obsidian's graph view
- Count: how many unique assets did you find that are not on the main website?

### Task 2 — Google Analytics Pivot
- Visit 5 websites in the same industry (e.g., 5 competing e-commerce sites)
- Extract the GA/AdSense ID from each using curl + grep
- Search PublicWWW or SpyOnWeb for each ID
- Identify: do any two competitors share a tracking ID? (= same operator?)

### Task 3 — Stylometry Exercise
- Find a public writer who has both named publications and anonymous blog posts
  (some journalists publish anonymously and under their real name)
- Collect 1000+ words from each source
- Manually identify 5 distinctive style markers that appear in both
- Use JGAAP to confirm or deny your hypothesis

### Task 4 — TraceLabs CTF
- Register at tracelabs.org
- Participate in the next available CTF event
- Submit at least 5 verified findings
- Review the judges' scoring feedback to understand what quality looks like

### Task 5 — Canary Token Deployment
- Create a free account at canarytokens.org
- Deploy 3 different token types:
  - A URL token (embed in a fake document or webpage)
  - An AWS key token (commit to a fake GitHub repo)
  - A Word document token (post to a public location)
- Monitor the dashboard for triggers
- Document: what information do you receive when a token fires?

### Task 6 — Full Investigation Report
- Combine ALL levels (1-10) into one complete investigation
- Target: a company with a public bug bounty program (scope allows passive recon)
- Deliverable: a professional OSINT report containing:
  - Executive summary (1 page)
  - Infrastructure map (domains, IPs, ASN, cloud assets)
  - People section (employees found, email formats, breach exposure)
  - Business context (registry data, tech stack from job postings)
  - Technical findings (Shodan, Nuclei, any active recon if authorized)
  - Entity relationship graph (Maltego or Obsidian)
  - Recommendations
  - All evidence archived and cited

---

## 15. Level 10 Summary

```
✅ Pivot mastery: 20+ pivot paths mapped from any starting entity
✅ GA/AdSense/nameserver pivots → link seemingly unrelated sites
✅ Favicon hash → all servers running the same application
✅ SSL serial/fingerprint → all domains on the same certificate
✅ JA3/JARM fingerprinting → identify C2 frameworks by TLS behavior
✅ Stylometry → attribute anonymous writing to known authors (JGAAP)
✅ Sock puppet persona building → credible, compartmentalized identity
✅ Breach data → password pattern analysis (legal analysis only)
✅ Entropy analysis (Python) → detect API keys in any text corpus
✅ Dark web OSINT: ransomware.live, Ahmia, Telegram monitoring tools
✅ Canary tokens → detect attackers pivoting through your infrastructure
✅ BGP hijacking detection via RIPE RIS and BGPMon
✅ Cloud misconfiguration patterns → AWS/GCP/Azure blind spots
✅ Threat actor OSINT: MITRE ATT&CK, Malpedia, IOC enrichment pipeline
✅ TraceLabs CTF → real-world missing persons OSINT with impact
✅ Obsidian knowledge graph → full entity relationship mapping
✅ Complete 6-phase professional investigation workflow

You have completed all 10 levels of the OSINT curriculum.
```

---

## 16. What Comes Next

```
You have now covered the full OSINT map. The next step is application:

Bug Bounty:
  → HackerOne, Bugcrowd, Intigriti, Synack
  → Start with private programs (lower competition, better triage)
  → Focus on recon-heavy programs (wildcard scope)

Professional Certifications:
  → OSCP (Offensive Security Certified Professional) — pentesting focus
  → TCM Security PNPT — practical penetration tester
  → GIAC GOSI — GIAC Open Source Intelligence
  → CREST CRT — UK-based professional certification

Communities:
  → OSINT Curious (osintcurio.us)
  → Bellingcat (bellingcat.com)
  → TraceLabs (tracelabs.org)
  → Reddit: r/OSINT, r/netsec, r/bugbounty
  → Twitter/X: #OSINT, #bugbounty communities

Continuing Education:
  → Michael Bazzell's OSINT Techniques book (updated annually): inteltechniques.com
  → Bellingcat's Online Investigation Toolkit
  → TCM Security courses (tcm-sec.com)
  → TryHackMe OSINT learning path
  → Hack The Box — OSINT challenges

Build in public:
  → Document your investigations (redacted for privacy)
  → Write up techniques you discover
  → Contribute to open source OSINT tools
  → Share wordlists, templates, and automation scripts
```

---

## 17. Master Tools Reference — Full Course

| Level | Tool | Purpose |
|---|---|---|
| 1 | Firefox Profile | Isolated research browser |
| 1 | VirtualBox + Kali | Isolated research environment |
| 1 | Obsidian / CherryTree | Documentation and knowledge graph |
| 2 | Sherlock / Maigret | Username search across platforms |
| 2 | GHunt | Gmail account OSINT |
| 2 | Phoneinfoga | Phone number OSINT framework |
| 2 | H8mail | Breach email search automation |
| 2 | Instaloader | Instagram bulk download |
| 2 | CrossLinked | LinkedIn email harvesting |
| 3 | ExifTool | Image/document metadata extraction |
| 3 | Yandex Images | Face-focused reverse image search |
| 3 | InVID/WeVerify | Video verification and keyframe extraction |
| 3 | SunCalc | Sun angle + shadow analysis |
| 3 | metagoofil | Domain-wide document metadata |
| 4 | Subfinder | Passive subdomain enumeration |
| 4 | Amass | Subdomain + org intelligence |
| 4 | dnsx | DNS resolution at scale |
| 4 | httpx | Web probing at scale |
| 4 | TruffleHog | Git history secret scanning |
| 4 | theHarvester | Email + subdomain harvesting |
| 5 | Google Earth Pro | Historical satellite imagery |
| 5 | Sentinel Hub | ESA free satellite imagery |
| 5 | WiGLE | Wi-Fi SSID geolocation |
| 5 | FlightAware + ADS-B Exchange | Flight tracking |
| 6 | bgp.he.net / BGPView | ASN and prefix intelligence |
| 6 | crt.sh | Certificate transparency lookup |
| 6 | SSL Labs / testssl.sh | TLS certificate analysis |
| 6 | s3scanner | Public S3 bucket checking |
| 6 | VirusTotal | Domain/IP/hash reputation |
| 6 | Shodan Monitor | Continuous asset monitoring |
| 7 | MCA21 / Zaubacorp | Indian company registry |
| 7 | OpenCorporates | Global company registry |
| 7 | SEC EDGAR | US public company filings |
| 7 | eCourts / NCLT | India court records |
| 7 | OpenSanctions | Aggregated sanctions lists |
| 7 | ICIJ Offshore Leaks | Panama/Pandora Papers database |
| 7 | myneta.info | India election affidavits |
| 8 | Nmap | Port scanning + service detection |
| 8 | Masscan | High-speed port discovery |
| 8 | ffuf | Web content/directory fuzzing |
| 8 | Nuclei | Template-based vulnerability scanning |
| 8 | EyeWitness | Web screenshot triage |
| 9 | notify | Alert to Discord/Slack/Telegram |
| 9 | recon-ng | Modular OSINT framework |
| 9 | SpiderFoot | Automated OSINT (200+ modules) |
| 9 | Maltego | Visual link analysis |
| 9 | uncover | Multi-engine search (Shodan+Censys+FOFA) |
| 10 | JGAAP | Stylometric authorship attribution |
| 10 | Canarytokens | Honey token deployment |
| 10 | ransomware.live | Ransomware victim tracking |
| 10 | MITRE ATT&CK | Threat actor TTP database |
| 10 | PublicWWW | GA/tracking ID pivot search |****
