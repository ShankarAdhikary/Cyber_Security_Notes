# OSINT — Level 6: Network & Infrastructure OSINT
> Network OSINT goes deeper than subdomain lists and open ports.
> It maps the full internet presence of a target — BGP routing,
> SSL certificate chains, cloud infrastructure, email architecture,
> threat intelligence feeds, and the technology relationships between
> all of it. This level bridges passive recon with the intelligence
> needed to understand an organization's true attack surface.

---

## 1. BGP & Autonomous System Intelligence

### 1.1 What BGP and ASNs Are

The internet is divided into **Autonomous Systems (AS)** — large networks
operated by a single organization (an ISP, a company, a university, a CDN).
Each AS is identified by a unique **ASN (Autonomous System Number)**.

**BGP (Border Gateway Protocol)** is the routing protocol that connects
all these ASes together. BGP routing tables are publicly observable —
you can see what IP prefixes an AS announces to the internet.

```
Why this matters for OSINT:
  Finding an org's ASN → finding ALL their IP ranges
  ASN also reveals:
    → Their upstream providers (who they buy transit from)
    → Their peering partners (who they exchange traffic with)
    → BGP communities (sometimes reveal internal routing policies)
    → Historical prefix announcements (what they used to own)
```

### 1.2 BGP Hurricane Electric (bgp.he.net)

```
URL: bgp.he.net

The most comprehensive free BGP intelligence tool.

Lookup by:
  Domain:   bgp.he.net/dns/example.com
  IP:       bgp.he.net/ip/93.184.216.34
  ASN:      bgp.he.net/AS15133
  Org name: bgp.he.net → search box

What each view shows:

  ASN view (bgp.he.net/AS15133):
  ├── ASN owner name and country
  ├── IPv4 and IPv6 prefixes announced (full IP range list)
  ├── Upstream providers (who they pay for transit)
  ├── Peers (who they peer with at IXPs)
  ├── Prefixes tab → download full prefix list as text
  ├── Graph tab → visual peering map
  └── Whois tab → ARIN/RIPE/APNIC registration data

  IP view (bgp.he.net/ip/93.184.216.34):
  ├── Which ASN this IP belongs to
  ├── Which prefix it falls in (e.g., 93.184.216.0/24)
  ├── Route origin
  └── BGP communities on the route
```

### 1.3 BGPView

```
URL: bgpview.io

Clean API and web interface for BGP data.

Web searches:
  bgpview.io/asn/15133         →  ASN details + prefixes
  bgpview.io/ip/93.184.216.34  →  IP to ASN lookup
  bgpview.io/prefix/93.184.216.0/24 → prefix details

API (free, no key):
# Get all IPv4 prefixes for an ASN
curl https://api.bgpview.io/asn/15133/prefixes | python3 -m json.tool

# Get ASN info from an IP
curl https://api.bgpview.io/ip/93.184.216.34

# Search by org name
curl "https://api.bgpview.io/search?query_term=Target+Corp"

# Get peers of an ASN
curl https://api.bgpview.io/asn/15133/peers

Extracting all IP ranges from an ASN:
curl -s https://api.bgpview.io/asn/15133/prefixes \
  | python3 -c "
import sys, json
data = json.load(sys.stdin)
for p in data['data']['ipv4_prefixes']:
    print(p['prefix'])
" > target_ip_ranges.txt
```

### 1.4 RIPE, ARIN, APNIC — Regional Registries

The internet's IP space is administered by five **Regional Internet Registries (RIRs)**:

```
ARIN:   arin.net        → North America (US, Canada)
RIPE:   ripe.net        → Europe, Middle East, Central Asia
APNIC:  apnic.net       → Asia-Pacific (includes India)
LACNIC: lacnic.net      → Latin America
AFRINIC:afrinic.net     → Africa

Each RIR maintains a WHOIS database for their region.

For Indian organizations → APNIC:
  whois -h whois.apnic.net 103.21.244.0

  Web: search.apnic.net → enter IP or org name

What each RIR record contains:
  ├── inetnum: IP range (e.g., 103.21.244.0 - 103.21.247.255)
  ├── netname: internal name of the block
  ├── descr:   description of the network
  ├── org:     organization ID
  ├── country: country code
  ├── admin-c: administrative contact reference
  ├── tech-c:  technical contact reference
  └── mnt-by:  who maintains this record (registrant's APNIC handle)
```

### 1.5 RADB — Routing Asset Database

```
RADB stores BGP route policies — which IP ranges an ASN announces.

# All prefixes announced by an ASN
whois -h whois.radb.net -- '-i origin AS15133' | grep "route:"

# All ASNs that originate a specific prefix
whois -h whois.radb.net 93.184.216.0/24

# Find all IPs an org has registered
whois -h whois.radb.net -- '-i mnt-by MAINT-AS15133'
```

### 1.6 Building a Full IP Target List

```bash
# Step 1: Find ASN from org name
curl -s "https://api.bgpview.io/search?query_term=Target+Corp" \
  | python3 -c "
import sys, json
data = json.load(sys.stdin)
for asn in data['data']['asns']:
    print(asn['asn'], asn['name'])"

# Step 2: Get all prefixes for the ASN
curl -s https://api.bgpview.io/asn/15133/prefixes \
  | python3 -c "
import sys, json
data = json.load(sys.stdin)
for p in data['data']['ipv4_prefixes']:
    print(p['prefix'])" > ip_ranges.txt

# Step 3: Expand CIDR ranges to individual IPs (careful with large ranges)
sudo apt install prips
while read cidr; do
  prips $cidr
done < ip_ranges.txt > all_ips.txt

# Step 4: Shodan search by ASN (more efficient than scanning IPs)
shodan search "asn:AS15133" --fields ip_str,port,org,product > shodan_results.txt

# Step 5: Or use Amass to enumerate by ASN
amass intel -asn 15133
```

---

## 2. SSL/TLS Certificate Intelligence

