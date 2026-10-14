# OSINT — Level 7: Business & Financial OSINT
> Business and financial OSINT extracts intelligence from corporate
> registries, regulatory filings, court records, financial databases,
> and public procurement data. It answers: Who owns this company?
> Who controls it? What do they own? What have they filed? What legal
> problems do they have? What vendors do they use? This level is
> critical for due diligence, investigative journalism, fraud
> investigation, and understanding an organization's full footprint.

---

## 1. Why Business OSINT Matters

### 1.1 What Business Records Reveal

```
Corporate registry filings:
  ├── Legal name (may differ from trading name)
  ├── Registration number (CIN in India)
  ├── Registered address (physical location)
  ├── Directors and their personal details
  ├── Share capital and ownership structure
  ├── Date of incorporation (company age)
  ├── Status: active / dormant / struck off / under liquidation
  └── Charges (mortgages / loans registered against the company)

Financial filings:
  ├── Revenue, profit, and loss
  ├── Assets and liabilities
  ├── Auditor name (pivot: who else do they audit?)
  ├── Bankers (which bank holds their accounts)
  └── Related party transactions (payments to associated entities)

Director data:
  ├── DIN (Director Identification Number) — unique per director
  ├── Other companies the same director controls
  ├── Date of appointment and resignation
  └── Nationality and address
```

### 1.2 The Business OSINT Pivot Chain

```
One company → Directors → Their other companies → Their domains →
Their infrastructure → Their employees → Their email formats →
Back to the original company's wider network
```

---

## 2. India — MCA21 & Company Registry

### 2.1 MCA21 Overview

```
URL: mca.gov.in

MCA21 is the Ministry of Corporate Affairs portal for India.
All registered companies and LLPs must file documents here.

What is publicly accessible (free):
  ├── Company master data (CIN, status, address, incorporation date)
  ├── Director details (name, DIN, date of appointment)
  ├── Charge information (loans/mortgages on assets)
  └── Some basic filings

What requires payment:
  ├── Full annual returns (Form MGT-7)
  ├── Financial statements (Balance sheet, P&L)
  ├── Charge documents
  └── Board resolution filings
  (Cost: ₹100 per document)
```

### 2.2 MCA21 Search Workflow

```
Step 1: Go to mca.gov.in
Step 2: MCA Services → Master Data → Company/LLP Master Data
Step 3: Enter company name → search

Key data returned:
  CIN:                  U72200MH2010PTC123456
  │   ├── U = Unlisted (L = Listed)
  │   ├── 72200 = NIC Industry Code (72200 = Software)
  │   ├── MH = Maharashtra (state of registration)
  │   ├── 2010 = year of incorporation
  │   ├── PTC = Private Company (PLC = Public, FLC = Foreign, OPC = One Person)
  │   └── 123456 = sequential registration number

Company Status:         Active / Strike Off / Under Liquidation
Registered Address:     Full address
Email ID:               Registered company email
Date of Incorporation:  2010-03-15
Authorized Capital:     ₹10,00,000
Paid-Up Capital:        ₹1,00,000
Directors:              List of current directors with DIN

Step 4: Click on each Director DIN
  → Full name
  → Date of birth (age)
  → All companies they are a director in (CRITICAL pivot)

Step 5: Download filings (paid — ₹100 each)
  → Annual return (Form MGT-7): shareholder list
  → Financial statement: revenue, assets, liabilities
  → Charge documents: loans and mortgages
```

### 2.3 DIN-Based Director Mapping

```
Each director has a unique DIN (Director Identification Number).
Using a DIN, you can find ALL companies that person directs.

Lookup tool: mca.gov.in → DIN Services → DIN Inquiry

Or via third-party tools:
  Tofler:     tofler.in   → free director lookup
  Zaubacorp:  zaubacorp.com → free, comprehensive
  VCCircle:   vccircle.com  → focused on funded startups

Workflow:
  Company A → Director: John Doe (DIN: 01234567)
    → John Doe is director in: Company A, Company B, Company C, Company D
  Company B → has different director: Jane Doe (DIN: 07654321)
    → Jane Doe is director in: Company A, Company B, Company E, Company F
  → Map: who controls which companies → ownership network
```

### 2.4 Free Third-Party MCA Data Portals

```
Tofler:     tofler.in
  → Free basic data: CIN, status, directors, charges
  → Paid: full financials, shareholder data

Zaubacorp:  zaubacorp.com
  → Free: company master data, director networks
  → Visual director network graph
  → Search by: name, CIN, director name, address

ICICI Direct Companies: (if listed)
  → Listed companies → BSE/NSE filings

Startupindia:  startupindia.gov.in
  → DPIIT-registered startups → funding, sector, contact

ClearTax:  cleartax.in/gstin-search
  → Search by GSTIN (GST number) → reveals company name, state, status
  → GSTIN format: 29AABCT1332L1ZA
    ├── 29 = state code (Karnataka)
    ├── AABCT1332L = PAN number
    ├── 1 = entity number
    ├── Z = default
    └── A = checksum
```

