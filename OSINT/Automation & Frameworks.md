# OSINT — Level 9: Automation & Frameworks
> At this level you stop running tools one at a time and start building
> systems. Automation means chaining tools into pipelines, scheduling
> continuous monitoring, writing custom scripts, and using OSINT
> frameworks that do multi-source correlation automatically. This is
> what separates a practitioner who can investigate one target slowly
> from one who can monitor hundreds of targets continuously and surface
> findings the moment they appear.

---

## 1. Why Automation Matters

### 1.1 The Scale Problem

```
Manual OSINT (one target, one session):
  Subfinder → manually check output → manually run httpx →
  manually check Shodan → manually check GitHub → done.
  Time: 2-3 hours per target.

Automated OSINT (continuous, multiple targets):
  Pipeline runs every 24 hours on 50 targets:
    → New subdomains appear → you are alerted within minutes
    → New open port detected → alert fires immediately
    → New cert issued for target domain → you see it before the defender does
    → New GitHub commit with secrets → detected and reported automatically
  Time invested: 4 hours to build the pipeline. Then zero.
```

### 1.2 What to Automate

```
✅ Good candidates for automation:
  → Subdomain enumeration (run nightly — new assets appear constantly)
  → Certificate monitoring (new certs = new infrastructure)
  → Port/service change detection (new service = new attack surface)
  → GitHub secret scanning (new commits may contain secrets)
  → Shodan monitor alerts (CVEs, open databases, new services)
  → URL collection from archives (growing corpus of endpoints)
  → Nuclei scans on new subdomains (template-based quick checks)

❌ Not suitable for full automation:
  → Manual geolocation analysis (requires human judgment)
  → Social media cross-platform correlation (context-dependent)
  → Legal/business record research (interpretation required)
  → Anything requiring authorization decisions mid-run
```

### 1.3 Automation Stack Overview

```
Layer 1: Individual Tools
  subfinder, amass, httpx, nuclei, gau, trufflehog...

Layer 2: Pipelines (chained tools)
  subfinder | dnsx | httpx | nuclei
  Bash scripts / Makefiles / Python scripts

Layer 3: Orchestration Frameworks
  recon-ng, SpiderFoot, Maltego
  Custom Python with async/concurrency

Layer 4: Scheduling & Alerting
  Cron jobs, GitHub Actions, notify tool
  Discord/Slack/Telegram webhooks for findings
```

---

## 2. Building OSINT Pipelines in Bash

### 2.1 The Unix Pipeline Philosophy

```bash
# OSINT tools are designed to work together via stdin/stdout
# Each tool reads from stdin, processes, writes to stdout

# Basic chain:
subfinder -d example.com -silent | dnsx -silent | httpx -silent

# Longer chain:
subfinder -d example.com -silent \
  | dnsx -silent \
  | httpx -silent -status-code -title \
  | grep "\[200\]" \
  | tee live_200.txt \
  | nuclei -tags exposure -silent

# Save intermediate results while continuing the chain
subfinder -d example.com -silent \
  | tee subdomains.txt \
  | dnsx -silent \
  | tee live_dns.txt \
  | httpx -silent -status-code \
  | tee web_hosts.txt
```

### 2.2 The Notify Tool — Alerting to Discord/Slack/Telegram

```bash
# Install
go install github.com/projectdiscovery/notify/cmd/notify@latest

# Configure: ~/.config/notify/provider-config.yaml
cat > ~/.config/notify/provider-config.yaml << 'EOF'
discord:
  - id: "osint-alerts"
    discord_channel: "osint-findings"
    discord_username: "OSINT Bot"
    discord_format: "{{data}}"
    discord_webhook_url: "https://discord.com/api/webhooks/YOUR_WEBHOOK_URL"

slack:
  - id: "slack-alerts"
    slack_channel: "#osint"
    slack_username: "OSINT Bot"
    slack_format: "{{data}}"
    slack_webhook_url: "https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK"

telegram:
  - id: "tg-alerts"
    telegram_api_key: "YOUR_BOT_API_KEY"
    telegram_chat_id: "YOUR_CHAT_ID"
    telegram_format: "{{data}}"
EOF

# Send any tool output to Discord/Slack/Telegram
echo "New subdomain found: dev.example.com" | notify -id osint-alerts

# Pipe findings directly
subfinder -d example.com -silent \
  | dnsx -silent \
  | notify -id osint-alerts

# Send nuclei findings as they are discovered
nuclei -l urls.txt -tags critical -silent \
  | notify -id osint-alerts -bulk
```

### 2.3 Continuous Subdomain Monitoring Script

```bash
#!/bin/bash
# monitors_subdomains.sh
# Run via cron: 0 6 * * * /path/to/monitor_subdomains.sh
# Alerts on NEW subdomains discovered since last run

TARGET=$1
DATE=$(date +%Y%m%d)
YESTERDAY=$(date -d "yesterday" +%Y%m%d 2>/dev/null || date -v-1d +%Y%m%d)
DATA_DIR="$HOME/osint_monitor/$TARGET"
mkdir -p $DATA_DIR

echo "[$(date)] Starting subdomain monitor for $TARGET"

# Collect subdomains from multiple sources
subfinder -d $TARGET -silent -all > $DATA_DIR/current_subfinder.txt
amass enum -passive -d $TARGET -silent > $DATA_DIR/current_amass.txt
curl -s "https://crt.sh/?q=%.$TARGET&output=json" \
  | python3 -c "
import sys, json
try:
    data = json.load(sys.stdin)
    names = set()
    for c in data:
        for n in c['name_value'].split('\n'):
            n = n.replace('*.','').strip()
            if n and '$TARGET' in n: names.add(n)
    print('\n'.join(sorted(names)))
except: pass" > $DATA_DIR/current_crt.txt

# Merge all current results
cat $DATA_DIR/current_*.txt | sort -u > $DATA_DIR/current_all.txt
echo "[*] Total subdomains today: $(wc -l < $DATA_DIR/current_all.txt)"

# Compare with yesterday's results
if [ -f "$DATA_DIR/previous_all.txt" ]; then
    NEW_SUBS=$(comm -13 $DATA_DIR/previous_all.txt $DATA_DIR/current_all.txt)
    REMOVED_SUBS=$(comm -23 $DATA_DIR/previous_all.txt $DATA_DIR/current_all.txt)

    if [ -n "$NEW_SUBS" ]; then
        echo "[!] NEW SUBDOMAINS FOUND for $TARGET:"
        echo "$NEW_SUBS"

        # Probe new subdomains for web services
        echo "$NEW_SUBS" | dnsx -silent | httpx -silent -status-code -title \
          | tee $DATA_DIR/new_sub_details.txt

        # Send alert
        echo "$NEW_SUBS" \
          | sed "s/^/[NEW SUBDOMAIN] $TARGET: /" \
          | notify -id osint-alerts 2>/dev/null

        # Run Nuclei on new subdomains
        echo "$NEW_SUBS" | httpx -silent \
          | nuclei -tags exposure,misconfiguration,default-login \
            -severity critical,high -silent \
          | tee -a $DATA_DIR/nuclei_new.txt \
          | notify -id osint-alerts 2>/dev/null
    else
        echo "[*] No new subdomains found"
    fi

    if [ -n "$REMOVED_SUBS" ]; then
        echo "[*] Removed subdomains:"
        echo "$REMOVED_SUBS"
    fi
else
    echo "[*] First run — establishing baseline"
fi

# Save current as previous for next run
cp $DATA_DIR/current_all.txt $DATA_DIR/previous_all.txt
echo "[$(date)] Monitor complete for $TARGET"
```