### 2.1 Why SSL Certificates Are Intelligence-Rich

SSL/TLS certificates contain:
```
├── Subject Common Name (CN)     →  the domain the cert is for
├── Subject Alternative Names    →  ALL other domains on the same cert
│   (SANs)                          (often reveals related infrastructure)
├── Organization (O)             →  legal name of the cert holder
├── Organizational Unit (OU)     →  department (sometimes internal names)
├── Issuer CN                    →  Certificate Authority
├── Valid From / To              →  issuance and expiry dates
├── Serial Number                →  unique identifier for the cert
├── Fingerprint (SHA-256)        →  unique hash of the cert
└── Key size and algorithm       →  security level (2048-bit RSA, P-256 ECDSA)
```

The **Subject Alternative Names (SANs)** field is the most valuable —
a single certificate often covers dozens of related domains.

### 2.2 Certificate Lookup Tools

**crt.sh**
```bash
# Find all certs for a domain (including SANs)
curl -s "https://crt.sh/?q=example.com&output=json" \
  | python3 -c "
import sys, json
data = json.load(sys.stdin)
for cert in data:
    print(cert['id'], cert['not_before'], cert['common_name'])"

# Extract all SANs from all certs
curl -s "https://crt.sh/?q=%.example.com&output=json" \
  | python3 -c "
import sys, json
data = json.load(sys.stdin)
names = set()
for cert in data:
    for name in cert['name_value'].split('\n'):
        name = name.replace('*.','').strip()
        if name: names.add(name)
for n in sorted(names): print(n)"

# Find certs by organization name
curl "https://crt.sh/?O=Target+Corp&output=json"
# → Returns ALL certs issued to "Target Corp" across ALL their domains
# → Reveals domains not containing "example.com"
```

**Censys Certificate Search**
```
URL: search.censys.io/certificates

Key queries:
  parsed.names: "example.com"
  parsed.subject.common_name: "example.com"
  parsed.subject.organization: "Target Corp"
  parsed.subject.organizational_unit: "IT Department"
  parsed.issuer.common_name: "Let's Encrypt"
  parsed.validity.end: [now TO *]        ← currently valid
  parsed.validity.end: [* TO now]        ← expired
  parsed.names: "*.internal.example.com" ← internal wildcard certs

API:
from censys.search import CensysCertificates
c = CensysCertificates()
for cert in c.search('parsed.subject.organization: "Target Corp"'):
    print(cert)
```

### 2.3 SSL Labs — Certificate Analysis

```
URL: ssllabs.com/ssltest/

What it checks:
  ├── Certificate chain validity
  ├── Protocol support (TLS 1.0, 1.1, 1.2, 1.3)
  ├── Cipher suite strength
  ├── Known vulnerabilities (POODLE, BEAST, ROBOT, HEARTBLEED)
  ├── HSTS configuration
  ├── Certificate transparency logging
  └── Overall grade (A+ to F)

Intelligence value:
  → TLS 1.0/1.1 still supported → outdated configuration
  → Self-signed certificate → may indicate internal/dev server
  → Certificate expired → poor maintenance hygiene
  → Grade F → serious misconfiguration → worth investigating further

CLI alternative: testssl.sh
  bash testssl.sh example.com
  # Performs the same checks locally without submitting to SSL Labs
  # install: git clone https://github.com/drwetter/testssl.sh.git
```

### 2.4 Finding Related Infrastructure via Shared Certificates

```
Technique: Multiple domains sharing the same certificate = related infrastructure

Step 1: Get a certificate's serial number or fingerprint
  curl -s "https://crt.sh/?q=example.com&output=json" \
    | python3 -c "import sys,json; [print(c['serial_number']) for c in json.load(sys.stdin)]"

Step 2: Search for other domains using the SAME certificate serial
  curl "https://crt.sh/?serial=<SERIAL>&output=json"

Step 3: The result may show dozens of domains on the same cert
  → These are all hosted by the same entity
  → Include them all in your recon scope

Example:
  example.com has a SAN cert covering:
    example.com
    www.example.com
    api.example.com        ← API endpoint
    cdn.example.com        ← CDN subdomain
    mail.example.com       ← mail server
    internal.example.com   ← internal tool exposed!
    dev.example.com        ← dev environment!
  → All from one certificate lookup
```

### 2.5 Shodan SSL Filters

```bash
# All hosts using certs for a domain
shodan search 'ssl.cert.subject.cn:"example.com"'

# All hosts with expired certs (org-scoped)
shodan search 'org:"Target Corp" ssl.cert.expired:true'

# Find hosts using TLS 1.0 (outdated)
shodan search 'org:"Target Corp" ssl.version:"TLSv1"'

# Find self-signed certs (issuer = subject)
shodan search 'ssl.cert.issuer.cn:"example.com" ssl.cert.subject.cn:"example.com"'

# Find certs with specific org name
shodan search 'ssl:"Target Corp"'

# Find all IPs using the same cert fingerprint
shodan search 'ssl.cert.fingerprint:"AA:BB:CC:..."'
```

---

## 3. Email Infrastructure OSINT

### 3.1 Why Email Infrastructure Matters

Email infrastructure reveals:
```
→  Which email provider they use (Google Workspace, Microsoft 365, self-hosted)
→  Which third-party services are authorized to send email for them
→  Whether they have DMARC enforcement (security maturity indicator)
→  Mail servers that may be directly accessible
→  Reporting addresses that reveal internal email patterns
```

### 3.2 MX Record Analysis

```bash
# Get mail servers
dig MX example.com

# Common MX patterns and what they reveal:
aspmx.l.google.com        →  Google Workspace (Gmail for Business)
mail.protection.outlook.com → Microsoft 365 / Exchange Online
mx.zoho.com               →  Zoho Mail
mxa.mailgun.org           →  Mailgun (transactional email, not primary)
in1.smtp.messagingengine.com → Fastmail
mail.example.com          →  Self-hosted mail server (investigate further)
```

