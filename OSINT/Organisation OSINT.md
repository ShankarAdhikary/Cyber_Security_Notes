# OSINT — Level 4: Organisation / Infrastructure OSINT
> Organisation OSINT maps everything an entity exposes publicly —
> its domains, subdomains, IP ranges, employees, technology stack,
> documents, cloud assets, and code. This is the foundation of
> penetration testing, bug bounty hunting, and corporate intelligence.
> Every technique here is passive — you query third parties only.

---

## 1. The Organisation OSINT Methodology

### 1.1 What You Are Building

```
TARGET: example.com / "Target Corp"
│
├── IDENTITY LAYER
│   ├── Legal name, registered entity
│   ├── Parent company, subsidiaries, acquisitions
│   └── Key personnel (executives, engineers)
│
├── DOMAIN LAYER
│   ├── Primary domain(s)
│   ├── Subdomains
│   ├── Related / sister domains
│   └── Expired domains
│
├── NETWORK LAYER
│   ├── IP addresses and ranges
│   ├── ASN (Autonomous System Number)
│   └── Cloud provider (AWS, GCP, Azure)
│
├── SERVICE LAYER
│   ├── Web applications
│   ├── APIs
│   ├── Login / admin panels
│   ├── Exposed databases
│   └── IoT / industrial systems
│
├── PEOPLE LAYER
│   ├── Employee names and roles
│   ├── Email addresses
│   └── Social profiles
│
├── DATA LAYER
│   ├── Public documents + metadata
│   ├── GitHub repositories
│   ├── Paste site leaks
│   └── Breach data
│
└── TECHNOLOGY LAYER
    ├── Tech stack (CMS, framework, server)
    ├── Third-party vendors
    └── Security controls (WAF, CDN)
```

### 1.2 Phased Approach

```
Phase 1: Seed Data        →  What do you already know? (company name, one domain)
Phase 2: Ownership        →  Who owns what? (WHOIS, ASN, company registry)
Phase 3: Scope Expansion  →  What else do they own? (related domains, subsidiaries)
Phase 4: Infrastructure   →  Where is it hosted? (IPs, cloud, CDN)
Phase 5: Services         →  What is running? (Shodan, Censys, CT logs)
Phase 6: People           →  Who works there? (LinkedIn, email, Hunter.io)
Phase 7: Data             →  What have they leaked? (GitHub, Google dorks, pastebin)
Phase 8: Correlation      →  Connect everything into a picture
```

---

## 2. Company & Legal Identity Research

Before looking at infrastructure, understand the legal entity — its full name,
parent company, subsidiaries, and registration details.

### 2.1 India — MCA21

```
URL: mca.gov.in/content/mca/global/en/mca/master-data/MDS.html

What it contains:
  ├── Company CIN (Corporate Identity Number)
  ├── Registered address
  ├── Directors and their DIN (Director Identification Number)
  ├── Date of incorporation
  ├── Authorized and paid-up capital
  ├── Annual filings and financial statements
  └── Charges (loans/mortgages registered against the company)

How to search:
  1. Go to mca.gov.in → MCA Services → Master Data → Company/LLP Search
  2. Enter company name or CIN
  3. Download filings from the "Documents" tab

Intelligence value:
  ├── Directors → search each director → find other companies they lead → scope expansion
  ├── Registered address → physical location
  ├── Incorporation date → company age → maturity of security practices
  └── Financial filings → revenue, auditor name, registered office
```

### 2.2 OpenCorporates

```
URL: opencorporates.com

What it is:
  The largest open database of companies — covers 200+ jurisdictions globally.
  Free for basic lookups; API available.

What to search:
  → Company name → returns all matching companies worldwide
  → Officer/director name → all companies that person is registered in
  → Company address → all companies at that address

Intelligence value:
  ├── Find subsidiaries and related companies
  ├── Find other domains registered to the same company
  ├── Cross-border entity mapping (holding companies, offshore entities)
  └── Officer networks → who controls what
```

### 2.3 Crunchbase

```
URL: crunchbase.com

What it contains:
  ├── Funding history (seed, Series A, B, C, IPO)
  ├── Investors and board members
  ├── Acquisitions (what companies they have bought)
  ├── Key employees and leadership
  ├── Technologies used (from their job postings and integrations)
  └── Office locations

Intelligence value:
  ├── Acquisitions → new domains/assets to include in scope
  ├── Funding → size and maturity of the organization
  ├── Investors → often listed alongside board members → key personnel
  └── Technologies → infer stack without touching the target
```

### 2.4 LinkedIn (Company Page)

```
Search: linkedin.com/company/target-company

What it reveals:
  ├── Exact employee count
  ├── Locations / offices
  ├── All employees (search by function, seniority, location)
  ├── Technology skills of employees → infer stack
  ├── Job postings → technologies they are hiring for → stack expansion
  └── Recent hires → growth areas → new infrastructure being built

Job posting intelligence:
  "Experience with AWS, Terraform, Kubernetes required"
  → They run workloads on AWS
  → Infrastructure as code with Terraform
  → Container orchestration with Kubernetes

  "Experience with Splunk SIEM" → they use Splunk for security monitoring
  "Okta SSO administration"     → they use Okta for identity
  "Palo Alto Firewall"          → their firewall vendor
```

### 2.5 Built With / Stack Overflow Insights

```
BuiltWith:    builtwith.com
  → Enter domain → full technology stack
  → Shows: CMS, CDN, analytics, payment processor, email platform, JS libs
  → Shows HISTORICAL technology (what they used to run)
  → Reveals third-party vendors (data processors, cloud services)

Wappalyzer:   wappalyzer.com / browser extension
  → Real-time tech detection on any webpage you visit
  → Categories: CMS, eCommerce, server, JS framework, analytics, security

WhatRuns:     whatruns.com (browser extension)
  → Similar to Wappalyzer; sometimes catches different technologies
```