### 2.5 GSTIN Intelligence

```
GSTIN (Goods and Services Tax Identification Number) is public.

Lookup: gst.gov.in → Search Taxpayer → By GSTIN

Returns:
  ├── Legal name of business
  ├── Trade name
  ├── State of registration
  ├── Registration date
  ├── Business type (Regular / Composition / Non-Resident)
  ├── GSTIN status (Active / Cancelled / Suspended)
  └── Principal and additional place of business addresses

Multiple GSTINs = multiple states of operation
→ Reveals physical presence across India

Reverse search (by PAN):
  Enter PAN → returns all GSTINs registered under that PAN
  → If PAN belongs to a director → reveals all their business entities
```

---

## 3. Global Company Registries

### 3.1 OpenCorporates

```
URL: opencorporates.com

The largest open database of company information globally.
Covers 200+ jurisdictions — free for basic lookups.

Search by:
  → Company name → all matching entities worldwide
  → Officer/director name → all companies they appear in
  → Company address → all companies at that address
  → Company number → specific company by registration number

API (free tier: 500 requests/month):
curl "https://api.opencorporates.com/v0.4/companies/search?q=Target+Corp&jurisdiction_code=gb"
curl "https://api.opencorporates.com/v0.4/officers/search?q=John+Doe"

Intelligence uses:
  → Find all subsidiaries and related companies
  → Map a director's global company portfolio
  → Find offshore entities (BVI, Cayman Islands, Malta)
  → Track mergers and acquisitions via name changes
  → Discover dissolved companies that may still have active infrastructure
```

### 3.2 UK — Companies House

```
URL: companieshouse.gov.uk
API: find-and-update.company-information.service.gov.uk

UK has the most transparent company registry globally.
All filings are publicly free — no payment required.

What is available for free:
  ├── Full accounts (financial statements)
  ├── Confirmation statements (shareholder list)
  ├── Director details (name, DOB partial, nationality, address)
  ├── PSC register (People with Significant Control — beneficial owners)
  ├── Charges (mortgages/loans)
  ├── Officers history (past and present directors)
  └── Insolvency events

PSC (Persons with Significant Control) register:
  → UK law requires disclosure of ultimate beneficial owners
  → Name, nationality, date of birth (month/year), service address
  → What control they have: >25% shares, >25% voting rights, director control

API (free, no key for basic queries):
curl "https://api.company-information.service.gov.uk/search/companies?q=Target+Corp"
curl "https://api.company-information.service.gov.uk/company/12345678"
curl "https://api.company-information.service.gov.uk/company/12345678/officers"
curl "https://api.company-information.service.gov.uk/company/12345678/persons-with-significant-control"
curl "https://api.company-information.service.gov.uk/company/12345678/filing-history"
```

### 3.3 US — SEC EDGAR

```
URL: sec.gov/edgar

EDGAR contains filings from all US publicly traded companies.
Free — no account required.

Key filing types:

10-K   →  Annual report (most comprehensive)
         Contents: business overview, risk factors, financial data,
         executive compensation, legal proceedings, properties

10-Q   →  Quarterly report (similar to 10-K, less detailed)

8-K    →  Current report (material events: acquisitions, CEO changes,
          data breaches, regulatory actions)

DEF 14A → Proxy statement (executive pay, board structure, shareholder votes)

S-1    →  IPO registration statement (before going public — extremely detailed)

13F    →  Institutional investor holdings (what stocks large funds own)
          → Reveals which funds have invested in a company

Intelligence value from 10-K:
  Risk Factors section:
    → Reveals technology stack ("we rely on AWS for hosting")
    → Reveals key vendors ("our payment processor is Stripe")
    → Reveals known security incidents ("we experienced a data breach in...")
    → Reveals regulatory issues

  Properties section:
    → All physical office locations worldwide

  Legal Proceedings section:
    → Ongoing lawsuits → parties involved, nature of dispute

EDGAR full-text search: efts.sec.gov/LATEST/search-index?q="target+corp"&dateRange=custom&startdt=2024-01-01

API:
curl "https://efts.sec.gov/LATEST/search-index?q=%22Target+Corp%22&forms=10-K"
curl "https://data.sec.gov/submissions/CIK0000123456.json"
```

### 3.4 US — State-Level Registries

```
Each US state has its own business registry.
Useful when a company is not publicly traded (not on EDGAR).

Key state registries:
  Delaware:    icis.corp.delaware.gov/ecorp/  (most companies incorporate here)
  California:  bizfileonline.sos.ca.gov
  New York:    dos.ny.gov/corps/bus_entity_search.html
  Texas:       mycpa.cpa.state.tx.us/coa/Index.html

What they contain:
  → Registered agent name and address
  → Officers (in many states)
  → Formation date
  → Status (active / dissolved)

OpenCorporates aggregates US state data — often easier to use.
```