### 2.4 New URL Discovery Pipeline

```bash
#!/bin/bash
# url_monitor.sh — finds new URLs from archives daily

TARGET=$1
DATA_DIR="$HOME/osint_monitor/$TARGET"
mkdir -p $DATA_DIR/urls

echo "[*] Collecting URLs for $TARGET..."

# Collect from multiple sources
gau --subs $TARGET > $DATA_DIR/urls/current_gau.txt
waybackurls $TARGET >> $DATA_DIR/urls/current_gau.txt
sort -u $DATA_DIR/urls/current_gau.txt -o $DATA_DIR/urls/current_all.txt

echo "[*] Total URLs: $(wc -l < $DATA_DIR/urls/current_all.txt)"

# Extract interesting categories
grep "?" $DATA_DIR/urls/current_all.txt > $DATA_DIR/urls/parameterized.txt
grep -iE "\.(js|json|xml|env|config|conf|bak|sql|log|txt|yaml|yml)$" \
  $DATA_DIR/urls/current_all.txt > $DATA_DIR/urls/interesting_files.txt

echo "[*] Parameterized URLs: $(wc -l < $DATA_DIR/urls/parameterized.txt)"
echo "[*] Interesting files: $(wc -l < $DATA_DIR/urls/interesting_files.txt)"

# Find NEW URLs since last run
if [ -f "$DATA_DIR/urls/previous_all.txt" ]; then
    NEW_URLS=$(comm -13 \
      <(sort $DATA_DIR/urls/previous_all.txt) \
      <(sort $DATA_DIR/urls/current_all.txt))
    COUNT=$(echo "$NEW_URLS" | wc -l)
    echo "[*] New URLs since last run: $COUNT"

    if [ $COUNT -gt 0 ]; then
        echo "$NEW_URLS" > $DATA_DIR/urls/new_urls.txt
        # Alert on new parameterized URLs
        echo "$NEW_URLS" | grep "?" \
          | sed "s/^/[NEW URL] /" \
          | notify -id osint-alerts 2>/dev/null
    fi
fi

cp $DATA_DIR/urls/current_all.txt $DATA_DIR/urls/previous_all.txt
```

---

## 3. Python Automation

### 3.1 Why Python for OSINT

```
Python strengths for OSINT:
  → Rich library ecosystem (requests, shodan, censys, tweepy, etc.)
  → Async/concurrent requests (aiohttp, asyncio)
  → Data parsing (JSON, HTML, XML, PDF, Excel)
  → API integration
  → Easy file I/O and data processing
  → Cross-platform

Key libraries:
  requests      →  HTTP requests
  aiohttp       →  Async HTTP (faster for bulk requests)
  beautifulsoup4→  HTML parsing
  lxml          →  Fast XML/HTML parsing
  shodan        →  Shodan API
  censys        →  Censys API
  tweepy        →  Twitter API
  instaloader   →  Instagram scraping
  python-whois  →  WHOIS queries
  dnspython     →  DNS queries
  ipaddress     →  IP range calculations
  pandas        →  Data analysis and CSV processing
  Pillow        →  Image processing
  exifread      →  EXIF extraction
```

### 3.2 OSINT Python Template