### 3.3 SPF Record Deep Analysis

**SPF (Sender Policy Framework)** lists all servers authorized to send email
for a domain. It directly reveals every email service the org uses.

```bash
dig TXT example.com | grep "v=spf1"

# Example SPF record:
v=spf1 include:_spf.google.com include:sendgrid.net include:amazonses.com
       ip4:93.184.216.34 ip4:198.51.100.0/24 ~all

# Breakdown:
include:_spf.google.com   →  Uses Google Workspace
include:sendgrid.net      →  Uses SendGrid for marketing email
include:amazonses.com     →  Uses Amazon SES for transactional email
ip4:93.184.216.34         →  This specific IP can send email (on-premise server?)
ip4:198.51.100.0/24       →  This IP range can send (bulk mailer / ESPs)
~all                      →  Soft fail (weak DMARC policy)

# Recursive SPF resolution (includes have their own includes):
# Use: dmarcly.com/tools/spf-record-raw-checker
# or:  mxtoolbox.com/spf.aspx
# → Expands all includes → shows the full flat list of authorized IPs

SPF security indicators:
  -all (hard fail)   →  Strict policy — good security hygiene
  ~all (soft fail)   →  Lenient — email can still be spoofed
  +all               →  Anyone can send! Critical misconfiguration
  (no SPF at all)    →  Domain can be spoofed freely
```

### 3.4 DMARC Record Analysis

**DMARC (Domain-based Message Authentication, Reporting & Conformance)**
defines what happens when email fails SPF/DKIM checks.

```bash
dig TXT _dmarc.example.com

# Example DMARC record:
v=DMARC1; p=reject; rua=mailto:dmarc-reports@example.com;
ruf=mailto:forensics@example.com; pct=100; sp=reject

# Breakdown:
p=reject          →  Fail = reject (strong policy — emails are blocked)
p=quarantine      →  Fail = spam folder (medium policy)
p=none            →  Fail = nothing happens (monitoring only — weak)
rua=              →  Aggregate report destination (internal email address!)
ruf=              →  Forensic report destination (may be a security team email)
pct=100           →  Applies to 100% of mail
sp=reject         →  Subdomains also have reject policy

Intelligence value:
  rua= and ruf= reveal INTERNAL EMAIL ADDRESSES:
    rua=mailto:dmarc@example.com
    → This is a real internal email address
    → Can be used for targeted phishing or employee enumeration
    → May reveal internal email naming convention

DMARC security indicators:
  p=reject          →  Best security — hard to spoof this domain
  p=quarantine      →  Medium security
  p=none            →  Domain can be spoofed (emails go to inbox!)
  No DMARC at all   →  Completely spoofable domain
```

### 3.5 DKIM Record

```bash
# DKIM records are at a selector prefix
# The selector is usually "default", "google", "k1", "s1", "s2", "mail"
dig TXT default._domainkey.example.com
dig TXT google._domainkey.example.com
dig TXT s1._domainkey.example.com
dig TXT selector1._domainkey.example.com   ← Microsoft 365

# DKIM record reveals:
p=<public key>    →  RSA public key (key length indicates security level)
k=rsa             →  Key type
1024-bit key      →  Weak — vulnerable to factoring
2048-bit key      →  Standard
4096-bit key      →  Strong

# Finding DKIM selectors:
# 1. Email headers (if you have an email from the domain)
#    Look for: DKIM-Signature: v=1; a=rsa-sha256; d=example.com; s=SELECTOR
# 2. Try common selectors: google, default, mail, smtp, k1, selector1, selector2
# 3. Tools: mxtoolbox.com/dkim.aspx
```

### 3.6 MXToolbox — Complete Email Analysis

```
URL: mxtoolbox.com

One-stop analysis for all email-related records:
  mxtoolbox.com/MXLookup.aspx     →  MX records
  mxtoolbox.com/spf.aspx          →  SPF record + recursive expansion
  mxtoolbox.com/dmarc.aspx        →  DMARC analysis
  mxtoolbox.com/dkim.aspx         →  DKIM verification
  mxtoolbox.com/blacklists.aspx   →  Is the domain/IP on spam blacklists?
  mxtoolbox.com/EmailHeaders.aspx →  Parse full email headers

Blacklist check:
  If a target's mail server IP is on spam blacklists → poor email security hygiene
  → They may be sending spam, have been compromised, or have poor configuration
```

---

## 4. Cloud Infrastructure Intelligence

### 4.1 Identifying the Cloud Provider

```bash
# Method 1: IP range lookup
# Check if the IP falls in a known cloud provider's published range

# AWS IP ranges (updated regularly)
curl -s https://ip-ranges.amazonaws.com/ip-ranges.json \
  | python3 -c "
import sys, json
data = json.load(sys.stdin)
target_ip = '54.239.28.85'
for p in data['prefixes']:
    import ipaddress
    try:
        if ipaddress.ip_address(target_ip) in ipaddress.ip_network(p['ip_prefix']):
            print(f'AWS Region: {p[\"region\"]} | Service: {p[\"service\"]}')
    except: pass"

# Azure IP ranges
# Download from: microsoft.com/en-us/download/details.aspx?id=56519

# GCP IP ranges
curl -s https://www.gstatic.com/ipranges/cloud.json \
  | python3 -c "
import sys, json
data = json.load(sys.stdin)
for p in data['prefixes']:
    if 'ipv4Prefix' in p:
        print(p['ipv4Prefix'], p.get('scope',''), p.get('service',''))"

# Method 2: Reverse DNS pattern
dig -x 54.239.28.85
# ec2-54-239-28-85.compute-1.amazonaws.com → AWS EC2
# 85.28.239.54.bc.googleusercontent.com   → Google Cloud
# 54-239-28-85.azure.com                  → Azure

# Method 3: ipinfo.io
curl https://ipinfo.io/54.239.28.85/org
# Returns: "AS16509 Amazon.com, Inc." or "AS15169 Google LLC" etc.

# Method 4: Shodan
shodan host 54.239.28.85
# org field shows cloud provider
```