### 3.5 EU — European Business Registries

```
OpenCorporates covers most EU countries.

Country-specific portals:
  Germany:    unternehmensregister.de  (full filings, free)
  France:     societe.ninja / infogreffe.fr
  Netherlands: openkvk.nl
  Spain:      registradores.org
  Ireland:    cro.ie (Companies Registration Office)

GLEIF — Global LEI Search:
  URL: gleif.org
  LEI = Legal Entity Identifier
  Search any company globally → get LEI, jurisdiction, status
  Also reveals: parent company, ultimate parent (ownership hierarchy)

  API:
  curl "https://api.gleif.org/api/v1/lei-records?filter[entity.legalName]=%22Target+Corp%22"
```

---

## 4. Financial Data Intelligence

### 4.1 Crunchbase

```
URL: crunchbase.com

The primary database for startup and venture capital intelligence.

Free tier: limited views per month
Paid: full access to all data

What it contains:
  ├── Funding rounds (seed, Series A/B/C, IPO)
  │   → Date, amount, investors, lead investor
  ├── Investors and board members
  ├── Acquisitions (what they bought, when, for how much)
  ├── Acquisitions BY others (was this company acquired?)
  ├── Team and leadership
  ├── Website, social media links
  ├── Technologies used (from job postings integration)
  └── Similar companies (competitive landscape)

Intelligence uses:
  Funding amount → company size and maturity
  Investors → who has board seats → governance structure
  Acquisitions → new domains/assets to add to recon scope
  Lead investor → may have insider knowledge of their tech stack
  Funding timeline → identify periods of rapid growth = infrastructure expansion
```

### 4.2 Tracxn & PitchBook (Startup Intelligence)

```
Tracxn:   tracxn.com   → free tier available; covers Indian startups well
PitchBook: pitchbook.com → comprehensive; subscription required

Use for:
  → Funding history of private companies
  → Investor networks
  → Market maps (who are the competitors?)
  → Technology categorization
```

### 4.3 Annual Reports (Publicly Listed Companies)

```
Where to find:
  Company investor relations page: ir.example.com
  BSE (Bombay Stock Exchange): bseindia.com → Company Info → Annual Report
  NSE (National Stock Exchange): nseindia.com → Company Info
  SEC EDGAR (US): sec.gov/edgar

What to extract from annual reports:
  Management Discussion & Analysis (MD&A):
    → Technology investments for the year
    → New systems being deployed
    → Digital transformation initiatives
    → Cybersecurity investments (or lack of)

  Notes to Financial Statements:
    → Related party transactions → associated companies
    → Contingent liabilities → ongoing legal matters
    → Lease commitments → office locations and sizes

  Auditor's Report:
    → Auditor name (Big Four? smaller firm?)
    → Any qualifications or concerns noted

  Director's Report:
    → New subsidiary incorporations
    → Acquisitions made during the year
    → Regulatory compliance status
```

### 4.4 BSE / NSE Filings (India)

```
BSE: bseindia.com
NSE: nseindia.com

For publicly listed Indian companies:
  → All stock exchange filings are public
  → Includes: quarterly results, board meeting outcomes,
    shareholding patterns, insider trading disclosures,
    corporate actions (dividends, splits, mergers)

Shareholding pattern (quarterly disclosure):
  → Shows: promoter holding %, FII holding %, public holding %
  → Change in promoter holding → confidence signal
  → Large FII entry → institutional interest

Insider trading disclosures:
  → Directors must disclose share purchases/sales
  → Large insider buying → positive signal
  → Panic selling → potential pre-announcement of bad news

CDSL / NSDL (depositories):
  → Track beneficial ownership of shares
  → Less accessible for OSINT; requires demat account context
```

---

## 5. Court Records & Legal Intelligence

### 5.1 Why Court Records Are Valuable

```
Court filings contain:
  ├── Names and addresses of all parties
  ├── Detailed allegations (may reveal internal operations)
  ├── Evidence submitted (sometimes internal documents)
  ├── Expert witnesses and their qualifications
  ├── Settlement amounts (sometimes)
  ├── Associated persons and entities named in the case
  └── Legal arguments that reveal internal facts

Types of cases to look for:
  → Employment disputes → reveals internal HR policies, misconduct
  → Contract disputes → reveals vendor relationships, payment terms
  → IP litigation → reveals what technologies/products they own
  → Data breach litigation → reveals breach details not in press
  → Regulatory enforcement → reveals compliance failures
  → Insolvency/bankruptcy → reveals financial distress
```

### 5.2 India — eCourts