---

## 3. WHOIS & Domain Intelligence

### 3.1 Domain WHOIS

```bash
# Basic WHOIS
whois example.com

# Key fields to extract:
#   Registrant Organization  →  legal name of owner
#   Registrant Email         →  contact email (pivot for reverse WHOIS)
#   Registrar                →  where the domain is registered
#   Name Server              →  DNS provider (Cloudflare? Route53? Self-hosted?)
#   Creation Date            →  age of the domain
#   Updated Date             →  recent activity

# Example output analysis:
Registrant Organization: Target Corp
Registrant Email:        domains@target.com       ← pivot: reverse WHOIS
Name Server:             NS1.CLOUDFLARE.COM        ← CDN in use
Creation Date:           2005-08-14T12:00:00Z      ← 20-year-old domain
```

### 3.2 IP WHOIS

```bash
# WHOIS an IP address → reveals the organization and IP range it belongs to
whois 93.184.216.34

# Key fields:
#   NetName       →  name of the network block
#   OrgName       →  organization owning the IP range
#   CIDR          →  the full IP range (e.g., 93.184.216.0/24)
#   ASN           →  autonomous system number
```

### 3.3 Reverse WHOIS — Finding All Domains Owned by an Entity

```
Tools:
  ViewDNS.info Reverse WHOIS:   viewdns.info/reversewhois/
  WhoisXML API:                 whoisxmlapi.com/reverse-whois-search
  DomainTools:                  domaintools.com (paid; most complete)
  SpyOnWeb:                     spyonweb.com

Usage:
  Enter the registrant email or organization name
  Returns: all domains registered with that email/org

Example:
  Input:  "domains@targetcorp.com"
  Output: targetcorp.com
          targetcorp.net
          targetcorp.io
          target-internal.com     ← interesting!
          target-staging.com      ← staging environment!
          targetcorp-backup.com   ← backup infrastructure

Each discovered domain → new recon target
```

### 3.4 DNS Records — Full Extraction

```bash
# A record — IPv4 address of the domain
dig example.com A

# AAAA record — IPv6 address
dig example.com AAAA

# MX records — mail servers
dig example.com MX

# NS records — name servers (DNS provider)
dig example.com NS

# TXT records — SPF, DKIM, DMARC, verification tokens
dig example.com TXT

# SOA record — primary NS + admin email
dig example.com SOA

# CNAME — alias record (reveals cloud services)
dig www.example.com CNAME

# SRV records — service locations
dig _sip._tcp.example.com SRV

# ALL records (not all servers honor this)
dig example.com ANY

# Reverse DNS — IP to hostname
dig -x 93.184.216.34

# Query a specific DNS server
dig @8.8.8.8 example.com A          # Query Google's DNS
dig @1.1.1.1 example.com A          # Query Cloudflare's DNS
dig @ns1.example.com example.com A  # Query their own DNS server
```

### 3.5 TXT Record Intelligence

TXT records are a goldmine — they contain verification tokens that reveal
which third-party services the organization uses.

```bash
dig example.com TXT

# Common TXT records and what they reveal:
"v=spf1 include:_spf.google.com include:sendgrid.net ~all"
→ Uses Google Workspace for email
→ Uses SendGrid for marketing email

"MS=ms12345678"
→ Uses Microsoft 365 / Azure AD

"google-site-verification=xxxxxxxxxx"
→ Uses Google Search Console / Google Workspace

"atlassian-domain-verification=xxxxxxxxxx"
→ Uses Jira, Confluence (Atlassian suite)

"facebook-domain-verification=xxxxxxxxxx"
→ Has a verified Facebook business page

"docusign=xxxxxxxxxx"
→ Uses DocuSign for document signing

"stripe-verification=xxxxxxxxxx"
→ Uses Stripe for payment processing

"amazonses:xxxxxxxxxx"
→ Uses Amazon SES for email sending

"_github-challenge-xxxxxxxxxx.example.com"
→ Has a verified GitHub organization
```

### 3.6 Zone Transfer Attempt (AXFR)

A DNS zone transfer copies all DNS records from the primary server.
Misconfigured servers allow this publicly — a critical finding.

```bash
# Step 1: Find the name servers
dig NS example.com

# Step 2: Attempt AXFR against each NS
dig axfr @ns1.example.com example.com
dig axfr @ns2.example.com example.com

# Successful output looks like:
# example.com.          3600  IN  SOA   ns1.example.com. admin.example.com. ...
# example.com.          3600  IN  A     93.184.216.34
# mail.example.com.     3600  IN  A     93.184.216.35
# dev.example.com.      3600  IN  A     10.0.0.5           ← internal IP!
# vpn.example.com.      3600  IN  A     93.184.216.40
# internal.example.com. 3600  IN  A     10.0.0.1           ← internal IP!
# ...
# (entire DNS zone dumped)

# Failed output:
# ; Transfer failed.
# (this is the expected / correct response — server is properly configured)
```

> A successful zone transfer is itself a **critical security finding**.
> It exposes the entire internal DNS structure including private/internal hostnames.

---

## 4. ASN & IP Range Discovery

### 4.1 Why ASN Matters

An Autonomous System Number (ASN) identifies a network operated by a single
organization. Finding the ASN reveals ALL IP ranges the organization owns —
not just the ones used for their public website.

```
Target: "Target Corp"
          ↓
ASN: AS12345  (found via bgp.he.net)
          ↓
IP Ranges: 104.16.0.0/16    (65,536 IPs)
           192.0.2.0/24     (256 IPs)
           198.51.100.0/22  (1,024 IPs)
          ↓
Every IP in these ranges is a potential asset of the target
```

### 4.2 Finding the ASN

