# OSINT — Complete Learning Map 

```
OSINT (Open Source Intelligence)
│
│   The art of collecting, analyzing, and correlating publicly available
│   information to build intelligence about a target — person, organization,
│   or infrastructure — without any unauthorized access.
│
├── LEVEL 1 — FOUNDATIONS
│   │
│   ├── What is OSINT?
│   │   ├── Definition: Intelligence gathered from public sources only
│   │   ├── Legal basis: All sources are publicly accessible
│   │   ├── Use cases: Pentesting, bug bounty, journalism, law enforcement, HR
│   │   └── OSINT vs. Hacking: You gather data, you do not exploit systems
│   │
│   ├── OSINT Mindset
│   │   ├── Think like an investigator, not a tool runner
│   │   ├── Every data point leads to another — pivot constantly
│   │   ├── Verify everything from at least two independent sources
│   │   ├── Document everything: screenshot + URL + timestamp + tool used
│   │   └── Separate collection from analysis — gather first, conclude later
│   │
│   ├── OSINT Framework (osintframework.com)
│   │   ├── A visual map of all OSINT tools organized by category
│   │   ├── Categories: Username, Email, Domain, IP, Social, Images, etc.
│   │   └── Use it as a reference — not every tool will apply to every case
│   │
│   ├── Legal & Ethical Rules
│   │   ├── Querying public data = legal everywhere
│   │   ├── Accessing private accounts or systems = illegal (even if found via OSINT)
│   │   ├── Using found credentials to log in = unauthorized access
│   │   ├── India: IT Act 2000, Section 66 (unauthorized access)
│   │   ├── US: CFAA (Computer Fraud and Abuse Act)
│   │   ├── EU: GDPR limits on collecting personal data
│   │   └── Bug bounty: Only test assets explicitly listed in scope
│   │
│   └── Operational Security (OpSec) for OSINT
│       ├── Use a dedicated browser (Firefox) with no personal logins
│       ├── Use a VPN or Tor to avoid revealing your IP to target servers
│       ├── Create sock puppet accounts (fake research accounts) where needed
│       ├── Never log into personal accounts during an investigation
│       ├── Use a VM (Virtual Machine) — isolate your research environment
│       └── Clear cookies and metadata before switching targets
│
│
├── LEVEL 2 — PERSON OSINT (Individual Investigations)
│   │
│   ├── 2.1 Username Enumeration
│   │   ├── Concept: One username often used across many platforms
│   │   ├── Sherlock — searches 300+ sites for a username
│   │   │   └── python3 sherlock.py <username>
│   │   ├── WhatsMyName — web-based username lookup (whatsmyname.app)
│   │   ├── Namechk (namechk.com) — checks username on social + domain
│   │   ├── KnowEm (knowem.com) — checks 500+ social networks
│   │   └── Maigret — advanced Sherlock fork with more detail
│   │       └── pip install maigret && maigret <username>
│   │
│   ├── 2.2 Email Address Investigation
│   │   ├── Email format discovery
│   │   │   ├── Hunter.io — find email format of an org (firstname.lastname@)
│   │   │   ├── Email Format (email-format.com) — patterns per company
│   │   │   └── Phonebook.cz — email search engine
│   │   │
│   │   ├── Email validation (is it real?)
│   │   │   ├── Hunter.io email verifier
│   │   │   ├── verify-email.org
│   │   │   └── NeverBounce
│   │   │
│   │   ├── Breach / Leak Checking
│   │   │   ├── Have I Been Pwned (haveibeenpwned.com) — official breach lookup
│   │   │   ├── DeHashed (dehashed.com) — paid; full breach data search
│   │   │   ├── IntelX (intelx.io) — breach data + darkweb index
│   │   │   ├── BreachDirectory (breachdirectory.org) — free breach lookup
│   │   │   └── LeakCheck (leakcheck.io)
│   │   │
│   │   └── Email → Social account pivot
│   │       ├── "Forgot password" on major platforms reveals partial phone / profile pic
│   │       ├── Search email in Google, LinkedIn, GitHub
│   │       └── GHunt — Google account OSINT from a Gmail address
│   │           └── pip install ghunt && ghunt email <target@gmail.com>
│   │
│   ├── 2.3 Phone Number Investigation
│   │   ├── Reverse phone lookup
│   │   │   ├── Truecaller (truecaller.com) — name from phone number
│   │   │   ├── NumVerify API — carrier, line type, location
│   │   │   ├── Phoneinfoga — framework for phone number OSINT
│   │   │   │   └── phoneinfoga scan -n "+91XXXXXXXXXX"
│   │   │   └── Sync.me, Eyecon — crowd-sourced contact lookups
│   │   │
│   │   ├── "Forgot password" technique
│   │   │   └── Enter phone on Gmail/Facebook/Instagram to see partial profile
│   │   │
│   │   └── Country/carrier identification
│   │       ├── First digits (country code) → country
│   │       └── NumVerify → carrier + line type (mobile/VoIP/landline)
│   │
│   ├── 2.4 Social Media OSINT
│   │   │
│   │   ├── General Approach
│   │   │   ├── Collect: profile picture, bio, username, posts, friends, location tags
│   │   │   ├── Archive everything — posts get deleted; screenshot + archive.ph
│   │   │   └── Never interact — liking/following alerts the target
│   │   │
│   │   ├── Twitter / X
│   │   │   ├── Advanced search: twitter.com/search-advanced
│   │   │   │   ├── from:<user> — tweets from a user
│   │   │   │   ├── to:<user> — tweets mentioning a user
│   │   │   │   ├── near:<city> within:<distance> — location tweets
│   │   │   │   └── since:YYYY-MM-DD until:YYYY-MM-DD — date range
│   │   │   ├── Twint — scrapes Twitter without API (no account needed)
│   │   │   │   └── twint -u <username> --since 2020-01-01
│   │   │   ├── Social Bearing (socialbearing.com) — Twitter analysis
│   │   │   └── Followerwonk — follower overlap analysis
│   │   │
│   │   ├── Instagram
│   │   │   ├── View profile without account: add ?__a=1 to URL (partially works)
│   │   │   ├── StoriesIG (storiesig.com) — view stories without account
│   │   │   ├── Imginn / Instaloader — bulk download public profile
│   │   │   │   └── instaloader <username>
│   │   │   └── Tagged location photos reveal physical location patterns
│   │   │
│   │   ├── Facebook
│   │   │   ├── Facebook ID lookup: graph.facebook.com/<username>
│   │   │   ├── IntelX and Sowdust's Graph Search tools
│   │   │   ├── Who Posted What (whopostedwhat.com) — keyword + date search
│   │   │   └── Lookup-id.com — find Facebook user ID from profile
│   │   │
│   │   ├── LinkedIn
│   │   │   ├── No account needed for basic profiles (Google: site:linkedin.com/in/ "name")
│   │   │   ├── Reveals job history → technology stack clues → pivot to tech OSINT
│   │   │   ├── CrossLinked — email harvesting from LinkedIn
│   │   │   │   └── python3 crosslinked.py -f "{first}.{last}@example.com" "Company Name"
│   │   │   └── ProxyCurl API — LinkedIn data extraction
│   │   │
│   │   ├── TikTok / YouTube / Reddit
│   │   │   ├── TikTok: search username, analyze posted location, audio metadata
│   │   │   ├── YouTube: channel ID → linked Google account → Google Maps reviews
│   │   │   └── Reddit: search.pullpush.io — searches ALL Reddit history
│   │   │       └── (Official Reddit search only shows recent posts)
│   │   │
│   │   └── Cross-Platform Correlation
│   │       ├── Same profile picture across platforms → reverse image search
│   │       ├── Same bio text → Google the phrase in quotes
│   │       ├── Same posting time patterns → infer time zone
│   │       └── Cross-post topics → build interest/behavioral profile
│   │
│   ├── 2.5 People Search Engines
│   │   ├── Spokeo (spokeo.com) — US people search
│   │   ├── Pipl (pipl.com) — deep people search
│   │   ├── BeenVerified (beenverified.com) — US records
│   │   ├── Intelius (intelius.com) — US people + address history
│   │   ├── TruePeopleSearch (truepeoplesearch.com) — US, free
│   │   └── Note: Most are US-centric; for India use Truecaller + LinkedIn
│   │
│   └── 2.6 Dark Web Mentions (Advanced)
│       ├── Ahmia (ahmia.fi) — Tor search engine, no Tor needed
│       ├── IntelX (intelx.io) — indexes darkweb + leaks + Tor
│       ├── DarkSearch (darksearch.io) — darkweb search
│       └── OnionSearch — Python tool for multi-engine darkweb search
│
│
├── LEVEL 3 — IMAGE & MEDIA OSINT
│   │
│   ├── 3.1 Reverse Image Search
│   │   ├── Google Lens (images.google.com) — best for faces and objects
│   │   ├── Yandex Images (yandex.com/images) — best reverse image tool overall
│   │   │   └── Especially strong for finding face matches
│   │   ├── TinEye (tineye.com) — finds exact copies + oldest version
│   │   ├── Bing Visual Search — good alternative to Google
│   │   └── PimEyes (pimeyes.com) — face search engine (paid; powerful)
│   │
│   ├── 3.2 Image Metadata (EXIF Data)
│   │   ├── What EXIF contains: GPS coordinates, camera model, date/time, software
│   │   ├── ExifTool — the standard tool
│   │   │   ├── exiftool image.jpg
│   │   │   └── exiftool -r /directory/   (recursive)
│   │   ├── Jeffrey's Exif Viewer (web-based: exifdata.com)
│   │   ├── Pic2Map (pic2map.com) — extracts GPS and shows on map
│   │   └── Note: Social media (Instagram, Facebook, Twitter) strips EXIF on upload
│   │       └── WhatsApp, Telegram, Signal also strip EXIF — direct shares may retain it
│   │
│   ├── 3.3 Geolocation from Images (GeoINT)
│   │   ├── Look for: signs, storefronts, license plates, architecture style, vegetation
│   │   ├── Sun angle + shadow direction → approximate direction photo was taken
│   │   ├── Google Street View — match landmarks and building facades
│   │   ├── GeoGuessr — trains geolocation intuition
│   │   ├── SunCalc (suncalc.org) — calculate sun position from date/time/location
│   │   └── Overpass Turbo (overpass-turbo.eu) — query OpenStreetMap features
│   │
│   ├── 3.4 Video OSINT
│   │   ├── Extract frames: ffmpeg -i video.mp4 -vf fps=1 frame%04d.jpg
│   │   ├── Reverse search each key frame via Google Lens / Yandex
│   │   ├── YouTube metadata: youtube-dl --get-description URL
│   │   ├── InVID / WeVerify (invid-project.eu) — video verification tool
│   │   └── Analyse background audio — language, music, ambient sounds = location clue
│   │
│   └── 3.5 AI Face Tools (Use Ethically)
│       ├── PimEyes — face search across internet
│       ├── FaceCheck.ID — face search engine
│       └── Legal note: Face recognition OSINT is heavily regulated in EU (GDPR)
│           └── Use only on public figures or with explicit consent
│
│
├── LEVEL 4 — ORGANISATION / INFRASTRUCTURE OSINT
│   │
│   ├── 4.1 Search Engine Reconnaissance
│   │   ├── Google Dorking
│   │   │   ├── site:example.com filetype:pdf
│   │   │   ├── site:example.com intitle:"index of"
│   │   │   ├── site:example.com inurl:admin OR inurl:login
│   │   │   ├── -site:www.example.com site:*.example.com  (subdomain enum)
│   │   │   ├── filetype:env "DB_PASSWORD"
│   │   │   └── site:pastebin.com "example.com"
│   │   │
│   │   ├── Google Hacking Database (GHDB)
│   │   │   └── exploit-db.com/google-hacking-database
│   │   │
│   │   └── Bing Operators
│   │       ├── ip:<address> — all domains hosted on that IP (virtual host discovery)
│   │       └── site:*.example.com
│   │
│   ├── 4.2 Device & Service Search Engines
│   │   ├── Shodan (shodan.io)
│   │   │   ├── Indexes: banners, open ports, service versions, SSL certs
│   │   │   ├── Key filters: org:, port:, product:, version:, country:, ssl:, vuln:
│   │   │   ├── Header-based dorks: "x-jenkins", "kbn-name: kibana"
│   │   │   ├── Favicon hash: http.favicon.hash:<hash>
│   │   │   ├── CLI: shodan search, shodan host, shodan domain
│   │   │   └── Shodan Monitor: alerts for new services / CVEs on monitored IPs
│   │   │
│   │   ├── Censys (search.censys.io)
│   │   │   ├── Strong certificate search: parsed.names: "*.example.com"
│   │   │   ├── Host search: autonomous_system.name: "Target Inc"
│   │   │   └── Better structured query language than Shodan
│   │   │
│   │   ├── FOFA (fofa.info)
│   │   │   ├── Strong Asian infrastructure coverage
│   │   │   └── Operators: domain=, cert=, title=, body=, header=
│   │   │
│   │   ├── Netlas (netlas.io) — modern, generous free tier
│   │   ├── ZoomEye (zoomeye.hk) — Chinese alternative
│   │   └── Criminal IP (criminalip.io) — threat intel + host search
│   │
│   ├── 4.3 Certificate Transparency (CT) Logs
│   │   ├── crt.sh — query: %.example.com → all subdomains with certs
│   │   │   └── curl "https://crt.sh/?q=%.example.com&output=json" | jq -r '.[].name_value' | sort -u
│   │   ├── Censys Certificates — structured cert queries
│   │   ├── Certspotter — monitoring and API access
│   │   └── Every TLS cert is logged permanently — internal/staging hosts often exposed
│   │
│   ├── 4.4 DNS Intelligence
│   │   ├── Record types: A, AAAA, CNAME, MX, NS, TXT, SOA, PTR, SRV
│   │   ├── dig example.com TXT → reveals cloud providers (SPF), SaaS tools (verification tokens)
│   │   ├── Zone transfer: dig axfr @ns1.example.com example.com
│   │   ├── WHOIS (domain): registrant, registrar, name servers, creation date
│   │   ├── WHOIS (IP): ASN, org, IP range owner
│   │   ├── Passive DNS: SecurityTrails, DNSDumpster, VirusTotal
│   │   ├── Reverse WHOIS: all domains registered by same email/org
│   │   │   └── ViewDNS.info → Reverse WHOIS
│   │   └── ASN lookup: bgp.he.net → full IP ranges owned by target
│   │
│   ├── 4.5 Subdomain Enumeration
│   │   ├── Passive (no target contact)
│   │   │   ├── Subfinder: subfinder -d example.com -silent
│   │   │   ├── Amass passive: amass enum -passive -d example.com
│   │   │   ├── theHarvester: theHarvester -d example.com -b all
│   │   │   └── crt.sh API
│   │   │
│   │   ├── DNS Resolution & Web Probing
│   │   │   ├── dnsx: cat subs.txt | dnsx -silent  (filter live hosts)
│   │   │   └── httpx: cat subs.txt | httpx -status-code -title -tech-detect
│   │   │
│   │   └── Active Brute-Force (authorized only)
│   │       ├── puredns: puredns bruteforce wordlist.txt example.com
│   │       └── dnsx wordlist: dnsx -d example.com -w wordlist.txt
│   │
│   ├── 4.6 Archive & URL Recon
│   │   ├── Wayback Machine (web.archive.org) — historical snapshots
│   │   ├── CDX API: curl "http://web.archive.org/cdx/search/cdx?url=example.com/*&output=text&fl=original"
│   │   ├── gau: gau --subs example.com  (Wayback + CommonCrawl + URLScan)
│   │   ├── waybackurls: waybackurls example.com
│   │   ├── JS analysis: extract endpoints with LinkFinder
│   │   ├── Parameterized URLs: cat urls.txt | grep "?" → injection candidates
│   │   └── URLScan.io — on-demand scan + archive of pages
│   │
│   ├── 4.7 Code & Repository Recon
│   │   ├── GitHub Dorking
│   │   │   ├── org:company filename:.env
│   │   │   ├── org:company "aws_secret_access_key"
│   │   │   ├── org:company "BEGIN RSA PRIVATE KEY"
│   │   │   └── org:company "mongodb://"
│   │   │
│   │   ├── Secret Scanning
│   │   │   ├── TruffleHog: trufflehog git https://github.com/org/repo.git
│   │   │   └── Gitleaks: gitleaks detect --source /path/to/repo
│   │   │
│   │   ├── Other platforms: GitLab, Bitbucket, Sourcegraph, Grep.app
│   │   │
│   │   └── Paste Sites: site:pastebin.com "example.com"
│   │
│   ├── 4.8 Employee & Org OSINT
│   │   ├── LinkedIn → employee names, job titles, tech stack used
│   │   ├── CrossLinked → harvest emails from LinkedIn
│   │   │   └── python3 crosslinked.py -f "{first}.{last}@example.com" "Company"
│   │   ├── Hunter.io → email format + employee list
│   │   ├── RocketReach → verified professional emails
│   │   ├── Crunchbase → funding, investors, acquisitions, leadership
│   │   └── theHarvester → bulk email + subdomain collection
│   │
│   └── 4.9 Document & Metadata OSINT
│       ├── Find public documents: site:example.com filetype:pdf OR docx OR xlsx
│       ├── FOCA — extracts metadata from public documents (Windows tool)
│       │   └── Finds: author names, software versions, printer names, internal paths
│       ├── ExifTool: exiftool document.pdf → reveals creating software + author
│       ├── metagoofil: automated public document metadata extractor
│       │   └── metagoofil -d example.com -t pdf,doc -o output/
│       └── Hidden metadata reveals: employee names, OS versions, internal server paths
│
│
├── LEVEL 5 — GEOSPATIAL & LOCATION OSINT (GeoINT)
│   │
│   ├── 5.1 Mapping Tools
│   │   ├── Google Maps — Street View for landmark matching
│   │   ├── Google Earth Pro — historical satellite imagery (free)
│   │   ├── Bing Maps — alternative satellite/aerial view
│   │   ├── OpenStreetMap — open-source; Overpass Turbo for querying features
│   │   └── Sentinel Hub (sentinel-hub.com) — free satellite imagery (ESA)
│   │
│   ├── 5.2 IP Geolocation
│   │   ├── ip-api.com — free JSON API: curl http://ip-api.com/json/<IP>
│   │   ├── ipinfo.io — org, ASN, approximate location
│   │   ├── MaxMind GeoIP — commercial database used by most tools
│   │   └── Limitations: Accuracy is city-level at best; VPN/proxy defeats it
│   │
│   ├── 5.3 Physical Location from Digital Traces
│   │   ├── Wi-Fi network names (SSIDs) → WiGLE (wigle.net) database
│   │   │   └── SSID in a photo/post → WiGLE → approximate physical location
│   │   ├── Bluetooth device names → MAC address lookup
│   │   ├── Cell tower IDs → OpenCelliD (opencellid.org)
│   │   └── IP address → rough city, ISP type
│   │
│   ├── 5.4 Flight & Maritime Tracking
│   │   ├── FlightAware (flightaware.com) — real-time flight tracking
│   │   ├── FlightRadar24 (flightradar24.com) — global live flight map
│   │   ├── ADS-B Exchange (adsbexchange.com) — unfiltered (no removals)
│   │   ├── MarineTraffic (marinetraffic.com) — ship tracking
│   │   └── VesselFinder (vesselfinder.com) — ship tracking alternative
│   │
│   ├── 5.5 Vehicle & Transport OSINT
│   │   ├── License plate lookup varies by country (India: vahan.nic.in)
│   │   ├── Vehicle history: Carfax (US), AutoDNA (EU)
│   │   └── Photo OSINT: license plate → owner via govt databases (India: RTO)
│   │
│   └── 5.6 Geofenced Social Media Search
│       ├── Twitter advanced: near:<city> within:<km>
│       ├── Instagram location tags → geotagged posts for a place
│       └── Snapchat Snap Map — public posts by location (web.snapchat.com/map)
│
│
├── LEVEL 6 — NETWORK & INFRASTRUCTURE OSINT
│   │
│   ├── 6.1 BGP & ASN Analysis
│   │   ├── BGP.he.net — full ASN, prefix, and peering data
│   │   ├── BGPView (bgpview.io) — ASN, prefixes, peers
│   │   ├── RIPE NCC, ARIN, APNIC WHOIS — regional IP registries
│   │   └── Find all IP ranges: whois -h whois.radb.net -- '-i origin AS<NUM>'
│   │
│   ├── 6.2 SSL/TLS Certificate Analysis
│   │   ├── crt.sh — full cert history per domain
│   │   ├── SSL Labs (ssllabs.com/ssltest) — cert chain, vulnerability grading
│   │   ├── TestSSL (testssl.sh) — CLI SSL analysis tool
│   │   └── Shodan ssl.cert.* filters — find all hosts using same cert
│   │
│   ├── 6.3 Cloud Infrastructure OSINT
│   │   ├── Identify cloud provider from IP
│   │   │   ├── AWS IP ranges: ip-ranges.amazonaws.com/ip-ranges.json
│   │   │   ├── Azure: Microsoft published IP range JSON
│   │   │   └── GCP: goog.json
│   │   │
│   │   ├── S3 / Cloud Storage bucket hunting
│   │   │   ├── GrayhatWarfare (buckets.grayhatwarfare.com) — public bucket search
│   │   │   ├── S3Scanner: s3scanner scan --bucket example-backup
│   │   │   ├── Google dork: site:s3.amazonaws.com "example"
│   │   │   └── Common patterns: company-name-backup, company-dev, company-assets
│   │   │
│   │   └── Firebase database discovery
│   │       └── URL pattern: https://<app-name>.firebaseio.com/.json
│   │           └── If returns data without auth → open database (critical finding)
│   │
│   ├── 6.4 Email Infrastructure Analysis
│   │   ├── MX record → identifies email provider (Google Workspace, Microsoft 365)
│   │   ├── SPF record → lists authorized mail servers
│   │   ├── DMARC record → rua= tag shows internal reporting email address
│   │   ├── DKIM record → confirms email signing infrastructure
│   │   └── MXToolbox (mxtoolbox.com) — visual email record analysis
│   │
│   └── 6.5 Threat Intelligence Feeds
│       ├── VirusTotal (virustotal.com) — hash, URL, IP, domain reputation
│       ├── AlienVault OTX (otx.alienvault.com) — free threat intel
│       ├── Abuse.ch — malware / C2 tracking (URLhaus, ThreatFox, Feodo)
│       ├── Shodan for C2: "Cobalt Strike" in banners
│       └── Censys / Criminal IP — known malicious infrastructure
│
│
├── LEVEL 7 — BUSINESS & FINANCIAL OSINT
│   │
│   ├── 7.1 Company Registry Data
│   │   ├── India: MCA21 (mca.gov.in) — directors, filings, CIN, registered address
│   │   ├── UK: Companies House (companieshouse.gov.uk) — free full filings
│   │   ├── US: SEC EDGAR (sec.gov/edgar) — public companies + filings
│   │   ├── EU: OpenCorporates (opencorporates.com) — 200+ jurisdictions
│   │   └── GLEIF (gleif.org) — global LEI (Legal Entity Identifier) search
│   │
│   ├── 7.2 Financial Data
│   │   ├── SEC EDGAR — 10-K, 10-Q filings reveal tech stack, vendors, risks
│   │   ├── Crunchbase — funding rounds, investors, acquisitions
│   │   ├── PitchBook / CB Insights — deeper VC/private equity data (paid)
│   │   └── Job postings → infer tech stack, expansion plans, internal projects
│   │
│   ├── 7.3 Court Records & Legal OSINT
│   │   ├── India: eCourts (ecourts.gov.in) — case status lookup
│   │   ├── US: PACER (pacer.gov) — federal court filings
│   │   ├── EU/UK: varies by country — often national portals
│   │   └── Legal filings often reveal internal emails, addresses, associates
│   │
│   └── 7.4 Job Posting Intelligence
│       ├── LinkedIn Jobs, Indeed, Naukri → tech stack from "Requirements" section
│       ├── "We use AWS, Terraform, Kubernetes, Splunk" → infrastructure revealed
│       ├── New security job postings → org may have had an incident
│       └── Remote location tags → office infrastructure details
│
│
├── LEVEL 8 — ACTIVE RECONNAISSANCE (Authorized Only)
│   │
│   ├── 8.1 Port Scanning
│   │   ├── Masscan — fast discovery across large ranges
│   │   │   └── sudo masscan <range> --top-ports 1000 --rate 1000
│   │   └── Nmap — detailed fingerprinting
│   │       ├── nmap -sV -sC --top-ports 1000 <target>
│   │       ├── nmap -p- -sS -sV -sC -T4 <target>  (full scan)
│   │       └── nmap --script=vuln,auth,discovery <target>  (NSE scripts)
│   │
│   ├── 8.2 Web Probing
│   │   ├── httpx — status, title, tech detection
│   │   │   └── cat hosts.txt | httpx -status-code -title -tech-detect
│   │   ├── EyeWitness — screenshots of web services (visual triage)
│   │   │   └── python3 EyeWitness.py --web -f urls.txt
│   │   └── Aquatone — screenshot + HTTP response triage tool
│   │
│   ├── 8.3 Technology Fingerprinting
│   │   ├── WhatWeb: whatweb -v example.com
│   │   ├── Wappalyzer browser extension — live tech detection
│   │   └── BuiltWith (builtwith.com) — tech history + current stack
│   │
│   ├── 8.4 Directory & Content Discovery
│   │   ├── ffuf: ffuf -w wordlist.txt -u https://example.com/FUZZ -fc 404
│   │   ├── gobuster: gobuster dir -u https://example.com -w wordlist.txt
│   │   └── Wordlists: SecLists/Discovery/Web-Content/
│   │
│   └── 8.5 API Endpoint Discovery
│       ├── Swagger/OpenAPI exposure: /api/docs, /swagger-ui, /api-docs
│       ├── JavaScript file analysis: LinkFinder, JSParser
│       ├── Postman collections: site:github.com "postman" "example.com"
│       └── API endpoint brute-force: ffuf with API wordlist
│           └── SecLists/Discovery/Web-Content/api/objects.txt
│
│
├── LEVEL 9 — OSINT AUTOMATION & FRAMEWORKS
│   │
│   ├── 9.1 All-in-One Frameworks
│   │   ├── Maltego — visual link analysis; paid but powerful; community edition free
│   │   │   └── Connects entities: person → email → domain → IP → phone
│   │   ├── SpiderFoot (spiderfoot.net)
│   │   │   ├── 200+ modules; web UI; fully automated OSINT
│   │   │   └── spiderfoot -l 127.0.0.1:5001  (start web UI)
│   │   ├── recon-ng — Metasploit-style modular framework
│   │   │   └── marketplace install all && modules load recon/domains-hosts/…
│   │   ├── OSINT Framework (osintframework.com) — visual reference map
│   │   └── Sn1per — automated recon + vulnerability scanner
│   │
│   ├── 9.2 Infrastructure-Specific Pipelines
│   │   ├── Subfinder + dnsx + httpx pipeline
│   │   │   └── subfinder -d target.com -silent | dnsx -silent | httpx -title
│   │   ├── Amass + Shodan correlation
│   │   ├── gau + ffuf (archive URLs → active fuzz)
│   │   └── Nuclei — template-based vulnerability scanner (post-recon)
│   │       └── nuclei -l urls.txt -t nuclei-templates/
│   │
│   ├── 9.3 People OSINT Automation
│   │   ├── Sherlock — username search (300+ sites)
│   │   ├── Maigret — deeper Sherlock with relationship data
│   │   ├── GHunt — Google account OSINT
│   │   ├── Phoneinfoga — phone number framework
│   │   └── H8mail — email breach lookup automation
│   │       └── h8mail -t target@email.com
│   │
│   └── 9.4 Reporting & Documentation Tools
│       ├── CherryTree — hierarchical note-taking for investigations
│       ├── Obsidian — linked notes; build a knowledge graph of findings
│       ├── Maltego — visual entity mapping (also a report artifact)
│       ├── Hunchly (Chrome extension) — automatic page capture during research
│       └── Screenshot tools: Flameshot (Linux), ShareX (Windows)
│
│
├── LEVEL 10 — ADVANCED OSINT TECHNIQUES
│   │
│   ├── 10.1 Sock Puppet Accounts
│   │   ├── Why: Access restricted content, follow targets, avoid burning real identity
│   │   ├── How to build: Unique name + AI face (thispersondoesnotexist.com)
│   │   │                  + burner email + VPN + dedicated browser profile
│   │   ├── Platform-specific aging: Post content for weeks before investigating
│   │   └── Never: Connect to personal accounts, reuse devices, or use home IP
│   │
│   ├── 10.2 Data Correlation & Pivoting
│   │   ├── One data point always leads to another — the OSINT pivot
│   │   ├── Email → username → social profiles → phone → address → associates
│   │   ├── IP → ASN → org → other IPs → other domains → exposed services
│   │   ├── Profile photo → reverse image → other platform accounts
│   │   └── Company reg. → directors → their personal domains → their infrastructure
│   │
│   ├── 10.3 Breach Data Analysis
│   │   ├── Sources: Have I Been Pwned, DeHashed, IntelX, BreachDirectory
│   │   ├── What to extract: email, password hash/plain, username, IP, phone
│   │   ├── Password reuse: found plaintext → try on other platforms (credential stuffing)
│   │   │   └── LEGAL NOTE: Testing credentials without authorization = crime
│   │   └── Credential patterns: "Summer2024!" → guess current password structure
│   │
│   ├── 10.4 Infrastructure Pivoting via Shared Attributes
│   │   ├── Shared SSL cert → find all other domains using the same cert
│   │   │   └── Shodan: ssl:"<cert_serial_or_org>"
│   │   ├── Shared favicon → find related hosts
│   │   │   └── Shodan: http.favicon.hash:<hash>
│   │   ├── Shared Google Analytics / Tag Manager ID
│   │   │   └── BuiltWith / PublicWWW → other sites using the same GA ID
│   │   ├── Shared nameserver → other domains managed by same team
│   │   └── Shared WHOIS email → reverse WHOIS → more domains
│   │
│   ├── 10.5 OSINT via Google Analytics / Tracking IDs
│   │   ├── View page source: look for UA-XXXXXXX-X or G-XXXXXXXXXX
│   │   ├── BuiltWith.com → search tracking ID → all sites using same ID
│   │   ├── SpyOnWeb (spyonweb.com) → GA ID / Adsense ID → related sites
│   │   └── RelatedWebsites → cross-domain attribution
│   │
│   ├── 10.6 Wireless & Physical OSINT
│   │   ├── WiGLE (wigle.net) — Wi-Fi network geolocation database
│   │   │   └── SSID or BSSID → physical location on a map
│   │   ├── Wardriving: map SSIDs in an area to confirm physical office location
│   │   └── Bluetooth: device names can leak org info (e.g., "CompanyName-Printer")
│   │
│   └── 10.7 Countering OSINT (Defensive)
│       ├── WHOIS privacy services — mask registrant info
│       ├── Remove personal info from people-search sites (opt-out pages)
│       ├── Audit your own digital footprint regularly (Google your name)
│       ├── Minimize public metadata on images and documents
│       ├── Use separate email addresses for different purposes
│       ├── Enable MFA everywhere — breach passwords alone won't help attacker
│       └── Monitor your domain/email in breach databases (HIBP alerts)
│
│
└── TOOLS MASTER REFERENCE
    │
    ├── Person OSINT
    │   ├── Sherlock, Maigret — username search
    │   ├── GHunt — Google/Gmail OSINT
    │   ├── Phoneinfoga — phone number recon
    │   ├── H8mail — breach email lookup
    │   ├── Twint — Twitter scraping
    │   ├── Instaloader — Instagram download
    │   └── Have I Been Pwned, DeHashed, IntelX — breach data
    │
    ├── Image & Media OSINT
    │   ├── Google Lens, Yandex Images — reverse image search
    │   ├── ExifTool — metadata extraction
    │   ├── InVID/WeVerify — video verification
    │   ├── PimEyes — face search
    │   └── SunCalc — sun angle analysis
    │
    ├── Organisation / Infrastructure OSINT
    │   ├── Shodan, Censys, FOFA, Netlas — device search
    │   ├── crt.sh, Certspotter — CT logs
    │   ├── Subfinder, Amass — subdomain enumeration
    │   ├── dnsx, httpx — DNS resolution and web probing
    │   ├── gau, waybackurls — archive URL collection
    │   ├── TruffleHog, Gitleaks — secret scanning
    │   ├── theHarvester — email + subdomain harvesting
    │   ├── Hunter.io, CrossLinked — email discovery
    │   ├── FOCA, ExifTool, metagoofil — document metadata
    │   └── GrayhatWarfare — public cloud buckets
    │
    ├── Geospatial OSINT
    │   ├── Google Earth Pro, Sentinel Hub — satellite imagery
    │   ├── WiGLE — Wi-Fi network geolocation
    │   ├── FlightAware, FlightRadar24 — flight tracking
    │   ├── MarineTraffic — ship tracking
    │   └── SunCalc, Overpass Turbo — geolocation analysis
    │
    ├── Frameworks & Automation
    │   ├── Maltego — visual link analysis
    │   ├── SpiderFoot — automated OSINT (200+ modules)
    │   ├── recon-ng — modular CLI framework
    │   ├── Nuclei — template-based vulnerability scanning
    │   └── CherryTree, Obsidian, Hunchly — documentation
    │
    └── Active Recon (Authorized Only)
        ├── Nmap — port scan + NSE scripts
        ├── Masscan — high-speed port discovery
        ├── ffuf, gobuster — directory/content discovery
        ├── EyeWitness, Aquatone — web screenshot triage
        └── Subzy — subdomain takeover detection
```