```
URL: ecourts.gov.in

Covers all District Courts and High Courts in India.
Supreme Court: supremecourt.nic.in

Search by:
  → Party name (company or individual)
  → Case number
  → Advocate name
  → FIR number (for criminal cases)

Returns:
  → Case title (parties involved)
  → Case type and number
  → Filing date
  → Current status (pending / disposed)
  → Next date of hearing
  → Order/judgment documents (PDF) — free

Cause list lookup:
  → See all cases scheduled for a specific court on a specific date
  → Search by: party name → find all pending cases

High Courts:
  Bombay HC:     bombayhighcourt.nic.in
  Delhi HC:      delhihighcourt.nic.in
  Madras HC:     hcmadras.tn.nic.in
  Calcutta HC:   calcuttahighcourt.gov.in
  NCLAT:         nclat.nic.in (company law appellate tribunal)
  NCLT:          nclt.gov.in (company law tribunal — insolvency, mergers)

NCLT is particularly valuable:
  → All insolvency proceedings are filed here
  → Corporate restructuring and mergers
  → Oppression and mismanagement cases
```

### 5.3 US — PACER (Federal Courts)

```
URL: pacer.gov

PACER (Public Access to Court Electronic Records) covers all US federal courts.
Cost: $0.10 per page (waived if under $30/quarter)

What it contains:
  → All federal civil cases
  → Federal criminal cases
  → Bankruptcy cases
  → Appellate cases

Free alternatives:
  CourtListener: courtlistener.com
    → Aggregates PACER + state court data for free
    → Full-text search across all federal opinions
    → REST API available

  RECAP: law.free.law/recap
    → Browser extension that uploads PACER documents to a free archive
    → Search the RECAP archive before paying PACER

  Justia: law.justia.com
    → Free federal case law (opinions only)

Google dork for court filings:
  site:courtlistener.com "Target Corp"
  site:pacer.gov "Target Corp"
```

### 5.4 Regulatory Filings & Enforcement Actions

```
India:
  SEBI (Securities and Exchange Board of India):
    sebi.gov.in → Enforcement → Orders
    → All enforcement orders against market participants
    → Insider trading cases, fraudulent trading, disclosure violations

  RBI (Reserve Bank of India):
    rbi.org.in → Notifications → Enforcement
    → Penalties against banks and NBFCs
    → Compounding orders

  CCI (Competition Commission of India):
    cci.gov.in → Orders
    → Antitrust and merger cases
    → Reveals market dominance and anti-competitive behavior

  IRDAI (Insurance Regulatory Authority):
    irdai.gov.in → Enforcement
    → Insurance company violations

USA:
  SEC Enforcement: sec.gov/litigation/litreleases
  FTC Actions:     ftc.gov/news-events/press-releases
  CFTC:            cftc.gov/LawRegulation/Enforcement
  DOJ:             justice.gov/news (press releases for federal actions)
  State AG actions: each state attorney general website

EU:
  European Commission Competition:
    ec.europa.eu/competition-policy/antitrust/cases
  GDPR enforcement:
    enforcementtracker.com
    → Tracks all GDPR fines across EU member states
    → Search by company or country
    → Shows fine amount, violation type, DPA that issued it
```

---

## 6. Job Posting Intelligence

### 6.1 Why Job Postings Are OSINT Gold

```
A job posting is a company voluntarily advertising their:
  → Technology stack ("Experience with AWS, Terraform, Kubernetes")
  → Security tooling ("Knowledge of Splunk, CrowdStrike, Okta")
  → Internal projects ("Working on our new data lake on Snowflake")
  → Expansion plans ("Opening new office in Hyderabad")
  → Current weaknesses ("Help us build our SOC from scratch")
  → Vendor relationships ("Experience with our ERP: SAP S/4HANA")
  → Team structure ("Report to the CISO of a 500-person security team")
```

### 6.2 Platforms to Search

```
LinkedIn Jobs:  linkedin.com/jobs
Indeed:         indeed.co.in
Naukri:         naukri.com        (India primary)
Glassdoor:      glassdoor.co.in
AngelList/Wellfound: wellfound.com  (startups)
iimjobs:        iimjobs.com       (senior roles in India)
Instahyre:      instahyre.com
Greenhouse:     greenhouse.io
Lever:          lever.co
Workable:       workable.com

Google dork for job postings:
  site:linkedin.com/jobs "Target Corp" "security engineer"
  site:naukri.com "Target Corp" "cloud infrastructure"
  "Target Corp" "we are looking for" "experience with"
```

### 6.3 Technology Stack Extraction from Job Postings