```bash
# Method 1: BGP Hurricane Electric
# URL: bgp.he.net
# Enter company name or domain → shows ASN, prefixes, peers

# Method 2: BGPView API
curl https://api.bgpview.io/search?query_term=Target+Corp

# Method 3: WHOIS on a known IP
whois 93.184.216.34
# Look for "OriginAS:" or "origin:" field

# Method 4: ASNLookup
# URL: asnlookup.com → enter company name

# Method 5: dig with ASN CYMRU service
dig TXT 34.216.184.93.origin.asn.cymru.com
# Note: reverse the IP octets!
```

### 4.3 Finding All IP Ranges for an ASN

```bash
# RADB WHOIS — find all prefixes announced by an ASN
whois -h whois.radb.net -- '-i origin AS12345' | grep "route:"

# BGPView API — cleaner output
curl https://api.bgpview.io/asn/12345/prefixes | python3 -m json.tool

# Amass intel mode
amass intel -asn 12345

# Hurricane Electric (web)
# bgp.he.net/AS12345#_prefixes
# Click "Prefixes" tab → see all IPv4 and IPv6 ranges

# Build a target IP list from CIDR ranges
# Install prips (prints all IPs in a range)
sudo apt install prips
prips 104.16.0.0/16 > all_ips_in_range.txt
# Warning: /16 = 65,536 IPs — handle with care
```

### 4.4 Amass Intel Mode — Org-Wide Discovery

```bash
# Discover ASN from organization name
amass intel -org "Target Corp"

# Output:
# 12345, TARGET CORP, 104.16.0.0/16
# 12346, TARGET CORP CDN, 198.51.100.0/22

# Discover all domains associated with an ASN
amass intel -asn 12345

# Reverse WHOIS via Amass
amass intel -whois -d example.com

# Full intel sweep (org name + WHOIS + ASN)
amass intel -org "Target Corp" -whois -d example.com
```

---

## 5. Subdomain Enumeration

### 5.1 Why Subdomains Matter

```
www.example.com        →  Main website
mail.example.com       →  Email server
vpn.example.com        →  VPN gateway (high value target)
dev.example.com        →  Development environment (often less hardened)
staging.example.com    →  Staging environment (may have debug enabled)
admin.example.com      →  Admin panel
api.example.com        →  API endpoint
jenkins.example.com    →  CI/CD server (often misconfigured)
jira.example.com       →  Internal project tracker
gitlab.example.com     →  Internal code repository
grafana.example.com    →  Monitoring dashboard (may be unauthenticated)
kibana.example.com     →  Elasticsearch dashboard (often unauthenticated)
s3.example.com         →  May be an S3 bucket proxy
backup.example.com     →  Backup system
legacy.example.com     →  Old system — possibly unpatched
```

### 5.2 Passive Subdomain Enumeration

```bash
# Subfinder — queries CT logs, passive DNS, APIs
subfinder -d example.com -silent -o subs_sf.txt

# With all sources (needs API keys for best results)
subfinder -d example.com -all -silent -o subs_sf.txt

# Multiple domains
subfinder -dL domains.txt -silent -o subs_all.txt

# Amass passive mode
amass enum -passive -d example.com -o subs_amass.txt

# theHarvester
theHarvester -d example.com -b all -f harvester_output

# crt.sh API
curl -s "https://crt.sh/?q=%.example.com&output=json" \
  | python3 -c "
import sys, json
data = json.load(sys.stdin)
names = set()
for entry in data:
    for name in entry['name_value'].split('\n'):
        names.add(name.replace('*.', ''))
print('\n'.join(sorted(names)))
" > subs_crt.txt

# Merge all sources and deduplicate
cat subs_sf.txt subs_amass.txt subs_crt.txt | sort -u > all_subs.txt
echo "[*] Total unique subdomains: $(wc -l < all_subs.txt)"
```

### 5.3 Configuring Subfinder API Keys

```bash
# Config location
nano ~/.config/subfinder/provider-config.yaml

# Add your keys (free accounts available for all):
securitytrails:
  - YOUR_SECURITYTRAILS_KEY     # securitytrails.com
shodan:
  - YOUR_SHODAN_KEY             # shodan.io
censys:
  - YOUR_CENSYS_API_ID
  - YOUR_CENSYS_SECRET          # search.censys.io
virustotal:
  - YOUR_VT_KEY                 # virustotal.com
binaryedge:
  - YOUR_BE_KEY                 # binaryedge.io
github:
  - YOUR_GITHUB_TOKEN           # github.com/settings/tokens
```

### 5.4 DNS Resolution — Filter Live Hosts

Raw subdomain lists contain many dead entries.
Resolve them to find which ones are actually alive.

```bash
# dnsx — fast DNS resolver
cat all_subs.txt | dnsx -silent -o live_subs.txt

# With record types
cat all_subs.txt | dnsx -a -cname -resp -silent

# Check for wildcard DNS (false positives)
# A wildcard returns the same IP for ANY subdomain
dig randomxyz123abc.example.com A
# If this returns an IP → wildcard DNS is configured
# → All subdomains will "resolve" even if they don't exist

# puredns — handles wildcard filtering automatically
puredns resolve all_subs.txt -r resolvers.txt -w live_subs_clean.txt
```

### 5.5 Web Probing — Find HTTP/S Services

```bash
# httpx — probe which subdomains serve web content
cat live_subs.txt | httpx -silent -status-code -title -content-length

# More detail — tech detection
cat live_subs.txt | httpx -silent -status-code -title -tech-detect -web-server

# Save to file
cat live_subs.txt | httpx -silent -status-code -title -o web_hosts.txt

# Flag interesting findings
cat web_hosts.txt | grep -iE "jenkins|gitlab|grafana|kibana|jira|admin|dashboard|api|dev|staging|test|internal"

# Screenshot all live web hosts (visual triage)
# EyeWitness:
python3 EyeWitness.py --web -f live_subs.txt --no-prompt -d screenshots/
```