---

## Learning Order (Recommended Path)

```
START HERE
    │
    ▼
Level 1 — Foundations & Mindset (1–2 days)
    │
    ▼
Level 2 — Person OSINT (1 week)
  Practice: Research a public figure (politician, CEO, public author)
    │
    ▼
Level 3 — Image & Media OSINT (3–4 days)
  Practice: GeoGuessr + find location of a challenge image
    │
    ▼
Level 4 — Organisation OSINT (2 weeks)
  Practice: Run full recon on your own domain or a bug bounty target
    │
    ▼
Level 5 — GeoINT (3–4 days)
  Practice: Geolocate challenge images from Bellingcat or GeoGuessr
    │
    ▼
Level 6 — Network & Infrastructure OSINT (1 week)
  Practice: Map the full infrastructure of a company (your own)
    │
    ▼
Level 7 — Business & Financial OSINT (3–4 days)
  Practice: Research a publicly listed company via SEC/MCA filings
    │
    ▼
Level 8 — Active Recon on authorized targets (ongoing)
  Practice: TryHackMe / HackTheBox OSINT rooms + bug bounty
    │
    ▼
Level 9 — Automation & Frameworks (1 week)
  Practice: Build your own recon pipeline with Bash/Python
    │
    ▼
Level 10 — Advanced Techniques (ongoing)
  Practice: Bellingcat challenges, TraceLabs CTF, OSINT CTFs
```

---

## Practice Platforms

| Platform | Focus | URL |
|---|---|---|
| TryHackMe | OSINT + Recon rooms | tryhackme.com |
| HackTheBox | Infrastructure OSINT | hackthebox.com |
| Bellingcat | Real-world investigative OSINT | bellingcat.com |
| TraceLabs CTF | Missing persons OSINT (real cases) | tracelabs.org |
| OSINT Curious | Challenges + webinars | osintcurio.us |
| GeoGuessr | Geolocation training | geoguessr.com |
| Sofia Santos | OSINT exercises | gralhix.com |
| OZINT | OSINT challenges | ozint.eu |

---

## Key Mindset Reminders

```
1. Tools don't do OSINT. Investigators do — tools assist.
2. Every piece of data is a pivot point to more data.
3. Never touch a system. You are observing, not interacting.
4. Document everything as you go — you cannot un-see a page that gets deleted.
5. Two-source rule: don't report a finding you can only verify from one place.
6. Stay in scope. "It was public" is not a defense for unauthorized access.
```