### 4.2 AWS Asset Discovery

**S3 Bucket Enumeration:**
```bash
# Naming patterns to try
# Buckets live at: https://<bucket>.s3.amazonaws.com
# OR:              https://s3.amazonaws.com/<bucket>

Patterns:
  company-name
  company-backup
  company-dev
  company-staging
  company-prod
  company-assets
  company-media
  company-uploads
  company-data
  company-logs
  company-images
  company-static
  company-public
  company-private
  company.com
  www.company.com

# S3Scanner — automated public bucket check
pip3 install s3scanner

# Scan a single bucket
s3scanner scan --bucket company-backup

# Scan from a wordlist
s3scanner scan --bucket-file bucket_names.txt

# Generate a wordlist from a company name
# company = "targetcorp"
python3 -c "
name='targetcorp'
words=['backup','dev','staging','prod','assets','media','logs','data',
       'uploads','public','private','static','images','files','archive']
for w in words: print(f'{name}-{w}')
for w in words: print(f'{name}.{w}')
for w in words: print(f'{w}-{name}')
print(name)
" > bucket_list.txt

s3scanner scan --bucket-file bucket_list.txt

# If a bucket is public, check its contents:
aws s3 ls s3://bucket-name --no-sign-request
aws s3 cp s3://bucket-name/file.txt . --no-sign-request
```

**GrayhatWarfare — Search Engine for Public Buckets:**
```
URL: buckets.grayhatwarfare.com

Searches across indexed public S3 buckets and Azure blob containers.
Search by: company name, domain, keyword, file type

No account needed for basic searches.
Shows: bucket name, file names, sizes, last modified dates

Usage:
  Search: "targetcorp"
  → Results show all public S3 files mentioning targetcorp
  → Look for: database dumps, config files, source code, user data
```

**AWS Metadata Service (Instance Reachability):**
```
The AWS instance metadata endpoint (169.254.169.254) is only reachable from
within AWS. However, if a web application is vulnerable to SSRF
(Server-Side Request Forgery), an attacker can use it to query:
  http://169.254.169.254/latest/meta-data/
  → Returns: instance ID, region, IAM role credentials, security groups

This is ACTIVE exploitation — only in authorized testing.
From an OSINT perspective:
  → Knowing a target runs on AWS + has SSRF → high-value finding
  → Look for SSRF indicators in the web app (URL parameters that fetch URLs)
```

### 4.3 Firebase Open Database Discovery

```bash
# Firebase databases follow this URL pattern:
# https://<app-name>.firebaseio.com/

# If a database is publicly readable:
curl https://targetapp.firebaseio.com/.json
# Returns: all data in the database (critical finding)
# If returns: null → database exists but is secured
# If returns: "Permission Denied" → secured
# If returns: JSON data → OPEN DATABASE → critical finding

# Common app name patterns to try:
company
company-prod
company-dev
company-app
company-mobile
company-default
company-firebase

# Check all common paths:
curl https://targetapp.firebaseio.com/users.json
curl https://targetapp.firebaseio.com/orders.json
curl https://targetapp.firebaseio.com/admin.json
```

### 4.4 Google Cloud Storage

```bash
# GCS bucket URL patterns:
# https://storage.googleapis.com/<bucket-name>
# https://<bucket-name>.storage.googleapis.com

# Check if a bucket is public:
curl -I https://storage.googleapis.com/company-backup
# 200 = public
# 403 = exists but private
# 404 = does not exist

# List contents of a public bucket:
curl https://storage.googleapis.com/company-backup
# Returns XML listing of all objects
```

### 4.5 Azure Blob Storage

```bash
# Azure blob storage URL pattern:
# https://<account>.blob.core.windows.net/<container>/<blob>

# Check for public containers:
curl "https://targetcorp.blob.core.windows.net/public?restype=container&comp=list"
# If returns XML listing → container is public

# Common account/container names:
# account = company name (targetcorp.blob.core.windows.net)
# container = backup, data, files, public, media, assets, logs
```

---

## 5. Threat Intelligence

### 5.1 What Threat Intelligence Reveals

```
Threat intelligence in OSINT context tells you:
  → Is a target IP associated with malware command-and-control?
  → Has a domain been used in phishing campaigns?
  → Is an IP on spam / abuse blacklists?
  → Has the target been involved in known attacks?
  → Is infrastructure associated with a known threat actor?
  → What vulnerabilities affect the target's exposed services?

For offensive OSINT: reveals security posture of target
For defensive OSINT: reveals threats targeting your organization
```

### 5.2 VirusTotal

```
URL: virustotal.com

VirusTotal aggregates results from 70+ security vendors.

What you can analyze:
  IP address:   vt.com/gui/ip-address/93.184.216.34
  Domain:       vt.com/gui/domain/example.com
  URL:          vt.com/gui/url/<url>
  File hash:    vt.com/gui/file/<sha256>

IP/Domain report includes:
  ├── Passive DNS (historical A records)
  ├── Subdomains
  ├── Communicating files (malware that contacted this IP)
  ├── Referrer files (documents that link to this domain)
  ├── Security vendor verdicts (clean / malicious / suspicious)
  ├── WHOIS data
  └── SSL certificates

API usage:
curl --request GET \
  --url "https://www.virustotal.com/api/v3/ip_addresses/93.184.216.34" \
  --header "x-apikey: YOUR_API_KEY"

# Python library:
pip3 install vt-py
import vt
client = vt.Client("YOUR_API_KEY")
ip = client.get_object("/ip_addresses/93.184.216.34")
print(ip.last_analysis_stats)
```

### 5.3 AlienVault OTX (Open Threat Exchange)