```python
#!/usr/bin/env python3
"""
OSINT Automation Template
Usage: python3 osint_template.py -d example.com
"""

import argparse
import asyncio
import json
import os
import sys
import time
from datetime import datetime
from pathlib import Path

import requests
import dns.resolver
import whois

# ─── Configuration ────────────────────────────────────────────────

CONFIG = {
    "shodan_key": os.environ.get("SHODAN_API_KEY", ""),
    "securitytrails_key": os.environ.get("SECURITYTRAILS_KEY", ""),
    "virustotal_key": os.environ.get("VIRUSTOTAL_KEY", ""),
    "output_dir": "osint_output",
    "timeout": 10,
    "user_agent": "Mozilla/5.0 (compatible; OSINT-Research/1.0)",
}

# ─── Utility Functions ────────────────────────────────────────────

def save_results(filename, data, output_dir=None):
    """Save results to file."""
    out_dir = Path(output_dir or CONFIG["output_dir"])
    out_dir.mkdir(parents=True, exist_ok=True)
    filepath = out_dir / filename
    if isinstance(data, (dict, list)):
        with open(filepath, "w") as f:
            json.dump(data, f, indent=2)
    else:
        with open(filepath, "w") as f:
            f.write(str(data))
    print(f"[+] Saved: {filepath}")
    return filepath


def log(msg, level="INFO"):
    """Timestamped logging."""
    ts = datetime.now().strftime("%H:%M:%S")
    symbols = {"INFO": "*", "WARN": "!", "ERROR": "X", "FOUND": "+"}
    sym = symbols.get(level, "*")
    print(f"[{ts}] [{sym}] {msg}")


def http_get(url, headers=None, timeout=None):
    """Safe HTTP GET with error handling."""
    try:
        headers = headers or {"User-Agent": CONFIG["user_agent"]}
        r = requests.get(url, headers=headers,
                         timeout=timeout or CONFIG["timeout"])
        return r
    except requests.exceptions.RequestException as e:
        log(f"HTTP error for {url}: {e}", "ERROR")
        return None

# ─── DNS Module ───────────────────────────────────────────────────

def get_dns_records(domain):
    """Extract all DNS records for a domain."""
    records = {}
    record_types = ["A", "AAAA", "MX", "NS", "TXT", "SOA", "CNAME"]

    for rtype in record_types:
        try:
            answers = dns.resolver.resolve(domain, rtype)
            records[rtype] = [str(r) for r in answers]
            log(f"{rtype}: {records[rtype]}", "FOUND")
        except (dns.resolver.NoAnswer, dns.resolver.NXDOMAIN,
                dns.exception.DNSException):
            records[rtype] = []

    return records


def extract_spf_services(spf_record):
    """Parse SPF record to extract email services."""
    services = []
    service_map = {
        "google.com": "Google Workspace",
        "sendgrid.net": "SendGrid",
        "amazonses.com": "Amazon SES",
        "mailchimp.com": "Mailchimp",
        "protection.outlook.com": "Microsoft 365",
        "zoho.com": "Zoho Mail",
        "mailgun.org": "Mailgun",
        "sparkpostmail.com": "SparkPost",
        "hubspotemail.net": "HubSpot",
        "salesforce.com": "Salesforce",
        "mktomail.com": "Marketo",
    }
    for keyword, service in service_map.items():
        if keyword in spf_record:
            services.append(service)
            log(f"Email service detected: {service}", "FOUND")
    return services

# ─── CT Log Module ────────────────────────────────────────────────

def get_ct_subdomains(domain):
    """Query crt.sh for subdomains via CT logs."""
    log(f"Querying CT logs for *.{domain}")
    url = f"https://crt.sh/?q=%.{domain}&output=json"
    r = http_get(url)
    if not r or r.status_code != 200:
        return set()

    names = set()
    try:
        data = r.json()
        for cert in data:
            for name in cert.get("name_value", "").split("\n"):
                name = name.replace("*.", "").strip().lower()
                if name and domain in name:
                    names.add(name)
    except (json.JSONDecodeError, KeyError):
        pass

    log(f"CT logs found {len(names)} subdomains", "FOUND")
    return names

# ─── Shodan Module ────────────────────────────────────────────────

def shodan_search(query, limit=100):
    """Search Shodan and return results."""
    if not CONFIG["shodan_key"]:
        log("Shodan API key not configured", "WARN")
        return []

    import shodan as shodan_api
    api = shodan_api.Shodan(CONFIG["shodan_key"])
    results = []
    try:
        search = api.search(query, limit=limit)
        for match in search.get("matches", []):
            results.append({
                "ip": match.get("ip_str"),
                "port": match.get("port"),
                "org": match.get("org"),
                "product": match.get("product"),
                "version": match.get("version"),
                "hostnames": match.get("hostnames", []),
                "country": match.get("location", {}).get("country_name"),
            })
        log(f"Shodan: {len(results)} results for '{query}'", "FOUND")
    except shodan_api.APIError as e:
        log(f"Shodan error: {e}", "ERROR")
    return results

# ─── WHOIS Module ─────────────────────────────────────────────────

def get_whois(domain):
    """Get WHOIS information for a domain."""
    try:
        w = whois.whois(domain)
        return {
            "registrar": str(w.registrar),
            "org": str(w.org),
            "email": str(w.emails),
            "creation_date": str(w.creation_date),
            "expiration_date": str(w.expiration_date),
            "name_servers": [str(ns) for ns in (w.name_servers or [])],
        }
    except Exception as e:
        log(f"WHOIS error: {e}", "ERROR")
        return {}

# ─── Main Orchestration ───────────────────────────────────────────

def run_osint(domain):
    """Full OSINT pipeline for a domain."""
    log(f"Starting OSINT for: {domain}")
    results = {"domain": domain, "timestamp": datetime.now().isoformat()}

    # 1. WHOIS
    log("Phase 1: WHOIS")
    results["whois"] = get_whois(domain)

    # 2. DNS Records
    log("Phase 2: DNS Records")
    results["dns"] = get_dns_records(domain)

    # 3. SPF Analysis
    spf_records = [r for r in results["dns"].get("TXT", []) if "v=spf1" in r]
    if spf_records:
        results["email_services"] = extract_spf_services(spf_records[0])

    # 4. CT Subdomains
    log("Phase 3: Certificate Transparency")
    results["ct_subdomains"] = list(get_ct_subdomains(domain))

    # 5. Shodan
    log("Phase 4: Shodan")
    results["shodan"] = shodan_search(f'ssl.cert.subject.cn:"{domain}"')

    # Save results
    outfile = save_results(f"{domain}_osint.json", results)
    log(f"OSINT complete. Results: {outfile}")
    return results

# ─── CLI Entry Point ──────────────────────────────────────────────

if __name__ == "__main__":
    parser = argparse.ArgumentParser(description="OSINT Automation")
    parser.add_argument("-d", "--domain", required=True, help="Target domain")
    parser.add_argument("-o", "--output", help="Output directory")
    args = parser.parse_args()

    if args.output:
        CONFIG["output_dir"] = args.output

    results = run_osint(args.domain)

    # Summary
    print("\n=== SUMMARY ===")
    print(f"Domain:      {results['domain']}")
    print(f"Registrar:   {results['whois'].get('registrar', 'N/A')}")
    print(f"Subdomains:  {len(results.get('ct_subdomains', []))}")
    print(f"Shodan hits: {len(results.get('shodan', []))}")
    print(f"Email svcs:  {', '.join(results.get('email_services', []))}")
```

### 3.3 Async HTTP Requests for Bulk Operations