### 5.6 Subdomain Takeover Detection

```bash
# subzy — automated takeover detection
go install github.com/PentestPad/subzy@latest
subzy run --targets live_subs.txt

# What it checks:
# CNAME records that point to unclaimed external services:
#   *.github.io           →  GitHub Pages
#   *.herokuapp.com       →  Heroku
#   *.s3.amazonaws.com    →  AWS S3
#   *.azurewebsites.net   →  Azure
#   *.netlify.app         →  Netlify
#   *.vercel.app          →  Vercel
#   *.surge.sh            →  Surge.sh

# Manual check for takeover indicators:
# Visit the subdomain → look for these messages:
# "There is no app at this address"   →  Heroku (takeover possible)
# "NoSuchBucket"                      →  AWS S3 (takeover possible)
# "Repository not found"              →  GitHub Pages (takeover possible)
# "404 Not Found" on nginx default    →  may or may not be takeover
```

---

## 6. Certificate Transparency (CT) Logs

### 6.1 What CT Logs Reveal

Every TLS certificate issued for a domain is permanently logged in public CT logs.
This includes certificates for internal tools, staging environments, and subdomains
that have never been publicly linked or indexed.

```bash
# crt.sh — primary interface
# Web: https://crt.sh/?q=%.example.com

# API query
curl -s "https://crt.sh/?q=%.example.com&output=json" \
  | jq -r '.[].name_value' \
  | sed 's/\*\.//g' \
  | sort -u

# Filter only subdomains (exclude the apex domain)
curl -s "https://crt.sh/?q=%.example.com&output=json" \
  | jq -r '.[].name_value' \
  | grep -v "^example.com$" \
  | sed 's/\*\.//g' \
  | sort -u > ct_subs.txt

# Also search for the org name in certificate fields
# Censys (web): parsed.subject.organization: "Target Corp"
# This finds certificates issued for domains NOT containing the main domain
# (e.g., a sister brand or recent acquisition)
```

### 6.2 What to Look For in CT Results

```
dev-api.example.com          →  Development API (often unauthenticated)
internal-tools.example.com   →  Internal tooling (may be publicly accessible)
*.internal.example.com       →  Wildcard for internal systems
vpn2.example.com             →  Secondary VPN endpoint
sso.example.com              →  Single Sign-On portal
corp.example.com             →  Corporate portal
rd.example.com               →  R&D infrastructure
uat.example.com              →  User Acceptance Testing environment
```

---

## 7. Device & Service Search Engines

### 7.1 Shodan — Complete Reference

```bash
# CLI setup
pip3 install shodan
shodan init YOUR_API_KEY

# Basic searches
shodan search 'org:"Target Corp"'
shodan search 'ssl.cert.subject.cn:"example.com"'
shodan search 'hostname:"example.com"'
shodan search 'net:104.16.0.0/16'

# Domain intelligence
shodan domain example.com
# Returns: subdomains, DNS records, all Shodan-found IPs

# Host details
shodan host 93.184.216.34

# Count results before full search
shodan count 'org:"Target Corp"'

# Download results
shodan download results 'org:"Target Corp"'
shodan parse --fields ip_str,port,org results.json.gz

# Stats / facets (top ports, countries, orgs)
shodan stats --facets port,country 'org:"Target Corp"'
```

**Key Shodan Filter Reference:**

```
Category      Filter                    Example
──────────────────────────────────────────────────────────────
Identity      org:                      org:"Target Corp"
              hostname:                 hostname:"example.com"
              ip:                       ip:93.184.216.34
              net:                      net:104.16.0.0/16
              asn:                      asn:AS13335

Service       port:                     port:8443
              product:                  product:"MongoDB"
              version:                  product:"Apache" version:"2.4.49"

Location      country:                  country:IN
              city:                     city:"Mumbai"
              isp:                      isp:"Reliance Jio"

HTTP          http.title:               http.title:"Grafana"
              http.html:                http.html:"x-jenkins"
              http.status:              http.status:200
              http.component:           http.component:"WordPress"
              http.favicon.hash:        http.favicon.hash:-1234567890
              http.waf:                 http.waf:"Cloudflare"
              http.server:              http.server:"nginx"

SSL           ssl:                      ssl:"example.com"
              ssl.cert.subject.cn:      ssl.cert.subject.cn:"example.com"
              ssl.cert.expired:         ssl.cert.expired:true
              ssl.alpn:                 ssl.alpn:"h2"
              ssl.version:              ssl.version:"TLSv1"

Vulnerability vuln:                     vuln:CVE-2021-44228  (paid)

Time          before:                   before:01/01/2025
              after:                    after:01/06/2024
```

**Header-Based Dorks (detect technologies):**

```
"x-jenkins"                    →  Jenkins CI/CD server
"kbn-name: kibana"             →  Kibana dashboard
"X-elastic-product: Elasticsearch" → Elasticsearch instance
"x-drupal-cache"               →  Drupal CMS
"X-Magento-Cache-Debug"        →  Magento eCommerce
"Set-Cookie: PHPSESSID"        →  PHP application
"Set-Cookie: _gitlab_session"  →  GitLab instance
"Set-Cookie: webvpn"           →  Cisco WebVPN portal
"Set-Cookie: mongoexpress="    →  Mongo Express (DB admin)
"Set-Cookie: phpMyAdmin"       →  phpMyAdmin (DB admin)
"X-App-Name: kibana"           →  Kibana (alternate header)
"access-control-allow-origin: *" → Permissive CORS
"WWW-Authenticate"             →  HTTP basic auth
"x-forwarded-for"              →  Behind a proxy/load balancer
```

### 7.2 Favicon Hash Attack

Every web application has a favicon (the small browser tab icon). Shodan stores
a hash of each favicon. Searching by this hash finds ALL servers running the same
application — including those not linked to the main domain.