```
URL: otx.alienvault.com

What it is:
  A free, community-driven threat intelligence platform.
  Anyone can submit and consume threat indicators (IOCs).

What you can look up:
  ├── IP address reputation and malicious activity
  ├── Domain reputation and associated malware
  ├── File hashes (malware identification)
  ├── CVEs and affected products
  └── Threat actor groups and their TTPs

Usage:
  1. Create free account at otx.alienvault.com
  2. Search any IP, domain, hash, or CVE
  3. View: associated malware, geolocation, passive DNS, threat actor tags

API:
pip3 install OTXv2
from OTXv2 import OTXv2, IndicatorTypes
otx = OTXv2("YOUR_API_KEY")
indicators = otx.get_indicator_details_full(IndicatorTypes.DOMAIN, "example.com")
```

### 5.4 Abuse.ch — Specialized Threat Feeds

```
Abuse.ch operates several specialized threat intelligence services:

URLhaus:    urlhaus.abuse.ch
  → Tracks malicious URLs spreading malware
  → Search: domain, IP, URL
  → If target domain appears here → it is (or was) distributing malware

ThreatFox:  threatfox.abuse.ch
  → Tracks malware IOCs (IPs, domains, hashes)
  → Useful for identifying C2 infrastructure

Feodo Tracker: feodotracker.abuse.ch
  → Tracks botnet command-and-control servers
  → Primarily banking trojans (Emotet, Dridex, TrickBot, QBot)
  → If a target IP appears here → it is a known C2 server

MalwareBazaar: bazaar.abuse.ch
  → Repository of malware samples
  → Search by hash, file name, YARA rule, tag

API (URLhaus example):
curl -X POST https://urlhaus-api.abuse.ch/v1/host/ \
  -d "host=example.com"
```

### 5.5 Shodan CVE Search

```bash
# Find hosts affected by a specific CVE
shodan search 'vuln:CVE-2021-44228'   ← Log4Shell (paid filter)

# Find vulnerable Apache versions known to affect a CVE
shodan search 'product:"Apache httpd" version:"2.4.49"'
# Apache 2.4.49 had path traversal CVE-2021-41773

# Find known vulnerable software (by product + version)
shodan search 'product:"Microsoft IIS httpd" version:"6.0"'
# IIS 6.0 = Windows 2003 server = very outdated

# Combine with org to scope to target
shodan search 'org:"Target Corp" product:"Apache httpd" version:"2.4.49"'

# Note: vuln: filter requires Shodan paid plan
# Workaround: search by product version and cross-reference CVE databases manually
```

### 5.6 Vulners & NVD — CVE Research

```
Vulners:   vulners.com
  → Search CVEs by product name, version, vendor
  → Returns full vulnerability details + CVSS score + PoC links

NVD:       nvd.nist.gov
  → US National Vulnerability Database — authoritative CVE source

Usage workflow:
  1. From Shodan: identify product "Apache httpd" version "2.4.49"
  2. Search vulners.com: "Apache httpd 2.4.49"
  3. Returns: CVE-2021-41773 (path traversal, CVSS 9.8)
  4. Cross-reference NVD for official details
  5. Document: target runs vulnerable software version → flag as high priority

Shodan + Vulners workflow:
shodan search 'org:"Target Corp"' --fields ip_str,port,product,version \
  | tee products.txt
# Then manually search each unique product:version in Vulners
```

### 5.7 Pulsedive — Threat Intelligence Aggregator

```
URL: pulsedive.com

Aggregates data from many threat intelligence sources into one.

Search by: IP, domain, URL, hash

Returns:
  ├── Risk score (low / medium / high / critical)
  ├── Feeds that flagged this indicator
  ├── Associated malware families
  ├── Passive DNS
  ├── WHOIS
  └── Historical behavior

Free tier: 30 queries/day
```

### 5.8 Hunting for Malicious Infrastructure

```
Shodan queries for common malicious infrastructure:

# Cobalt Strike C2 servers
shodan search '"Cobalt Strike"'
shodan search 'product:"Cobalt Strike Beacon"'

# Metasploit listeners
shodan search '"msf"'
shodan search 'product:"Metasploit"'

# Common C2 frameworks
shodan search '"Empire"'
shodan search '"Covenant"'
shodan search '"Havoc C2"'

# Default certificate on C2 (self-signed)
shodan search 'ssl.cert.subject.cn:"Major Cobalt Strike"'
shodan search 'ssl.cert.issuer.cn:"Major Cobalt Strike"'

# Phishing kits
shodan search 'http.title:"Microsoft - Sign In"' country:IN
# (legitimate Microsoft login pages would be on microsoft.com)

# Open C2 panel
shodan search 'http.title:"Cobalt Strike"'
shodan search 'http.title:"Empire Stager"'
```

---

## 6. Network Service Fingerprinting (Passive)

### 6.1 Banner Intelligence from Shodan

Shodan stores raw banners — the first response from a service when connected.
Analyzing banners reveals exact versions, configurations, and operational details.

```bash
# Get the full banner for a specific host and port
shodan host 93.184.216.34

# Example SSH banner:
# SSH-2.0-OpenSSH_8.4p1 Debian-5+deb11u1
# → Version: OpenSSH 8.4
# → OS: Debian 11 (Bullseye)
# → Check: any CVEs for OpenSSH 8.4?

# Example HTTP banner:
# HTTP/1.1 200 OK
# Server: Apache/2.4.51 (Debian)
# X-Powered-By: PHP/7.4.30
# → Apache 2.4.51 on Debian
# → PHP 7.4.30 (EOL — no longer receiving security updates!)
# → Check vulners.com for known PHP 7.4.x CVEs

# Example FTP banner:
# 220 vsFTPd 3.0.3
# → vsftpd 3.0.3 → check CVE database

# Example SMTP banner:
# 220 mail.example.com ESMTP Postfix (Ubuntu)
# → Reveals mail server software, hostname, and OS
```

### 6.2 Default Credentials Intelligence