```python
#!/usr/bin/env python3
"""
Async subdomain prober — check hundreds of URLs concurrently
"""
import asyncio
import aiohttp
from typing import List, Dict

async def probe_url(session: aiohttp.ClientSession,
                    url: str,
                    semaphore: asyncio.Semaphore) -> Dict:
    """Probe a single URL asynchronously."""
    async with semaphore:
        try:
            async with session.get(url, timeout=aiohttp.ClientTimeout(total=10),
                                   allow_redirects=True,
                                   ssl=False) as resp:
                title = ""
                if "text/html" in resp.headers.get("content-type", ""):
                    text = await resp.text(errors="ignore")
                    # Extract title
                    import re
                    match = re.search(r"<title[^>]*>(.*?)</title>",
                                     text, re.IGNORECASE | re.DOTALL)
                    if match:
                        title = match.group(1).strip()[:100]

                return {
                    "url": url,
                    "status": resp.status,
                    "title": title,
                    "server": resp.headers.get("server", ""),
                    "content_length": resp.headers.get("content-length", ""),
                }
        except Exception:
            return {"url": url, "status": 0, "error": True}


async def probe_all(urls: List[str], concurrency: int = 50) -> List[Dict]:
    """Probe multiple URLs concurrently."""
    semaphore = asyncio.Semaphore(concurrency)
    connector = aiohttp.TCPConnector(ssl=False, limit=concurrency)

    async with aiohttp.ClientSession(connector=connector) as session:
        tasks = [probe_url(session, url, semaphore) for url in urls]
        results = await asyncio.gather(*tasks, return_exceptions=False)

    return [r for r in results if not r.get("error")]


def probe_urls(urls: List[str], concurrency: int = 50) -> List[Dict]:
    """Synchronous wrapper for async probing."""
    return asyncio.run(probe_all(urls, concurrency))


# Usage
if __name__ == "__main__":
    with open("subdomains.txt") as f:
        domains = [line.strip() for line in f if line.strip()]

    # Add http and https variants
    urls = []
    for d in domains:
        if not d.startswith("http"):
            urls.extend([f"https://{d}", f"http://{d}"])
        else:
            urls.append(d)

    print(f"[*] Probing {len(urls)} URLs with concurrency=50")
    results = probe_urls(urls)

    # Filter and display interesting results
    for r in sorted(results, key=lambda x: x.get("status", 0)):
        if r["status"] in [200, 301, 302, 403, 401]:
            print(f"[{r['status']}] {r['url']:60} {r.get('title',''):50} {r.get('server','')}")
```

---

## 4. recon-ng

### 4.1 What recon-ng Is

recon-ng is a full-featured web reconnaissance framework built in Python.
It uses a module-based architecture similar to Metasploit — each module
performs a specific OSINT task.

```
Architecture:
  Workspaces → separate investigations
  Modules    → individual OSINT tasks (querying APIs, scraping, etc.)
  Database   → stores all findings (hosts, contacts, credentials, etc.)
  Reports    → output in various formats
```

### 4.2 Installation & Setup

```bash
# Install
pip3 install recon-ng
# or
git clone https://github.com/lanmaster53/recon-ng.git
cd recon-ng && pip3 install -r REQUIREMENTS

# Start
recon-ng
```

### 4.3 Core Commands

```
# Inside recon-ng shell:

# Workspace management
workspaces create example_investigation
workspaces load example_investigation
workspaces list

# Module management
marketplace search                    # List all available modules
marketplace search domain             # Search modules by keyword
marketplace search subdomain          # Subdomain-related modules
marketplace install all               # Install all modules
marketplace install recon/domains-hosts/hackertarget  # Install specific module

# Module usage
modules load recon/domains-hosts/hackertarget
info                                  # Show module info + required keys
options set SOURCE example.com        # Set the target
run                                   # Execute the module

# View results stored in database
show hosts                            # All discovered hosts
show contacts                         # All discovered emails/contacts
show credentials                      # Any found credentials
show leaks                            # Data leaks
show vulnerabilities                  # Discovered vulnerabilities

# Manage API keys
keys list                             # Show configured API keys
keys add shodan_api YOURKEYHERE       # Add an API key

# Reports
modules load reporting/html
options set FILENAME /tmp/report.html
run
```

### 4.4 Most Useful recon-ng Modules

```
Category: recon/domains-hosts/
  hackertarget            → Subdomain discovery via HackerTarget API
  brute_hosts             → DNS brute-force
  certificate_transparency → CT log subdomain discovery
  bing_domain_web         → Bing subdomain discovery
  google_site_web         → Google site: subdomain discovery
  shodan_hostname         → Shodan hostname search
  threatcrowd             → ThreatCrowd subdomain discovery
  virustotal              → VirusTotal subdomain discovery
  netcraft                → Netcraft subdomain discovery

Category: recon/hosts-hosts/
  bing_ip                 → Find domains hosted on same IP (Bing ip:)
  ipinfodb                → IP geolocation
  ssltools                → SSL certificate info

Category: recon/domains-contacts/
  whois_pocs              → Extract contacts from WHOIS
  pgp_search              → Search PGP keyservers for email addresses
  hunter_io               → Email discovery via Hunter.io
  linkedin_auth           → LinkedIn contact discovery
  metacrawler_emails      → Email harvesting from metacrawler

Category: recon/contacts-credentials/
  hibp                    → Have I Been Pwned breach check
  hashes_org              → Hash lookup

Category: recon/companies-contacts/
  linkedin_auth           → Employee discovery via LinkedIn

Category: recon/netblocks-hosts/
  shodan_net              → Shodan search by IP range
  reverse_resolve         → Reverse DNS on IP range

Category: reporting/
  html                    → HTML report
  json                    → JSON export
  csv                     → CSV export
  xml                     → XML export
  list                    → Simple text list
```

### 4.5 Full recon-ng Workflow