```bash
# Step 1: Get the favicon
curl -s https://example.com/favicon.ico -o favicon.ico
# OR find it from the page source: <link rel="icon" href="/favicon.ico">

# Step 2: Calculate the MurmurHash3 hash
python3 << 'EOF'
import mmh3
import base64

with open('favicon.ico', 'rb') as f:
    favicon = base64.encodebytes(f.read())

hash_value = mmh3.hash(favicon)
print(f"Favicon hash: {hash_value}")
EOF

# Install mmh3 if needed: pip3 install mmh3

# Step 3: Search Shodan
shodan search "http.favicon.hash:<HASH_VALUE>"

# Step 4: Result = all servers worldwide running that same application
# Filter to your target's ASN or IP range to find their instances
shodan search "http.favicon.hash:<HASH_VALUE> org:'Target Corp'"
```

### 7.3 Censys

```
URL: search.censys.io

# Host search
autonomous_system.name: "Target Corp"
services.port: 8443
ip: 93.184.216.34
labels: iot

# Certificate search
parsed.names: "*.example.com"
parsed.subject.common_name: "example.com"
parsed.subject.organization: "Target Corp"
parsed.issuer.common_name: "Let's Encrypt"

# Combined queries
autonomous_system.name: "Target Corp" and services.port: 27017
# → MongoDB instances on Target Corp's infrastructure

# API
pip3 install censys
censys config   # set API keys from search.censys.io/account
censys search 'autonomous_system.name: "Target Corp"' --index-type hosts
censys view hosts 93.184.216.34
```

### 7.4 FOFA

```
URL: fofa.info

# Key operators
domain="example.com"
cert="example.com"
org="Target Corp"
title="Grafana"
body="x-jenkins"
header="X-Powered-By: PHP/7.4"
port="8080"
country="IN"
city="Mumbai"

# Combined
cert="example.com" && port="8443"
domain="example.com" && title="Login"
org="Target Corp" && (title="Kibana" || title="Grafana")

# Logical operators
&&   AND
||   OR
!=   NOT (exclude)
```

---

## 8. Google Dorking for Infrastructure

### 8.1 Subdomain and File Discovery

```
# Find subdomains not indexed on main site
-site:www.example.com site:*.example.com

# All pages on the domain
site:example.com

# Specific file types
site:example.com filetype:pdf
site:example.com ext:xls OR ext:xlsx OR ext:csv
site:example.com ext:sql OR ext:bak OR ext:log
site:example.com ext:env OR ext:config OR ext:conf

# Exposed sensitive files
site:example.com filetype:pdf "confidential"
site:example.com filetype:xls "password"
site:example.com "DB_PASSWORD" OR "API_KEY" OR "SECRET_KEY"
```

### 8.2 Login Pages and Admin Panels

```
site:example.com inurl:admin
site:example.com inurl:login
site:example.com inurl:dashboard
site:example.com inurl:portal
site:example.com inurl:wp-admin
site:example.com intitle:"admin panel"
site:example.com intitle:"login" inurl:login
```

### 8.3 Directory Listings

```
site:example.com intitle:"index of"
site:example.com intitle:"index of /" "backup"
site:example.com intitle:"index of /" ".sql"
site:example.com intitle:"index of /" "config"
site:example.com intitle:"index of /" "private"
```

### 8.4 Exposed Technology

```
site:example.com "Powered by WordPress"
site:example.com intitle:"phpMyAdmin"
site:example.com inurl:phpmyadmin
site:example.com "Jenkins" intitle:"Dashboard"
site:example.com intitle:"Grafana"
site:example.com intitle:"Kibana"
```

### 8.5 Error Messages (Reveal Stack Info)

```
site:example.com "Warning: mysql_connect()"
site:example.com "ORA-01756"            ← Oracle DB error
site:example.com "Microsoft OLE DB"     ← MSSQL error
site:example.com "Fatal error:"         ← PHP fatal error
site:example.com "stack trace"          ← Application stack trace
site:example.com "SQL syntax error"
```

### 8.6 Paste Sites and Leaks

```
site:pastebin.com "example.com"
site:pastebin.com "example.com" "password"
site:pastebin.com "@example.com"
site:gist.github.com "example.com"
site:paste.ee "example.com"
site:hastebin.com "example.com"
```

### 8.7 Google Hacking Database (GHDB)

```
URL: exploit-db.com/google-hacking-database

Categories:
  Files Containing Passwords
  Sensitive Directories
  Web Server Detection
  Vulnerable Files
  Vulnerable Servers
  Error Messages
  Files Containing Juicy Info
  Login Portals
  Various Online Devices
  Advisories and Vulnerabilities

Usage:
  Browse categories → copy a dork query
  Add: site:example.com to scope it to your target
  Adapt the query as needed
```

---

## 9. GitHub & Code Repository Recon

### 9.1 GitHub Dorking for Organisation Data

```
# Search within organization's repos
org:targetcompany

# Find sensitive files
org:targetcompany filename:.env
org:targetcompany filename:config.yml
org:targetcompany filename:database.yml
org:targetcompany filename:settings.py
org:targetcompany filename:secrets.json
org:targetcompany filename:.aws/credentials
org:targetcompany filename:id_rsa
org:targetcompany filename:.htpasswd

# Find leaked credentials
org:targetcompany "password"
org:targetcompany "api_key"
org:targetcompany "secret_key"
org:targetcompany "access_token"
org:targetcompany "AKIA"               ← AWS Access Key ID prefix
org:targetcompany "aws_secret_access_key"
org:targetcompany "mongodb://"
org:targetcompany "postgres://"
org:targetcompany "BEGIN RSA PRIVATE KEY"

# Find internal infrastructure references
org:targetcompany "internal.example.com"
org:targetcompany "192.168."
org:targetcompany "10.0.0."
org:targetcompany ".example.com"

# Find JWT secrets
org:targetcompany "JWT_SECRET"
org:targetcompany "jwt_secret"
```