```
Collect ALL job postings from a company over 6-12 months
→ Build a comprehensive technology inventory

Categories to extract:

Cloud & Infrastructure:
  "AWS / Azure / GCP"     → cloud provider
  "Terraform / Ansible"   → IaC tooling
  "Kubernetes / Docker"   → container platform
  "VMware"                → on-premise virtualization

Security Tooling:
  "SIEM: Splunk / QRadar / Sentinel" → log management
  "EDR: CrowdStrike / SentinelOne"   → endpoint security
  "WAF: Akamai / Cloudflare"         → web protection
  "IAM: Okta / Azure AD / Ping"      → identity provider
  "PAM: CyberArk / BeyondTrust"      → privileged access
  "Vulnerability: Qualys / Tenable"  → vulnerability management

Development:
  "Python / Go / Java / Node.js"     → primary languages
  "React / Angular / Vue"            → frontend framework
  "PostgreSQL / MySQL / MongoDB"     → databases
  "Kafka / RabbitMQ"                 → message queuing
  "GitHub / GitLab / Bitbucket"      → code hosting

Data & Analytics:
  "Snowflake / Databricks / Redshift" → data warehouse
  "Spark / Hadoop"                    → data processing
  "Tableau / PowerBI / Looker"        → BI tooling

Example analysis:
  Job: "Senior Cloud Security Engineer at Target Corp
        Requirements: AWS, Terraform, Kubernetes, Okta, CrowdStrike Falcon,
        Splunk SIEM, Qualys VMDR, experience with PCI-DSS compliance"
  Intelligence:
    ├── Cloud: AWS (confirmed)
    ├── IaC: Terraform (confirmed)
    ├── Containers: Kubernetes (confirmed)
    ├── IAM: Okta (confirmed)
    ├── EDR: CrowdStrike Falcon (confirmed)
    ├── SIEM: Splunk (confirmed)
    ├── VM: Qualys VMDR (confirmed)
    └── Compliance scope: PCI-DSS → processes payment card data
```

### 6.4 Headcount & Growth Intelligence

```
LinkedIn company page → Employee count + growth chart
  → Rapid growth → infrastructure scaling → new systems being deployed
  → Layoffs → cost-cutting → potential security tool downgrade

LinkedIn employee count by department:
  → Filter employees by: Engineering / Security / IT / Finance
  → Large engineering team → complex internal infrastructure
  → Small security team → potentially understaffed → more risks

Glassdoor reviews:
  → "We use ancient legacy systems" → old, potentially vulnerable tech
  → "The security team is overwhelmed" → thin coverage
  → "No MDM on company laptops" → endpoint management gap
  → Reviews mentioning specific tools by name

Layoff.fyi (layoffs.fyi):
  → Tracks tech layoffs globally
  → Which departments were cut (security team reduction = risk indicator)
```

---

## 7. Procurement & Government Contract Intelligence

### 7.1 India — Government Procurement

```
GeM (Government e-Marketplace): gem.gov.in
  → Search vendor by: name, category, state
  → View: all government contracts won by a vendor
  → Reveals: product lines, government clients, contract sizes

CPPP (Central Public Procurement Portal): eprocure.gov.in
  → All central government tenders
  → Awarded tender details: vendor name, amount, department

State e-procurement portals:
  → Each state has its own portal
  → Maharashtra: mahatenders.gov.in
  → Karnataka: kppp.karnataka.gov.in
  → Search by: vendor name or product category

PFMS (Public Financial Management System): pfms.nic.in
  → Tracks government expenditure
  → Search by: vendor / recipient
  → Reveals: government payments to a company

Intelligence value:
  Company wins contract for "Cybersecurity Assessment of National Highways"
  → They have government security clearance
  → They have experience in specific sectors
  → May have access to sensitive government infrastructure
```

### 7.2 US — Government Contracts

```
USASpending: usaspending.gov
  → All US federal contract and grant spending
  → Search by: vendor name, NAICS code, agency, fiscal year
  → Returns: contract amounts, agencies, contract descriptions

SAM.gov: sam.gov
  → System for Award Management
  → All registered federal contractors
  → Includes: CAGE code, DUNS/UEI number, socioeconomic status,
    active exclusions (debarred companies)

FPDS (Federal Procurement Data System): fpds.gov
  → Detailed federal contract data
  → More granular than USASpending

Intelligence value:
  Company has contract for "Cloud Migration of [Agency] Systems"
  → They have access to government cloud infrastructure
  → May have FedRAMP authorization
  → Know the agency's systems inside out
```

### 7.3 EU — TED (Tenders Electronic Daily)

```
URL: ted.europa.eu

All EU public procurement above threshold values.
Search by: company name, CPV code (product category), country

Also: each EU member state has its own procurement portal
```

---

## 8. Beneficial Ownership & Shell Company Research

### 8.1 What Beneficial Ownership Research Finds

```
A company may be owned by another company, which is owned by a trust,
which is managed by a foundation in a tax haven. This is called
"layered ownership" — designed to obscure who ultimately controls the assets.

Beneficial ownership OSINT uncovers:
  → Who ultimately controls a company (despite layers)
  → Hidden connections between entities
  → Offshore structures and tax havens
  → Shell companies with no real business
  → Asset hiding (often in divorce, fraud, or sanctions evasion cases)
```

### 8.2 ICIJ Offshore Leaks Database