```
# Start a new investigation
workspaces create targetcorp

# Configure API keys
keys add shodan_api YOUR_SHODAN_KEY
keys add hunter_io YOUR_HUNTER_KEY
keys add virustotal_api YOUR_VT_KEY
keys add github_api YOUR_GITHUB_TOKEN

# Phase 1: Subdomain discovery
modules load recon/domains-hosts/certificate_transparency
options set SOURCE example.com
run

modules load recon/domains-hosts/hackertarget
options set SOURCE example.com
run

modules load recon/domains-hosts/virustotal
options set SOURCE example.com
run

# Phase 2: IPs from hostnames
modules load recon/hosts-hosts/resolve
run    # Resolves all discovered hostnames to IPs

# Phase 3: Reverse DNS on found IPs
modules load recon/hosts-hosts/reverse_resolve
run

# Phase 4: Shodan enrichment
modules load recon/hosts-hosts/shodan_hostname
run

# Phase 5: Contact discovery
modules load recon/domains-contacts/hunter_io
options set SOURCE example.com
run

# Phase 6: View all results
show hosts
show contacts

# Phase 7: Generate report
modules load reporting/html
options set FILENAME /tmp/targetcorp_report.html
options set CREATOR "Your Name"
options set CUSTOMER "Target Corp"
run
```

---

## 5. SpiderFoot

### 5.1 What SpiderFoot Is

SpiderFoot is an automated OSINT platform with 200+ modules covering
everything from DNS to social media to dark web to breach data.
It has a web UI, a CLI, and an API.

```bash
# Install
pip3 install spiderfoot

# Start web UI
spiderfoot -l 127.0.0.1:5001

# Open browser: http://127.0.0.1:5001
```

### 5.2 SpiderFoot Web UI Workflow

```
1. Open http://127.0.0.1:5001
2. Click "New Scan"
3. Enter:
   - Scan Name: "Target Corp Investigation"
   - Scan Target: example.com (or IP, or name, or email, or phone)
   - Target Type: Domain/IPv4/IPv6/Person/Username/Email/Phone

4. Choose scan type:
   - "All" → runs all 200+ modules (slow but thorough)
   - "Investigate" → balanced coverage
   - "Footprint" → infrastructure focused
   - "Passive" → passive sources only (no direct contact with target)

5. Configure API keys (Settings → API Keys):
   Shodan, VirusTotal, Censys, SecurityTrails, HaveIBeenPwned,
   Hunter.io, LinkedIn, GitHub, etc.

6. Run scan → watch results populate in real time

7. Results view:
   → Browse: by data type (IPs, emails, subdomains, etc.)
   → Graph: visual entity relationship map
   → Export: CSV, JSON, GEXF (for Gephi visualization)
```

### 5.3 SpiderFoot CLI

```bash
# Scan a domain (passive only)
spiderfoot -s example.com -m sfp_dns,sfp_subdomain_brute,sfp_crt \
  -o OUTPUT_FILE.json -q

# Scan with all passive modules
spiderfoot -s example.com -t INTERNET_NAME \
  --modules-passive -o results.json

# Scan with specific modules
spiderfoot -s example.com \
  -m sfp_shodan,sfp_censys,sfp_virustotal,sfp_crtsh \
  -o results.json -q

# Full scan (aggressive — all modules)
spiderfoot -s example.com -t INTERNET_NAME -o results.json

# Available target types:
#  INTERNET_NAME, IP_ADDRESS, IPV6_ADDRESS,
#  EMAILADDR, PHONE_NUMBER, HUMAN_NAME, USERNAME
```

### 5.4 SpiderFoot HX (Hosted Version)

```
URL: spiderfoot.net/hx/

Cloud-hosted version — no installation needed.
Free tier: limited scans per month.
Paid: full access.

Advantage: pre-configured API keys for many sources.
```

---

## 6. Maltego

### 6.1 What Maltego Is

Maltego is a visual link analysis and OSINT platform. It represents
entities (persons, domains, IPs, emails, companies) as nodes in a graph
and connects them via "transforms" — automated queries that find related entities.

```bash
# Download: maltego.com (Community Edition is free)
# Requires Java installation

# Versions:
# Community Edition (CE): free; limited transforms (12 results per transform)
# Professional:           paid; unlimited
# Enterprise:             paid; team collaboration
```

### 6.2 Core Concepts

```
Entity:     A data point — domain, IP, email, person, company, phone, etc.
Transform:  A query that takes one entity and finds related entities
            e.g., "Find subdomains" transform on a domain entity
Machine:    A sequence of transforms run automatically
Graph:      The visual canvas showing all entities and their relationships

Workflow:
  1. Add a starting entity (e.g., domain: example.com)
  2. Right-click → Run transform → choose what to look for
  3. New entities appear on the canvas connected to the original
  4. Right-click any new entity → run more transforms
  5. Build a web of relationships
```

### 6.3 Key Transforms

```
On a Domain entity:
  → To DNS Name (subdomains)
  → To Email Address (from WHOIS)
  → To IP Address
  → To MX record
  → To NS record
  → To Website
  → Shodan: devices for this domain
  → VirusTotal: info on this domain

On an IP entity:
  → To DNS Name (reverse DNS)
  → To Domain (what domains are hosted here)
  → Shodan: services on this IP
  → To ASN / netblock
  → To Location (geolocation)

On an Email entity:
  → To Person (from associated accounts)
  → To Phone (from breach data)
  → Have I Been Pwned: breaches
  → To Domain (extract domain part)
  → PGP search

On a Person entity:
  → To Email Address
  → To Phone
  → To Social Media accounts
  → To Companies (from LinkedIn)

Transform Hubs (available in Maltego):
  VirusTotal Transform Hub → IP/domain reputation
  Shodan Transform Hub     → device and service discovery
  HaveIBeenPwned           → breach data
  Hunter.io                → email discovery
  IPAPI                    → IP geolocation
  PassiveTotal/RiskIQ      → threat intelligence
  Censys                   → certificate and host data
```

### 6.4 Maltego Machines

```
Machines automate sequences of transforms for common investigations.

Built-in machines:
  Company Stalker:   starts with a domain → finds emails + social profiles
  Person Stalker:    starts with email → finds social profiles + phone
  Footprint L1:      light infrastructure footprint
  Footprint L2:      medium footprint
  Footprint L3:      deep footprint (aggressive)

Running a machine:
  1. Add starting entity to canvas
  2. Machines → Run Machine → select machine
  3. Watch graph build automatically
  4. Review and pivot manually from interesting nodes
```

---

## 7. GitHub Actions for OSINT Automation

### 7.1 Using GitHub Actions as a Free OSINT Scheduler