### 9.2 TruffleHog — Scanning Git History

```bash
# Install
pip3 install trufflehog

# Scan a specific GitHub org repo
trufflehog git https://github.com/targetcompany/repo.git

# Scan with verification (only report confirmed active secrets)
trufflehog git https://github.com/targetcompany/repo.git --only-verified

# Scan all repos of an org (requires GitHub token)
trufflehog github --org=targetcompany --token=ghp_YOUR_TOKEN

# Scan local cloned repo
git clone https://github.com/targetcompany/repo.git
trufflehog git file://./repo

# Scan a specific commit range
trufflehog git https://github.com/org/repo.git \
  --since-commit=COMMIT_HASH --branch=main
```

### 9.3 Gitleaks

```bash
# Download from: github.com/gitleaks/gitleaks/releases
wget https://github.com/gitleaks/gitleaks/releases/latest/download/gitleaks_linux_amd64 -O gitleaks
chmod +x gitleaks

# Scan a cloned repo
./gitleaks detect --source /path/to/repo -v

# Scan current directory
./gitleaks detect -v

# Save report
./gitleaks detect --source /path/to/repo --report-path gitleaks_report.json

# Scan with custom config
./gitleaks detect --config gitleaks.toml
```

### 9.4 Grep.app

```
URL: grep.app

A fast regex search across all public GitHub repositories.
Useful when you want to search for a domain or technology across many repos at once.

Search examples:
  example.com password
  "example.com" AND "api_key"
  "internal.example.com"
```

---

## 10. Employee & Email Harvesting

### 10.1 Hunter.io

```
URL: hunter.io

Features:
  Domain search: enter domain → see all emails found publicly
  Email finder: enter name + domain → predict email address
  Email verifier: confirm if an address is valid
  Bulk tasks: upload a list of names → get emails

API usage:
curl "https://api.hunter.io/v2/domain-search?domain=example.com&api_key=YOUR_KEY"
curl "https://api.hunter.io/v2/email-finder?domain=example.com&first_name=John&last_name=Doe&api_key=YOUR_KEY"

Free tier: 25 searches/month
```

### 10.2 CrossLinked — LinkedIn Email Extraction

```bash
# Install
pip3 install crosslinked

# Harvest emails using LinkedIn (no login required)
crosslinked -f "{first}.{last}@example.com" "Company Name"

# Common format variations to try
crosslinked -f "{first}@example.com" "Company Name"
crosslinked -f "{f}{last}@example.com" "Company Name"
crosslinked -f "{first}{last}@example.com" "Company Name"
crosslinked -f "{first}_{last}@example.com" "Company Name"

# Output: names.txt (employee names) and emails.txt (constructed addresses)
```

### 10.3 theHarvester

```bash
# Install
pip3 install theHarvester

# Full harvest (all sources)
theHarvester -d example.com -b all

# Specific sources
theHarvester -d example.com -b google,bing,linkedin,shodan,virustotal,certspotter

# Save HTML report
theHarvester -d example.com -b all -f report.html

# Limit results per source
theHarvester -d example.com -b google -l 500

# What it returns:
#   Emails: john.doe@example.com, admin@example.com
#   Hosts: mail.example.com, dev.example.com
#   IPs: 93.184.216.34
#   LinkedIn profiles (names and titles)
```

---

## 11. Document & Metadata Recon

### 11.1 Finding Public Documents

```
# Google dorks for documents
site:example.com filetype:pdf
site:example.com filetype:doc OR filetype:docx
site:example.com filetype:xls OR filetype:xlsx
site:example.com filetype:ppt OR filetype:pptx
site:example.com filetype:txt

# Sensitive document types
site:example.com filetype:pdf "internal use only"
site:example.com filetype:xlsx "employee"
site:example.com filetype:docx "confidential"
```

### 11.2 metagoofil — Automated Metadata Harvesting

```bash
# Basic run
metagoofil -d example.com -t pdf,doc,xls,ppt -o output/

# More results
metagoofil -d example.com -t pdf,doc,xls,ppt,docx,xlsx,pptx -l 200 -n 20 -o output/

# Flags:
# -d   target domain
# -t   file types (comma-separated)
# -l   number of results to search (default 200)
# -n   number of files to download (default 5)
# -o   output directory

# After running:
# Check output/ for:
#   metadata.html  →  visual report of all extracted metadata
#   Usernames list →  IT account names
#   Software list  →  all software versions found
#   Path list      →  internal server paths
```

### 11.3 Extracting Metadata Manually

```bash
# ExifTool on downloaded documents
exiftool *.pdf > all_pdf_metadata.txt
exiftool *.docx >> all_pdf_metadata.txt

# Key fields to extract
exiftool -Author -Creator -LastModifiedBy -Company -Template *.pdf

# Look for internal paths in Template field:
# Template: C:\Users\jdoe\AppData\Roaming\Microsoft\Templates\corporate.dotx
#            ↑ username revealed: jdoe
#            → AD username = jdoe → try jdoe@example.com

# Look for server paths:
# Template: \\fileserver01\shared\templates\corp_template.dotx
#            ↑ internal file server: fileserver01
```

---

## 12. Passive DNS & Historical Records

### 12.1 SecurityTrails

```
URL: securitytrails.com

Free account: 50 API calls/month

Features:
  Historical DNS: past A, MX, NS records with dates
  Subdomains: passive DNS-sourced subdomain list
  WHOIS history: registrant changes over time
  Associated domains: other domains on the same IP

Web usage:
  securitytrails.com/domain/example.com/dns
  → Click "Historical Data" → see all past IPs this domain has pointed to

API usage:
  curl --request GET \
    --url "https://api.securitytrails.com/v1/domain/example.com" \
    --header "apikey: YOUR_KEY"

  curl --request GET \
    --url "https://api.securitytrails.com/v1/domain/example.com/subdomains" \
    --header "apikey: YOUR_KEY"

Intelligence from history:
  Old IPs → may still host dev/staging environments
  Old name servers → infer past CDN/hosting providers
  Registrant email changes → ownership changes
```