```
URL: offshoreleaks.icij.org

The International Consortium of Investigative Journalists (ICIJ) maintains
a free, searchable database from major leaked datasets:
  → Panama Papers (2016)
  → Paradise Papers (2017)
  → Pandora Papers (2021)
  → Offshore Leaks (2013)
  → Bahamas Leaks (2016)
  → FinCEN Files
  → Luanda Leaks

Search by: person name, company name, country

Returns:
  → Entities in offshore jurisdictions (BVI, Cayman, Jersey, etc.)
  → Connected persons (directors, beneficiaries, shareholders)
  → Registered agents
  → Addresses

IMPORTANT: Appearance in this database ≠ illegal activity.
           Many offshore structures are legal.
           It is investigative context, not proof of wrongdoing.
```

### 8.3 OpenOwnership — Beneficial Ownership Register

```
URL: register.openownership.org

Aggregates beneficial ownership disclosures from participating countries.
Searchable by: company name, person name, country

Currently covers: UK, Ukraine, Denmark, Norway, Slovakia, and others.
India is not yet a full participant — use MCA21 for Indian data.
```

### 8.4 GLEIF — LEI Ownership Chains

```
URL: gleif.org/en/lei-data/gleif-concatenated-file/download-the-concatenated-file

LEI (Legal Entity Identifier) records include:
  → Direct parent entity
  → Ultimate parent entity

API:
curl "https://api.gleif.org/api/v1/lei-records?filter[entity.legalName]=Target+Corp"

# Get parent relationships
curl "https://api.gleif.org/api/v1/lei-records/LEI_CODE/ultimate-parent-relationship"

This builds the ownership chain upward to the ultimate controller.
```

### 8.5 Company Network Visualization Tools

```
Orbis (Bureau van Dijk): orbis.bvdinfo.com
  → Most comprehensive company ownership database (paid; used by banks)
  → Maps ownership chains globally
  → Used for KYC/AML compliance

LittleSis: littlesis.org
  → Free database of connections between powerful people and organizations
  → Focus: US politicians, corporations, lobbyists
  → Visual network maps

Kumu: kumu.io
  → Create your own entity relationship maps
  → Import data from spreadsheets

Maltego:
  → Run corporate registry transforms
  → Visualize director → company → IP → domain relationships
```

---

## 9. Domain & Asset Ownership Research

### 9.1 Connecting Domains to Companies

```
Method 1: WHOIS registrant org
  whois example.com | grep "Registrant Organization"
  → "Target Corp" confirmed as domain owner

Method 2: SSL certificate organization
  curl "https://crt.sh/?q=example.com&output=json" \
    | python3 -c "import json,sys; [print(c.get('issuer_name')) for c in json.load(sys.stdin)]"

Method 3: Company website → WHOIS cross-reference
  Company claims to own domain → verify via WHOIS registrant

Method 4: TM (Trademark) database
  India: ipindia.gov.in/trade-marks.htm
  US:    tmsearch.uspto.gov
  → Search trademark by brand name → see registrant
  → Trademark owner = company that controls the brand

Method 5: Google AdSense / Analytics ID (see Level 10)
  → Shared tracking IDs across domains confirm same operator
```

### 9.2 Intellectual Property as OSINT

```
Patents:
  India:   ipindia.gov.in → patent search
  US:      patents.google.com (Google Patents — free, comprehensive)
  Global:  patentscope.wipo.int (WIPO — international patents)

  Patent application reveals:
    → Inventor names (engineers and researchers)
    → Assignee (company that owns the patent)
    → Technical claims (what technology they have developed)
    → Filing date (when the technology was developed)
    → Priority date (original invention date)

Trademarks:
  India:   ipindia.gov.in/trade-marks.htm
  US:      tmsearch.uspto.gov
  EU:      euipo.europa.eu/eSearch

  Trademark registration reveals:
    → All brand names owned by the company
    → Classes of goods/services they operate in
    → Date of first use (how long the brand has been in market)

Copyrights:
  India:   copyright.gov.in (search by title or author)
  US:      copyright.gov/public-records

Intelligence use:
  Company files patent for "Distributed Ledger Security System"
  → They are developing blockchain security products
  → Inventors named → LinkedIn search → find the team
  → Technical claims → understand exactly what they are building
```

---

## 10. Sanctions, PEPs & Watch Lists

### 10.1 Sanctions Lists

```
Sanctions lists identify individuals and entities that are subject
to financial and trade restrictions by governments.

OFAC (US):
  URL: sanctionssearch.ofac.treas.gov
  → US Treasury Office of Foreign Assets Control
  → Searches: SDN (Specially Designated Nationals) list
  → Includes: individuals, companies, vessels, aircraft
  → API: ofac.treasury.gov/api/download/SDN.xml

UN Security Council:
  URL: scsanctions.un.org/consolidatedlist
  → Global sanctions list

EU Sanctions:
  URL: data.europa.eu/data/datasets/consolidated-list-of-persons-groups-and-entities-subject-to-eu-financial-sanctions

UK OFSI:
  URL: gov.uk/government/publications/financial-sanctions-consolidated-list-of-targets

India — MEA:
  mea.gov.in → UN sanctions implementation
  → India implements UNSC sanctions

Open Sanctions:
  URL: opensanctions.org
  Aggregates 150+ sanctions and watchlist sources into one searchable database.
  Free API:
  curl "https://api.opensanctions.org/search/default?q=target+name&schema=Company"
```