GitHub Actions runs workflows on a schedule — free for public repositories.
Use it to run your OSINT pipelines daily and receive alerts.

```yaml
# .github/workflows/osint_monitor.yml

name: Daily OSINT Monitor
on:
  schedule:
    - cron: '0 1 * * *'    # Run at 01:00 UTC every day
  workflow_dispatch:        # Allow manual trigger

jobs:
  subdomain_monitor:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repo
        uses: actions/checkout@v3

      - name: Install Go tools
        run: |
          go install github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest
          go install github.com/projectdiscovery/dnsx/cmd/dnsx@latest
          go install github.com/projectdiscovery/httpx/cmd/httpx@latest
          go install github.com/projectdiscovery/nuclei/v3/cmd/nuclei@latest
          go install github.com/projectdiscovery/notify/cmd/notify@latest

      - name: Run subdomain enumeration
        env:
          TARGET: ${{ secrets.TARGET_DOMAIN }}
        run: |
          subfinder -d $TARGET -silent | dnsx -silent | \
            httpx -silent -status-code -title > current_results.txt
          cat current_results.txt

      - name: Compare with previous results
        run: |
          if [ -f previous_results.txt ]; then
            diff previous_results.txt current_results.txt > diff_output.txt || true
            NEW=$(grep "^>" diff_output.txt | wc -l)
            echo "New findings: $NEW"
            if [ $NEW -gt 0 ]; then
              grep "^>" diff_output.txt | sed 's/^> //' > new_findings.txt
            fi
          else
            echo "First run - establishing baseline"
            cp current_results.txt previous_results.txt
          fi

      - name: Send Discord alert if new findings
        if: ${{ hashFiles('new_findings.txt') != '' }}
        env:
          DISCORD_WEBHOOK: ${{ secrets.DISCORD_WEBHOOK_URL }}
        run: |
          FINDINGS=$(cat new_findings.txt | head -20)
          curl -X POST $DISCORD_WEBHOOK \
            -H "Content-Type: application/json" \
            -d "{\"content\": \"**New OSINT Findings**\n\`\`\`$FINDINGS\`\`\`\"}"

      - name: Commit updated results
        run: |
          cp current_results.txt previous_results.txt
          git config user.email "osint-bot@github.com"
          git config user.name "OSINT Bot"
          git add previous_results.txt
          git commit -m "Update OSINT baseline $(date +%Y-%m-%d)" || true
          git push || true
```

### 7.2 Secrets Configuration

```
In GitHub repository → Settings → Secrets and variables → Actions:
  TARGET_DOMAIN    = example.com
  DISCORD_WEBHOOK_URL = https://discord.com/api/webhooks/...
  SHODAN_API_KEY   = your_key
  NOTIFY_CONFIG    = (base64-encoded notify config)
```

---

## 8. Building a Personal OSINT Dashboard

### 8.1 Storing Results in SQLite

```python
#!/usr/bin/env python3
"""
OSINT Result Database — store and query all findings
"""
import sqlite3
from datetime import datetime
from pathlib import Path

class OSINTDatabase:
    def __init__(self, db_path="osint_findings.db"):
        self.conn = sqlite3.connect(db_path)
        self.conn.row_factory = sqlite3.Row
        self._init_tables()

    def _init_tables(self):
        """Create tables if they don't exist."""
        self.conn.executescript("""
            CREATE TABLE IF NOT EXISTS subdomains (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                domain TEXT,
                subdomain TEXT UNIQUE,
                status_code INTEGER,
                title TEXT,
                server TEXT,
                tech TEXT,
                first_seen TEXT,
                last_seen TEXT
            );

            CREATE TABLE IF NOT EXISTS ports (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                ip TEXT,
                port INTEGER,
                service TEXT,
                version TEXT,
                first_seen TEXT,
                last_seen TEXT,
                UNIQUE(ip, port)
            );

            CREATE TABLE IF NOT EXISTS findings (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                target TEXT,
                type TEXT,
                severity TEXT,
                title TEXT,
                description TEXT,
                url TEXT,
                tool TEXT,
                timestamp TEXT
            );

            CREATE TABLE IF NOT EXISTS emails (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                domain TEXT,
                email TEXT UNIQUE,
                source TEXT,
                timestamp TEXT
            );
        """)
        self.conn.commit()

    def add_subdomain(self, domain, subdomain, status=None,
                      title=None, server=None, tech=None):
        now = datetime.now().isoformat()
        try:
            self.conn.execute("""
                INSERT INTO subdomains
                  (domain, subdomain, status_code, title, server, tech,
                   first_seen, last_seen)
                VALUES (?, ?, ?, ?, ?, ?, ?, ?)
                ON CONFLICT(subdomain) DO UPDATE SET
                  status_code=excluded.status_code,
                  title=excluded.title,
                  last_seen=excluded.last_seen
            """, (domain, subdomain, status, title, server, tech, now, now))
            self.conn.commit()
        except sqlite3.Error as e:
            print(f"DB error: {e}")

    def add_finding(self, target, finding_type, severity,
                    title, description, url="", tool=""):
        now = datetime.now().isoformat()
        self.conn.execute("""
            INSERT INTO findings
              (target, type, severity, title, description, url, tool, timestamp)
            VALUES (?, ?, ?, ?, ?, ?, ?, ?)
        """, (target, finding_type, severity, title, description, url, tool, now))
        self.conn.commit()

    def get_new_subdomains(self, domain, since_hours=24):
        """Find subdomains added in the last N hours."""
        cursor = self.conn.execute("""
            SELECT subdomain, status_code, title
            FROM subdomains
            WHERE domain = ?
              AND first_seen > datetime('now', '-' || ? || ' hours')
            ORDER BY first_seen DESC
        """, (domain, since_hours))
        return cursor.fetchall()

    def get_critical_findings(self):
        """Get all critical and high findings."""
        cursor = self.conn.execute("""
            SELECT * FROM findings
            WHERE severity IN ('critical', 'high')
            ORDER BY timestamp DESC
        """)
        return cursor.fetchall()

    def summary(self, domain):
        """Print a summary for a domain."""
        total = self.conn.execute(
            "SELECT COUNT(*) FROM subdomains WHERE domain=?", (domain,)
        ).fetchone()[0]
        new = len(self.get_new_subdomains(domain))
        findings = self.conn.execute(
            "SELECT severity, COUNT(*) FROM findings WHERE target=? GROUP BY severity",
            (domain,)
        ).fetchall()
        print(f"\n=== {domain} ===")
        print(f"Total subdomains: {total} ({new} new in 24h)")
        for row in findings:
            print(f"  {row[0]}: {row[1]} findings")
```