### 12.2 DNSDumpster

```
URL: dnsdumpster.com

Free, no account needed.

What it returns:
  ├── Full DNS record dump (A, MX, NS, TXT, SOA)
  ├── Subdomains found via passive DNS
  ├── Banner grabbing (HTTP headers) for each host
  ├── IP geolocation
  └── Visual network map (downloadable as image)

Usage:
  Enter domain → click "Search" → view results
  Download the visual map → include in your report
```

---

## 13. Cloud Asset Discovery

### 13.1 S3 Bucket Enumeration

AWS S3 buckets are frequently misconfigured to allow public access.
A publicly accessible S3 bucket can contain databases, backups, source code, or user data.

```bash
# Common bucket naming patterns to try
# Buckets are named: <company>-<purpose>
# Accessible at: https://<bucket>.s3.amazonaws.com

Patterns to try:
  example-com
  example-backup
  example-dev
  example-staging
  example-assets
  example-media
  example-logs
  example-data
  example-internal
  target-prod
  targetcorp-backup

# GrayhatWarfare — search public buckets (web)
URL: buckets.grayhatwarfare.com
Search: company name or domain

# S3Scanner — automated bucket checking
pip3 install s3scanner
s3scanner scan --bucket example-backup
s3scanner scan --bucket-file bucket_list.txt  # scan many buckets

# What to look for in a public bucket:
# ├── Database dumps (.sql, .dump)
# ├── Backup archives (.zip, .tar.gz)
# ├── Source code
# ├── Configuration files
# ├── User-uploaded data
# └── Log files
```

### 13.2 Google Cloud & Azure

```bash
# Google Cloud Storage buckets
# Pattern: https://storage.googleapis.com/<bucket-name>
# Try same naming patterns as S3

# Azure Blob Storage
# Pattern: https://<account>.blob.core.windows.net/<container>
# Try: example.blob.core.windows.net
#      targetcorp.blob.core.windows.net

# Firebase open database
# URL pattern: https://<app-name>.firebaseio.com/.json
# If it returns JSON data without auth → open database

curl https://targetapp.firebaseio.com/.json
# If returns data → critical finding (unauthenticated Firebase DB)
# If returns null or permission error → properly secured
```

### 13.3 Identifying Cloud Provider from IP

```bash
# Download cloud provider IP ranges
# AWS:
curl -s https://ip-ranges.amazonaws.com/ip-ranges.json | \
  python3 -c "import sys,json; [print(p['ip_prefix']) for p in json.load(sys.stdin)['prefixes']]" \
  > aws_ranges.txt

# Check if a target IP is in AWS
grep "93.184.216." aws_ranges.txt

# Alternative: ipinfo.io
curl https://ipinfo.io/93.184.216.34
# Returns: org field shows "AMAZON-02" or "CLOUDFLARENET" etc.

# Or simply:
curl https://ipinfo.io/93.184.216.34/org
```

---

## 14. The Complete Organisation Recon Pipeline

```bash
#!/bin/bash
# Full passive recon pipeline for a target domain
TARGET="example.com"
OUTPUT_DIR="recon_$TARGET"
mkdir -p $OUTPUT_DIR

echo "[*] Starting recon for $TARGET"
echo "[*] Output directory: $OUTPUT_DIR"

# Phase 1: DNS Records
echo "[1/8] Extracting DNS records..."
dig $TARGET A MX NS TXT SOA +noall +answer > $OUTPUT_DIR/dns_records.txt
for ns in $(dig NS $TARGET +short); do
  echo "[*] Attempting zone transfer on $ns"
  dig axfr @$ns $TARGET >> $OUTPUT_DIR/zone_transfer_attempt.txt 2>&1
done

# Phase 2: WHOIS
echo "[2/8] WHOIS lookup..."
whois $TARGET > $OUTPUT_DIR/whois_domain.txt

# Phase 3: CT Logs
echo "[3/8] Certificate Transparency logs..."
curl -s "https://crt.sh/?q=%.$TARGET&output=json" \
  | python3 -c "
import sys, json
data = json.load(sys.stdin)
names = set()
for entry in data:
    for name in entry['name_value'].split('\n'):
        names.add(name.replace('*.', ''))
print('\n'.join(sorted(names)))" > $OUTPUT_DIR/ct_subdomains.txt
echo "[*] CT log subdomains: $(wc -l < $OUTPUT_DIR/ct_subdomains.txt)"

# Phase 4: Passive Subdomain Enumeration
echo "[4/8] Passive subdomain enumeration..."
subfinder -d $TARGET -silent -o $OUTPUT_DIR/subs_subfinder.txt
amass enum -passive -d $TARGET -o $OUTPUT_DIR/subs_amass.txt

# Phase 5: Merge and Deduplicate
echo "[5/8] Merging subdomain lists..."
cat $OUTPUT_DIR/subs_subfinder.txt \
    $OUTPUT_DIR/subs_amass.txt \
    $OUTPUT_DIR/ct_subdomains.txt \
  | sort -u > $OUTPUT_DIR/all_subdomains.txt
echo "[*] Total unique subdomains: $(wc -l < $OUTPUT_DIR/all_subdomains.txt)"

# Phase 6: DNS Resolution
echo "[6/8] Resolving live subdomains..."
cat $OUTPUT_DIR/all_subdomains.txt \
  | dnsx -silent > $OUTPUT_DIR/live_subdomains.txt
echo "[*] Live subdomains: $(wc -l < $OUTPUT_DIR/live_subdomains.txt)"

# Phase 7: Web Probe
echo "[7/8] Probing web services..."
cat $OUTPUT_DIR/live_subdomains.txt \
  | httpx -silent -status-code -title -web-server \
  > $OUTPUT_DIR/web_hosts.txt

# Phase 8: Highlight interesting findings
echo "[8/8] Flagging interesting hosts..."
cat $OUTPUT_DIR/web_hosts.txt \
  | grep -iE "jenkins|gitlab|grafana|kibana|jira|admin|dashboard|staging|dev|internal|api" \
  > $OUTPUT_DIR/interesting_hosts.txt

echo ""
echo "=== RECON COMPLETE ==="
echo "All subdomains:      $(wc -l < $OUTPUT_DIR/all_subdomains.txt)"
echo "Live subdomains:     $(wc -l < $OUTPUT_DIR/live_subdomains.txt)"
echo "Web hosts:           $(wc -l < $OUTPUT_DIR/web_hosts.txt)"
echo "Interesting hosts:   $(wc -l < $OUTPUT_DIR/interesting_hosts.txt)"
echo "Output saved to:     $OUTPUT_DIR/"
```