```
Many exposed services use default credentials. Shodan results showing:
  "Powered by DrayTek"    →  DrayTek router → default: admin/admin
  "NETGEAR"               →  NETGEAR router → default: admin/password
  "Hikvision"             →  CCTV cameras   → default: admin/12345
  "phpMyAdmin"            →  DB admin panel → try default creds
  "Tomcat"                →  Apache Tomcat  → default: tomcat/tomcat, admin/admin

Resources:
  Default creds database: cirt.net/passwords
  Router default creds:   routerpasswords.com
  Default creds list:     github.com/ihebski/DefaultCreds-cheat-sheet

IMPORTANT: Testing default credentials without authorization is illegal.
           Use this knowledge to:
             → Document that a service is exposed
             → Report that default credentials are a risk
             → Never actually authenticate
```

### 6.3 Protocol-Specific Intelligence

```bash
# FTP — check for anonymous login possibility
# (Do not actually login without authorization)
shodan search 'port:21 "Anonymous FTP" org:"Target Corp"'
# If found → anonymous FTP access is possible → critical finding

# SMB — exposed file shares
shodan search 'port:445 org:"Target Corp"'
# SMB exposed to internet → potential EternalBlue, credential access

# RDP — exposed remote desktop
shodan search 'port:3389 org:"Target Corp"'
# RDP exposed → brute-force / credential stuffing target

# Redis — commonly unauthenticated
shodan search 'port:6379 org:"Target Corp"'
# Redis with no auth → direct data access

# MongoDB — commonly unauthenticated
shodan search 'port:27017 product:"MongoDB" org:"Target Corp"'
# Open MongoDB → read/write database access

# Elasticsearch — often unauthenticated
shodan search 'port:9200 product:"Elastic" org:"Target Corp"'

# Exposed Kubernetes dashboard
shodan search 'http.title:"Kubernetes Dashboard" org:"Target Corp"'

# Exposed Jupyter notebooks
shodan search 'http.title:"Jupyter" port:8888 org:"Target Corp"'

# Exposed Grafana (check for default admin/admin)
shodan search 'http.title:"Grafana" org:"Target Corp"'
```

---

## 7. Internet Exchange Points (IXPs)

### 7.1 What IXPs Reveal

**Internet Exchange Points (IXPs)** are physical locations where ISPs and
organizations interconnect to exchange traffic. Membership in an IXP is public.

```
Why it matters:
  → Confirms the organization operates significant network infrastructure
  → Reveals peering partners (business relationships)
  → Reveals geographical presence (which cities they have infrastructure in)
  → ASN at multiple IXPs = globally distributed infrastructure

Key IXPs (global):
  DE-CIX (Frankfurt)   →  Largest IXP globally
  AMS-IX (Amsterdam)   →  Second largest
  LINX (London)
  DECIX Mumbai / Chennai  →  India
  BBIX (Tokyo, Singapore)

IXP member lookup:
  PeeringDB:  peeringdb.com
  → Search by ASN or org name
  → Shows: all IXPs they connect at, contact info, IX ports, IP addresses at each IX
  → Also shows: data center presence (which data centers they co-locate in)
```

### 7.2 PeeringDB

```
URL: peeringdb.com

What to look up:
  1. Search by ASN → see all exchange points
  2. See which data centers they operate in
  3. Find NOC (Network Operations Center) email addresses
  4. See their peering policy (open / selective / no peering)

API:
curl "https://www.peeringdb.com/api/net?asn=15133"
curl "https://www.peeringdb.com/api/netixlan?asn=15133"  # IX connections
curl "https://www.peeringdb.com/api/netfac?asn=15133"   # Facility/data center connections

Intelligence value:
  IXP location → city where they have infrastructure
  Data center → physical building → security implications
  Peering partners → business relationships → associated companies
  NOC email → internal contact point
```

---

## 8. Network Topology Mapping

### 8.1 Traceroute Intelligence

```bash
# Standard traceroute — shows hop-by-hop path to target
traceroute example.com
tracert example.com    (Windows)

# What each hop reveals:
# Hop 1: Your gateway (local router)
# Hop 2-3: Your ISP's network
# Hop 4-X: Transit networks (ISPs in between)
# Last hop: Target's network edge

# Reading router hostnames in hops:
# ae-4.r04.nycmny01.us.bb.gin.ntt.net
#   ↑ Interface  ↑ Router ID  ↑ City  ↑ Country  ↑ Provider (NTT)
# Decoding hostnames reveals carrier and city for each hop

# MTR (My TraceRoute) — better than traceroute
sudo apt install mtr
mtr --report example.com
mtr --report-cycles 10 --report example.com

# TCP traceroute (bypasses ICMP filtering)
sudo traceroute -T -p 80 example.com
sudo traceroute -T -p 443 example.com

# Paris traceroute (more consistent path)
sudo apt install paris-traceroute
paris-traceroute example.com
```

### 8.2 What Traceroute Reveals

```
1. Transit providers used
   → Hops through "ntt.net" → uses NTT as transit
   → Hops through "cogentco.com" → uses Cogent as transit

2. CDN presence
   → Last hops resolve to "akamai.net" → behind Akamai CDN
   → Last hops resolve to "cloudflare.net" → behind Cloudflare
   → If behind CDN → the CDN may mask the real origin IP

3. Geographic routing
   → Hops through "fra" = Frankfurt → traffic routes through Germany
   → Latency jumps → identify location changes in the path

4. Organization's network edge
   → The last named hop before the target is their border router
   → Its hostname may contain their company name or city code
```

### 8.3 Finding Real IP Behind CDN/Cloudflare