### 10.2 PEP (Politically Exposed Persons)

```
PEPs are individuals in prominent public positions:
  → Government officials and their families
  → Senior executives of state-owned companies
  → Senior military officers
  → Senior judiciary officials
  → Senior political party officials

Why it matters:
  → PEPs are higher risk for corruption, bribery, and money laundering
  → Companies must conduct enhanced due diligence on PEPs
  → Connections to PEPs may indicate political risk

Free PEP databases:
  Every Politician: everypolitician.org
  OpenSanctions:    opensanctions.org (includes PEPs)
  Wikidata:         wikidata.org → sparql queries for politicians
  Wikipedia:        list of politicians → their business connections

India-specific:
  ADR (Association for Democratic Reforms): myneta.info
  → All candidate affidavits for elections
  → Search by: name, party, constituency
  → Reveals: declared assets, liabilities, criminal cases, educational qualifications
  → Filed by candidates themselves → self-declared data
```

### 10.3 myneta.info — India Election Affidavits

```
URL: myneta.info

Every candidate standing for election in India must file an affidavit
declaring their assets, liabilities, criminal record, and income.

What it reveals:
  ├── Declared assets (movable and immovable property)
  ├── Spouse's assets
  ├── Outstanding liabilities (loans)
  ├── Criminal cases (pending and decided)
  ├── Educational qualifications
  └── Annual income (declared)

Search by: candidate name, constituency, election year

Intelligence use:
  A company director contests or supports a politician
  → myneta.info → their declared assets
  → Cross-reference with company filings → are assets declared?
  → Unexplained wealth gap → corruption indicator
```

---

## 11. Business Intelligence Workflow

### 11.1 Full Business OSINT Pipeline

```
Target: "Target Corp" (Indian private limited company)

Phase 1: Corporate Identity
  └── MCA21 → CIN, directors, address, incorporation date
  └── Zaubacorp → director network map
  └── OpenCorporates → any international entities
  └── GLEIF → LEI and parent company chain

Phase 2: Financial Intelligence
  └── Tofler / Zaubacorp → financial summary (free)
  └── MCA21 → download annual filing (₹100)
  └── Crunchbase → funding history (if startup)
  └── BSE/NSE → if listed → all exchange filings

Phase 3: Legal & Regulatory
  └── eCourts → search by company name
  └── NCLT → insolvency or company law proceedings
  └── SEBI → enforcement orders
  └── RBI → if financial entity → penalties

Phase 4: Tax & GST
  └── GST portal → GSTIN → state registrations
  └── PAN → reverse GSTIN search → all entities under same PAN

Phase 5: Technology Intelligence
  └── LinkedIn → job postings → tech stack extraction
  └── Glassdoor → employee reviews → internal tool mentions
  └── BuiltWith → current and historical tech stack

Phase 6: People
  └── LinkedIn → employees → job titles → org chart
  └── Hunter.io → email format → employee emails
  └── CrossLinked → bulk email harvest

Phase 7: Procurement
  └── GeM → government contracts
  └── CPPP → central government tenders

Phase 8: IP Assets
  └── IP India → patents and trademarks
  └── Google Patents → all patent applications

Phase 9: Adverse Media
  └── Google News → "Target Corp" site:news.google.com
  └── GDELT → global news event database
  └── Factiva / LexisNexis (paid) → full newspaper archive
```

### 11.2 Adverse Media Search

```
Google News searches:
  "Target Corp" fraud
  "Target Corp" breach OR hack OR "data leak"
  "Target Corp" lawsuit OR litigation OR settlement
  "Target Corp" fine OR penalty OR regulatory
  "Target Corp" bankruptcy OR insolvency OR liquidation
  "Target Corp" investigation OR probe OR arrested

Google dork for news:
  site:timesofindia.com OR site:economictimes.com OR site:livemint.com "Target Corp"

GDELT (Global Database of Events, Language, and Tone):
  URL: gdeltproject.org
  → Monitors news worldwide in 100+ languages
  → Search by: company name, event type, location
  → Free API with full-text search

Factiva / LexisNexis (paid):
  → Full newspaper and wire archive going back decades
  → Used by banks, law firms, and compliance teams
  → Academic access often available via university library
```

---

## 12. Level 7 Practice Tasks

### Task 1 — Indian Company Deep Dive
- Pick any Indian private limited company (not publicly listed)
- Extract from MCA21: CIN, all directors with DINs, registered address, incorporation date
- For each director: find all other companies they direct using Zaubacorp
- Map the director network — how many companies share at least one director?
- Find the GSTIN and check state-wise registrations