---

## 9. Tool Chaining Reference

### 9.1 The ProjectDiscovery Stack

ProjectDiscovery makes a suite of complementary tools that are
designed to chain together. This is the most common toolkit in
modern bug bounty automation.

```bash
# Complete PD stack installation
go install github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest
go install github.com/projectdiscovery/amass/v4/...@master
go install github.com/projectdiscovery/dnsx/cmd/dnsx@latest
go install github.com/projectdiscovery/naabu/v2/cmd/naabu@latest
go install github.com/projectdiscovery/httpx/cmd/httpx@latest
go install github.com/projectdiscovery/katana/cmd/katana@latest
go install github.com/projectdiscovery/nuclei/v3/cmd/nuclei@latest
go install github.com/projectdiscovery/notify/cmd/notify@latest
go install github.com/projectdiscovery/interactsh/cmd/interactsh-client@latest
go install github.com/projectdiscovery/uncover/cmd/uncover@latest

# Update all nuclei templates
nuclei -update-templates
```

**uncover — unified search across Shodan/Censys/FOFA/etc.:**
```bash
# Search across multiple engines simultaneously
echo 'ssl.cert.subject.cn:"example.com"' | uncover -e shodan,censys,fofa

# Or directly
uncover -q 'http.title:"Grafana" org:"Target Corp"' -e shodan,censys

# Useful for getting IPs fast
uncover -q 'org:"Target Corp" port:8443' -f ip -silent
```

### 9.2 The Bug Bounty Recon Pipeline

The most widely used community recon pipeline for bug bounty:

```bash
#!/bin/bash
# bug_bounty_recon.sh — full pipeline for a bug bounty target
# Usage: ./bug_bounty_recon.sh example.com

TARGET=$1
DIR="recon_$(date +%Y%m%d)_$TARGET"
mkdir -p $DIR

# Step 1: Passive subdomain enum
echo "[1] Subdomain enumeration..."
subfinder -d $TARGET -silent -all > $DIR/subs_sf.txt
amass enum -passive -d $TARGET -silent > $DIR/subs_amass.txt
curl -s "https://crt.sh/?q=%.$TARGET&output=json" \
  | python3 -c "
import sys,json
d=json.load(sys.stdin)
s=set()
for c in d:
    for n in c['name_value'].split('\n'):
        n=n.replace('*.','').strip()
        if '$TARGET' in n: s.add(n)
print('\n'.join(s))" > $DIR/subs_crt.txt
cat $DIR/subs_*.txt | sort -u > $DIR/all_subs.txt
echo "[*] Total: $(wc -l < $DIR/all_subs.txt) subdomains"

# Step 2: DNS resolution
echo "[2] Resolving live subdomains..."
cat $DIR/all_subs.txt | dnsx -silent > $DIR/live_subs.txt
echo "[*] Live: $(wc -l < $DIR/live_subs.txt)"

# Step 3: Port scan
echo "[3] Port scanning..."
cat $DIR/live_subs.txt | naabu -top-ports 1000 -silent > $DIR/open_ports.txt

# Step 4: Web probe
echo "[4] Web probing..."
cat $DIR/live_subs.txt | httpx -silent -status-code -title -tech-detect \
  -web-server -o $DIR/web_hosts.txt

# Step 5: URL collection
echo "[5] Collecting URLs..."
cat $DIR/live_subs.txt | gau --threads 5 > $DIR/urls.txt
cat $DIR/live_subs.txt | waybackurls >> $DIR/urls.txt
sort -u $DIR/urls.txt -o $DIR/urls.txt
echo "[*] URLs: $(wc -l < $DIR/urls.txt)"

# Step 6: Nuclei scan
echo "[6] Nuclei scanning..."
cat $DIR/web_hosts.txt | awk '{print $1}' | \
  nuclei -tags exposure,misconfiguration,default-login,takeover \
  -severity critical,high -silent \
  -o $DIR/nuclei_findings.txt

# Step 7: JavaScript crawling
echo "[7] JS crawling..."
cat $DIR/web_hosts.txt | awk '{print $1}' | \
  katana -js-crawl -silent > $DIR/js_endpoints.txt

# Step 8: Interesting findings
echo "[8] Flagging interesting hosts..."
grep -iE "jenkins|gitlab|grafana|kibana|admin|dashboard|internal|staging|dev|api" \
  $DIR/web_hosts.txt > $DIR/interesting.txt

# Summary
echo ""
echo "=== RECON COMPLETE: $TARGET ==="
echo "Subdomains:    $(wc -l < $DIR/all_subs.txt)"
echo "Live DNS:      $(wc -l < $DIR/live_subs.txt)"
echo "Web hosts:     $(wc -l < $DIR/web_hosts.txt)"
echo "URLs:          $(wc -l < $DIR/urls.txt)"
echo "Nuclei hits:   $(wc -l < $DIR/nuclei_findings.txt 2>/dev/null || echo 0)"
echo "Interesting:   $(wc -l < $DIR/interesting.txt)"
```

---

## 10. Reporting Automation

### 10.1 Generating Markdown Reports from JSON