---

## 15. Level 4 Practice Tasks

### Task 1 — Company Identity Mapping
- Pick any publicly listed company (Infosys, TCS, Wipro, etc.)
- Find: CIN on MCA21, entry on OpenCorporates, Crunchbase profile
- Map: subsidiaries (at least 3), directors (at least 3), office locations
- From each subsidiary → check if they have a separate domain

### Task 2 — Full DNS Intelligence
- Choose any large website (news site, e-commerce, tech company)
- Extract: all DNS record types using dig
- Analyze TXT records → list all third-party services revealed
- Attempt a zone transfer (it will fail — document the attempt and result)
- Run WHOIS on the domain and the primary IP — find the ASN

### Task 3 — Subdomain Enumeration Pipeline
- Run the full pipeline from Section 14 on a domain you own
  OR on a bug bounty target from HackerOne that allows passive recon
- Document: total subdomains found, how many are live, any interesting findings

### Task 4 — Shodan Organisation Search
- Search Shodan for a large company: `org:"Microsoft"` or `org:"Google LLC"`
- Find: total number of hosts, top 5 open ports, any exposed services
- Search for their subdomains via: `ssl.cert.subject.cn:"microsoft.com"`
- Flag any exposed admin panels, databases, or unencrypted services

### Task 5 — GitHub Secret Hunt
- Search GitHub for: `org:<any public company> filename:.env`
- Document what you find (without testing or using any credentials)
- Run TruffleHog on any public repo from the company
- Report: what types of secrets are exposed, how they could be remediated

### Task 6 — Document Metadata
- Find 5 PDFs on any organization's public website
- Run ExifTool on each
- List: all unique author names found, all software versions, any internal paths
- What does this tell you about the organization?

---

## 16. Level 4 Summary

```
✅ Map the organization: legal entity → subsidiaries → domains → IPs → services
✅ WHOIS (domain + IP) → registrant, ASN, IP range
✅ ASN → full IP ranges owned by the target (bgp.he.net, RADB)
✅ Reverse WHOIS → all domains registered to the same entity
✅ DNS: dig all record types; TXT reveals all third-party services
✅ Zone transfer (AXFR) attempt on every NS — a success is a critical finding
✅ CT logs (crt.sh): %.example.com → all subdomains that ever had a cert
✅ Passive subdomain enum: Subfinder + Amass + CT logs + theHarvester → merge
✅ DNS resolve (dnsx) → filter live hosts → web probe (httpx)
✅ Shodan: org:, ssl.cert.subject.cn:, favicon hash, header-based dorks
✅ GitHub: org: dorks for secrets + TruffleHog on repos
✅ Google dorks: directory listings, exposed files, error messages, admin panels
✅ S3 / cloud bucket naming → GrayhatWarfare → public bucket search
✅ Document metadata: metagoofil → employee names, software, internal paths
✅ Job postings → tech stack inference

When you are done here → Proceed to Level 5: GeoINT
```

---

## 17. Tools Quick Reference

| Tool | Purpose | Install | Key Command |
|---|---|---|---|
| **subfinder** | Passive subdomain enum | `go install` | `subfinder -d example.com -silent` |
| **amass** | Subdomain + org recon | `go install` | `amass enum -passive -d example.com` |
| **dnsx** | DNS resolution | `go install` | `cat subs.txt \| dnsx -silent` |
| **httpx** | Web probing | `go install` | `cat subs.txt \| httpx -status-code -title` |
| **subzy** | Takeover detection | `go install` | `subzy run --targets subs.txt` |
| **theHarvester** | Email + subdomain harvest | `pip3 install` | `theHarvester -d example.com -b all` |
| **crosslinked** | LinkedIn email harvest | `pip3 install` | `crosslinked -f "{f}{last}@co.com" "Co"` |
| **trufflehog** | Git secret scanning | `pip3 install` | `trufflehog git <url>` |
| **gitleaks** | Git secret scanning | binary | `gitleaks detect --source <path>` |
| **shodan** | Device/service search | `pip3 install` | `shodan search 'org:"Target"'` |
| **metagoofil** | Document metadata | `apt install` | `metagoofil -d example.com -t pdf,doc` |
| **s3scanner** | S3 bucket checking | `pip3 install` | `s3scanner scan --bucket example-backup` |
| **dig** | DNS queries | built-in | `dig example.com TXT` |
| **whois** | WHOIS lookup | built-in | `whois example.com` |
| **puredns** | Wildcard-safe DNS resolve | `go install` | `puredns resolve subs.txt` |
| **EyeWitness** | Web screenshots | `git clone` | `python3 EyeWitness.py --web -f urls.txt` |