### Task 2 — UK Companies House Investigation
- Search Companies House for a UK company
- Download the full PSC (Persons with Significant Control) register
- Find: ultimate beneficial owner, % control, nationality
- Check if any directors also appear in the ICIJ Offshore Leaks database

### Task 3 — SEC EDGAR Filing Intelligence
- Find the most recent 10-K of any US tech company
- Extract from the filing:
  - All office locations listed in the Properties section
  - All legal proceedings mentioned
  - Technology dependencies mentioned in Risk Factors
  - Executive compensation (from DEF 14A if available)

### Task 4 — Job Posting Tech Stack Extraction
- Collect 10 current job postings from one company (LinkedIn / Naukri)
- Build a complete technology inventory from the requirements sections
- Categorize by: Cloud, Security Tooling, Development, Data
- Identify any EOL (End of Life) technologies mentioned
- Note: does any posting reveal a specific internal project name?

### Task 5 — Sanctions & PEP Check
- Take 5 company names or executive names from previous exercises
- Check each against:
  - OpenSanctions (opensanctions.org)
  - OFAC SDN list
  - ICIJ Offshore Leaks
- Document any matches found and their context

### Task 6 — Election Affidavit Research (India)
- Go to myneta.info
- Search for any sitting Member of Parliament or MLA from your state
- Extract: declared assets (movable + immovable), liabilities, pending criminal cases
- Cross-reference: are any of the declared businesses findable on MCA21?

### Task 7 — Adverse Media Timeline
- Pick any company that has been in the news for negative reasons
- Build a timeline of:
  - Regulatory actions (SEBI, RBI, FTC — whichever applies)
  - Court cases
  - News coverage
  - Financial changes (funding drops, layoffs)
- Identify: what was the first public signal of trouble?

---

## 13. Level 7 Summary

```
✅ MCA21 → CIN, directors (DIN), address, charges, filings (India)
✅ DIN lookup → all companies a director controls → network mapping
✅ GSTIN search → multi-state presence, all entities under same PAN
✅ OpenCorporates → 200+ jurisdictions, director networks globally
✅ UK Companies House → free full filings + PSC (beneficial ownership)
✅ SEC EDGAR → 10-K, 8-K, S-1 → tech stack, legal issues, properties
✅ Crunchbase → funding, investors, acquisitions → scope expansion
✅ BSE/NSE → quarterly results, shareholding, insider trades (listed India)
✅ eCourts + NCLT → pending cases, insolvency, judgments
✅ SEBI/RBI enforcement → regulatory violations
✅ Job postings → full tech stack, security tooling, internal project names
✅ GeM + CPPP → government contracts → client list and capabilities
✅ Patents → inventor names, technology roadmap, filing dates
✅ ICIJ Offshore Leaks → offshore entities, beneficial ownership
✅ OpenSanctions → aggregated sanctions and PEP lists
✅ myneta.info → declared assets of Indian political candidates
✅ Adverse media → Google News + GDELT → full negative coverage timeline

When you are done here → Proceed to Level 8: Active Reconnaissance
```

---

## 14. Tools & Resources Quick Reference

| Resource | Purpose | URL |
|---|---|---|
| **MCA21** | Indian company registry | mca.gov.in |
| **Zaubacorp** | MCA data + director network | zaubacorp.com |
| **Tofler** | Indian company financials | tofler.in |
| **GST Portal** | GSTIN lookup | gst.gov.in |
| **OpenCorporates** | 200+ jurisdiction registry | opencorporates.com |
| **Companies House** | UK registry (free filings) | companieshouse.gov.uk |
| **SEC EDGAR** | US public company filings | sec.gov/edgar |
| **CourtListener** | US federal court records | courtlistener.com |
| **eCourts** | India court records | ecourts.gov.in |
| **NCLT** | India company law tribunal | nclt.gov.in |
| **SEBI Orders** | India securities enforcement | sebi.gov.in |
| **Crunchbase** | Startup/VC intelligence | crunchbase.com |
| **BSE** | India listed company filings | bseindia.com |
| **NSE** | India listed company filings | nseindia.com |
| **GeM** | India government procurement | gem.gov.in |
| **USASpending** | US federal contracts | usaspending.gov |
| **GLEIF** | Global LEI + parent chain | gleif.org |
| **ICIJ Offshore Leaks** | Panama/Pandora Papers | offshoreleaks.icij.org |
| **OpenSanctions** | 150+ sanctions lists | opensanctions.org |
| **OFAC SDN** | US sanctions | sanctionssearch.ofac.treas.gov |
| **myneta.info** | India election affidavits | myneta.info |
| **GDELT** | Global news database | gdeltproject.org |
| **Google Patents** | Patent search | patents.google.com |
| **IP India** | India patents & trademarks | ipindia.gov.in |
| **LittleSis** | Corporate/political networks (US) | littlesis.org |
| **PeeringDB** | ISP/network peering data | peeringdb.com |