```python
#!/usr/bin/env python3
"""
Auto-generate a Markdown report from OSINT JSON data
"""
import json
from datetime import datetime

def generate_report(data: dict, output_file: str = "report.md"):
    domain = data.get("domain", "Unknown")
    ts = data.get("timestamp", datetime.now().isoformat())

    lines = [
        f"# OSINT Report: {domain}",
        f"**Generated:** {ts}",
        "",
        "---",
        "",
        "## 1. Domain Overview",
        "",
        "| Field | Value |",
        "|---|---|",
    ]

    # WHOIS section
    whois = data.get("whois", {})
    for key, value in whois.items():
        lines.append(f"| {key.title()} | {value} |")
    lines.append("")

    # DNS Records
    lines.extend([
        "## 2. DNS Records",
        "",
    ])
    dns = data.get("dns", {})
    for rtype, records in dns.items():
        if records:
            lines.append(f"**{rtype}:**")
            for r in records:
                lines.append(f"  - `{r}`")
    lines.append("")

    # Email services
    email_svcs = data.get("email_services", [])
    if email_svcs:
        lines.extend([
            "## 3. Email Services Detected",
            "",
        ])
        for svc in email_svcs:
            lines.append(f"- {svc}")
        lines.append("")

    # Subdomains
    subs = data.get("ct_subdomains", [])
    lines.extend([
        f"## 4. Subdomains ({len(subs)} found)",
        "",
        "| Subdomain |",
        "|---|",
    ])
    for sub in sorted(subs)[:50]:  # First 50
        lines.append(f"| `{sub}` |")
    if len(subs) > 50:
        lines.append(f"| ... and {len(subs)-50} more |")
    lines.append("")

    # Shodan findings
    shodan = data.get("shodan", [])
    if shodan:
        lines.extend([
            f"## 5. Exposed Services ({len(shodan)} found via Shodan)",
            "",
            "| IP | Port | Product | Version | Country |",
            "|---|---|---|---|---|",
        ])
        for s in shodan[:20]:
            lines.append(
                f"| {s.get('ip','')} | {s.get('port','')} | "
                f"{s.get('product','')} | {s.get('version','')} | "
                f"{s.get('country','')} |"
            )
    lines.append("")

    with open(output_file, "w") as f:
        f.write("\n".join(lines))
    print(f"[+] Report saved: {output_file}")


if __name__ == "__main__":
    import sys
    with open(sys.argv[1]) as f:
        data = json.load(f)
    generate_report(data, sys.argv[2] if len(sys.argv) > 2 else "report.md")
```

---

## 11. Level 9 Practice Tasks

### Task 1 — Build the Monitoring Script
- Set up the subdomain monitoring script from Section 2.3 on your own domain
  OR on a domain from a bug bounty program that allows passive monitoring
- Run it manually once to establish a baseline
- Schedule it via cron to run daily
- Set up a Discord webhook for alerts (free: discord.com → server settings → integrations → webhooks)

### Task 2 — recon-ng Full Investigation
- Install recon-ng and complete the full workflow from Section 4.5
- Target: any large organization with a public bug bounty program
- Configure at least 3 API keys (Shodan, VirusTotal, Hunter.io are all free)
- Generate an HTML report and review the output

### Task 3 — SpiderFoot Passive Scan
- Install SpiderFoot and run the web UI
- Run a "Passive" scan on a domain of your choice
- Review the entity graph — identify 3 data points that surprised you
- Export results as CSV and analyze in a spreadsheet

### Task 4 — Python OSINT Script
- Take the template from Section 3.2
- Extend it with one additional module of your choice:
  - SecurityTrails API subdomain lookup
  - VirusTotal domain report
  - Email format detection via Hunter.io
  - Breach check via HIBP API
- Test it on a domain and review the JSON output

### Task 5 — GitHub Actions Pipeline
- Fork a public GitHub repository
- Create the GitHub Actions workflow from Section 7.1
- Configure repository secrets (TARGET_DOMAIN, DISCORD_WEBHOOK_URL)
- Trigger it manually (workflow_dispatch)
- Review the action logs and confirm it ran successfully

### Task 6 — Bug Bounty Pipeline
- Find a bug bounty program with a wildcard scope (*.example.com)
  → HackerOne: hackerone.com/programs → filter by "wildcard" in scope
- Run the complete pipeline from Section 9.2 on the target
- Document: total subdomains, interesting hosts, any nuclei findings
- Write a brief report summarizing your findings

---

## 12. Level 9 Summary

```
✅ Automation converts one-time recon into continuous intelligence
✅ notify tool → pipe any tool output to Discord/Slack/Telegram
✅ Monitoring scripts compare daily results → alert only on changes
✅ Python async (aiohttp) → probe hundreds of URLs concurrently
✅ recon-ng → modular framework; workspaces; database; API-driven modules
✅ SpiderFoot → 200+ modules; web UI; passive + active modes; entity graph
✅ Maltego → visual link analysis; entities + transforms; graph-based pivoting
✅ GitHub Actions → free scheduled OSINT runner + alert integration
✅ uncover → search Shodan/Censys/FOFA simultaneously with one query
✅ The ProjectDiscovery stack: subfinder→dnsx→httpx→nuclei→notify
✅ SQLite database → store all findings; query for new/changed assets
✅ Markdown report generation → from JSON data to shareable document
✅ Bug bounty pipeline → single script from target domain to findings

When you are done here → Proceed to Level 10: Advanced OSINT Techniques
```

---

## 13. Tools Quick Reference

| Tool | Purpose | Install | Key Command |
|---|---|---|---|
| **notify** | Alert to Discord/Slack/Telegram | `go install` | `echo "msg" \| notify -id myalert` |
| **recon-ng** | Modular OSINT framework | `pip3 install recon-ng` | `recon-ng` |
| **SpiderFoot** | Automated OSINT (200+ modules) | `pip3 install spiderfoot` | `spiderfoot -l 127.0.0.1:5001` |
| **Maltego** | Visual link analysis | Download | maltego.com |
| **uncover** | Multi-engine search (Shodan+Censys+FOFA) | `go install` | `uncover -q 'org:"Target"' -e shodan,censys` |
| **katana** | JS crawler + endpoint discovery | `go install` | `katana -u https://example.com -js-crawl` |
| **aiohttp** | Async HTTP for Python | `pip3 install aiohttp` | (library) |
| **shodan** (py) | Shodan Python API | `pip3 install shodan` | `import shodan` |
| **dnspython** | DNS queries in Python | `pip3 install dnspython` | `import dns.resolver` |
| **requests** | HTTP in Python | `pip3 install requests` | `import requests` |
| **GitHub Actions** | Free OSINT scheduler | github.com | YAML workflow file |
| **SQLite** | Local OSINT database | built-in Python | `import sqlite3` |