```
Many organizations hide their real server IPs behind Cloudflare.
These techniques find the real origin IP:

Method 1: Historical DNS (before they added CDN)
  SecurityTrails → Historical DNS → A record from before the CDN
  → Often the real IP is still serving the site

Method 2: Non-CDN subdomains
  The CDN only protects certain subdomains (usually www, mail)
  → Try: direct.example.com, cpanel.example.com, ftp.example.com
  → These often bypass CDN and show the real IP

Method 3: MX record IP
  dig MX example.com → resolves to a mail server IP
  → Mail servers are rarely behind CDN
  → If on same subnet as web server → real server IP range revealed

Method 4: Shodan historical data
  shodan search 'hostname:"example.com"'
  → Shodan may have indexed the server before CDN was added

Method 5: SSL certificate subject vs. actual hostname
  openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -noout -text
  → If cert is not from Cloudflare → real cert exposes info

Method 6: Favicon hash search
  Get favicon from example.com
  Calculate hash → search Shodan
  → If same favicon on a non-CDN IP → real server found

Method 7: Email headers from the domain
  Send an email to a real address at the domain
  → Check headers of the bounce/reply
  → Sending server IP = internal mail server → subnet clue
```

---

## 9. Network Monitoring & Continuous Intelligence

### 9.1 Shodan Monitor

```bash
# Create an alert for an IP range
shodan alert create "Target Corp" 104.16.0.0/16

# List your alerts
shodan alert list

# Available triggers (events that fire the alert):
shodan alert triggers

# Enable specific triggers on an alert
shodan alert enable <alertID> new_service,open_database,vulnerable,ssl_expired,malware,iot

# Trigger reference:
new_service          →  New open port/service detected
open_database        →  Unauthenticated DB (MongoDB, Redis, Elasticsearch)
vulnerable           →  Host matched a known CVE
ssl_expired          →  SSL certificate has expired
internet_scanner     →  Host is actively scanning the internet
uncommon             →  Unusual/rarely-seen service
malware              →  Host associated with malware activity
iot                  →  IoT device detected
industrial_control_system → ICS/SCADA protocol detected

# Domain monitoring (monitors DNS-resolved IPs automatically)
shodan alert domain example.com

# View findings from your alerts
# → Go to monitor.shodan.io in browser
```

### 9.2 Certificate Monitoring

```
Certspotter:  sslmate.com/certspotter

Free plan: monitor up to 5 domains
Sends email alert when a new certificate is issued for your monitored domains.

API (free):
curl "https://api.certspotter.com/v1/issuances?domain=example.com&include_subdomains=true&expand=dns_names" \
  -H "Authorization: Bearer YOUR_API_TOKEN"

Returns: all recent certificate issuances
New issuance = new subdomain or service being deployed → new recon target

crt.sh monitoring:
  No built-in alerts, but you can automate:
  # Check for new certs daily via cron
  curl -s "https://crt.sh/?q=%.example.com&output=json" \
    | python3 -c "import json,sys; [print(c['not_before'],c['common_name']) for c in json.load(sys.stdin)]" \
    | sort -r | head -20
```

### 9.3 Passive DNS Monitoring

```
SecurityTrails Monitors:
  → Set up alerts for DNS record changes on a domain
  → Notified when: A record changes, new subdomain appears, MX changes

VirusTotal Intelligence:
  → Subscribe to domain for change notifications (paid plan)
  → Notifies when: new subdomains indexed, SSL cert changes, new files referencing domain

Manual periodic check workflow:
  # Save current state
  subfinder -d example.com -silent > state_$(date +%Y%m%d).txt

  # Compare with previous state (run weekly)
  diff state_20260907.txt state_20261014.txt
  # New lines = newly discovered subdomains
  # Removed lines = decommissioned assets
```

---

## 10. Full Network OSINT Workflow

```bash
#!/bin/bash
TARGET_DOMAIN="example.com"
TARGET_ORG="Target Corp"
OUTPUT="net_osint_${TARGET_DOMAIN}"
mkdir -p $OUTPUT

echo "=== NETWORK OSINT: $TARGET_DOMAIN ==="

# Phase 1: ASN & IP Range Discovery
echo "[1/7] Finding ASN and IP ranges..."
# Manual: go to bgp.he.net, search for $TARGET_ORG, note ASN
# API:
curl -s "https://api.bgpview.io/search?query_term=${TARGET_ORG// /+}" \
  | python3 -c "
import sys, json
data = json.load(sys.stdin)
if 'asns' in data.get('data', {}):
    for asn in data['data']['asns']:
        print(f'ASN: {asn[\"asn\"]} | {asn[\"name\"]}')" \
  | tee $OUTPUT/asn_results.txt

# Phase 2: Email Infrastructure
echo "[2/7] Analyzing email infrastructure..."
dig MX $TARGET_DOMAIN +short > $OUTPUT/mx_records.txt
dig TXT $TARGET_DOMAIN +short | grep "v=spf" > $OUTPUT/spf_record.txt
dig TXT _dmarc.$TARGET_DOMAIN +short > $OUTPUT/dmarc_record.txt
echo "MX records: $(cat $OUTPUT/mx_records.txt)"
echo "SPF: $(cat $OUTPUT/spf_record.txt)"
echo "DMARC: $(cat $OUTPUT/dmarc_record.txt)"

# Phase 3: SSL Certificate Analysis
echo "[3/7] Querying certificate transparency logs..."
curl -s "https://crt.sh/?q=%.${TARGET_DOMAIN}&output=json" \
  | python3 -c "
import sys, json
data = json.load(sys.stdin)
names = set()
for c in data:
    for n in c['name_value'].split('\n'):
        n = n.replace('*.','').strip()
        if n: names.add(n)
for n in sorted(names): print(n)" > $OUTPUT/ct_domains.txt
echo "Domains found via CT: $(wc -l < $OUTPUT/ct_domains.txt)"

# Phase 4: Shodan Intelligence
echo "[4/7] Querying Shodan..."
shodan search "org:\"${TARGET_ORG}\"" \
  --fields ip_str,port,product,version,country > $OUTPUT/shodan_results.txt
echo "Shodan results: $(wc -l < $OUTPUT/shodan_results.txt)"

# Phase 5: Threat Intelligence
echo "[5/7] Checking VirusTotal..."
# Manual: visit virustotal.com/gui/domain/$TARGET_DOMAIN

# Phase 6: Cloud Asset Check
echo "[6/7] Checking for public cloud assets..."
# S3 check (selected patterns)
for bucket in ${TARGET_DOMAIN//./-} ${TARGET_DOMAIN%%.*} ${TARGET_DOMAIN%%.*}-backup; do
  STATUS=$(curl -s -o /dev/null -w "%{http_code}" \
    "https://${bucket}.s3.amazonaws.com")
  if [[ $STATUS == "200" || $STATUS == "403" ]]; then
    echo "S3 bucket exists (status $STATUS): $bucket" | tee -a $OUTPUT/cloud_assets.txt
  fi
done

# Firebase check
FB_STATUS=$(curl -s -o /dev/null -w "%{http_code}" \
  "https://${TARGET_DOMAIN%%.*}.firebaseio.com/.json")
if [[ $FB_STATUS == "200" ]]; then
  echo "OPEN Firebase database found!" | tee -a $OUTPUT/cloud_assets.txt
fi

echo ""
echo "=== COMPLETE ==="
echo "Output saved to: $OUTPUT/"
ls -lh $OUTPUT/
```

---

## 11. Level 6 Practice Tasks

### Task 1 — ASN & BGP Analysis
- Pick any large Indian ISP or company (Reliance Jio, Airtel, Infosys)
- Find their ASN using bgp.he.net
- List all IPv4 prefixes they announce
- Identify their top 3 upstream providers
- Use PeeringDB to find which IXPs they connect at

### Task 2 — Email Security Audit
- Choose any company's domain
- Extract: MX, SPF, DMARC, and DKIM records (try common selectors)
- Analyze each record:
  - Who provides their email?
  - Which third-party services are in their SPF?
  - What is their DMARC policy (none / quarantine / reject)?
  - Does the rua/ruf field reveal internal email addresses?
- Score their email security posture (strong / medium / weak)

### Task 3 — SSL Certificate Chain Investigation
- Pick any target with multiple subdomains
- Use crt.sh to find all certificates issued for that domain
- Find: the certificate with the most SANs → list all domains on that cert
- Search Censys for `parsed.subject.organization: "company name"` 
  → Find any domains not containing the main domain name

### Task 4 — Cloud Asset Discovery
- Choose a well-known tech company
- Generate a bucket name wordlist using the script in Section 4.2
- Run s3scanner against your list
- Check GrayhatWarfare for any indexed public files
- Document your findings (bucket names, status codes, file types found)

### Task 5 — Threat Intelligence Check
- Take 5 IP addresses from a previous Shodan search (Level 4 exercises)
- Check each on: VirusTotal, AlienVault OTX, Pulsedive
- Note any that are flagged as malicious or suspicious
- Cross-reference with Abuse.ch feeds

### Task 6 — CDN Origin Discovery
- Find a website that uses Cloudflare (check with Shodan: `http.waf:"Cloudflare"`)
- Attempt to find the real origin IP using at least 3 methods from Section 8.3
- Document which method worked and what it revealed

---

## 12. Level 6 Summary

```
✅ ASN = the key to an org's full IP range — find it via bgp.he.net / BGPView
✅ RADB WHOIS: '-i origin AS<NUM>' → all announced prefixes
✅ RIR WHOIS (ARIN/RIPE/APNIC) → registration details per region
✅ SSL certs SANs reveal related domains on the same certificate
✅ crt.sh by Organization Name → finds ALL domains, not just the main one
✅ SPF record → full list of email services used by the org
✅ DMARC rua/ruf → internal email addresses leaked in a public DNS record
✅ Shodan CVE search + Vulners → version → vulnerability mapping
✅ S3Scanner + GrayhatWarfare → public cloud bucket discovery
✅ Firebase /.json check → unauthenticated database = critical finding
✅ VirusTotal + OTX + Abuse.ch → threat reputation of IPs and domains
✅ Shodan Monitor → continuous alerts for new services, CVEs, open DBs
✅ PeeringDB → IXP membership + data center presence
✅ Traceroute → transit providers + CDN detection
✅ 6 methods for finding real IP behind Cloudflare

When you are done here → Proceed to Level 7: Business & Financial OSINT
```

---

## 13. Tools Quick Reference

| Tool | Purpose | URL / Command |
|---|---|---|
| **bgp.he.net** | ASN + prefix lookup | bgp.he.net |
| **BGPView** | BGP API (free) | api.bgpview.io |
| **PeeringDB** | IXP + data center presence | peeringdb.com |
| **RIPE/ARIN/APNIC** | Regional IP registry WHOIS | ripe.net / arin.net / apnic.net |
| **crt.sh** | CT log search | crt.sh |
| **SSL Labs** | Certificate + TLS analysis | ssllabs.com/ssltest |
| **testssl.sh** | CLI TLS analysis | `bash testssl.sh example.com` |
| **MXToolbox** | Email record analysis | mxtoolbox.com |
| **s3scanner** | Public S3 bucket check | `s3scanner scan --bucket X` |
| **GrayhatWarfare** | Public cloud bucket search | buckets.grayhatwarfare.com |
| **VirusTotal** | IP/domain/hash reputation | virustotal.com |
| **AlienVault OTX** | Free threat intelligence | otx.alienvault.com |
| **Abuse.ch** | Malware / botnet tracking | urlhaus.abuse.ch |
| **Pulsedive** | Threat intel aggregator | pulsedive.com |
| **Vulners** | CVE + exploit database | vulners.com |
| **Shodan Monitor** | Continuous asset monitoring | monitor.shodan.io |
| **Certspotter** | Certificate issuance alerts | sslmate.com/certspotter |
| **ip-api.com** | IP geolocation | ip-api.com |
| **mtr** | Enhanced traceroute | `mtr --report example.com` |
