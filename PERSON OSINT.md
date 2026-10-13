# OSINT — Level 2: Person OSINT
> Person OSINT is the process of building a complete profile of an individual using only
> publicly available information. This is used in security investigations, journalism,
> background verification, fraud detection, and missing persons cases.
> Everything here is passive — you observe, never interact.

---

## 1. The Person OSINT Methodology

Before running any tool, define what you are looking for. A person investigation
typically tries to answer one or more of these questions:

```
Who are they?         →  Real name, aliases, age, photo
Where are they?       →  City, address, workplace, frequent locations
How to reach them?    →  Email, phone, social handles
What do they do?      →  Job, employer, skills, interests
Who do they know?     →  Associates, family, colleagues
What have they done?  →  Posts, publications, court records, breach data
What do they own?     →  Domains, companies, vehicles, property
```

### 1.1 The Person OSINT Pyramid

Build from the widest, least personal layer inward:

```
        ▲
       / \
      /   \       Level 5: Physical (address, location, routine)
     /─────\
    /       \     Level 4: Private details (breach data, leaks, dark web)
   /─────────\
  /           \   Level 3: Deep social (posts, connections, behavior patterns)
 /─────────────\
/               \ Level 2: Accounts (social profiles, username, email, phone)
─────────────────
                  Level 1: Identity (name, photo, employer — public web)
```

Start at Level 1. Each level uses the previous level's findings as input.

### 1.2 Starting Points

You rarely start with everything. Common starting points:

| You Have | Where to Start |
|---|---|
| Full name | Google, LinkedIn, social search |
| Email address | Username extraction, breach check, GHunt |
| Username | Sherlock, WhatsMyName |
| Phone number | Truecaller, Phoneinfoga |
| Photo | Reverse image search (Yandex, Google Lens) |
| Company name | LinkedIn, Hunter.io, WHOIS |
| Domain / website | WHOIS, certificate lookup |

---

## 2. Username Enumeration

A username is often reused across many platforms. Finding it on one platform
opens access to all others where the same username exists.

### 2.1 Concept

```
Person creates an account → picks a username → reuses it across platforms
                                                       ↓
                                             Instagram: johndoe_1990
                                             Reddit: johndoe_1990
                                             GitHub: johndoe_1990
                                             Steam: johndoe_1990
                                             Forum: johndoe_1990 ← different info here
```

Each platform reveals different information about the same person.

### 2.2 Sherlock

The most widely used username search tool. Checks a username across 300+ platforms
and returns URLs where the account exists.

```bash
# Install
pip3 install sherlock-project

# Or clone from GitHub
git clone https://github.com/sherlock-project/sherlock.git
cd sherlock
pip3 install -r requirements.txt

# Basic search
python3 sherlock.py johndoe

# Search multiple usernames
python3 sherlock.py johndoe john.doe john_doe_1990

# Save results to a file
python3 sherlock.py johndoe --output results.txt

# Search only specific sites
python3 sherlock.py johndoe --site twitter --site github

# Timeout increase (useful for slow sites)
python3 sherlock.py johndoe --timeout 10

# Print only found accounts (suppress 404s)
python3 sherlock.py johndoe --print-found
```

**Reading the output:**

```
[+] Twitter:          https://twitter.com/johndoe          ← account EXISTS
[-] Facebook:         Not Found!                           ← no account
[!] Instagram:        Claimed! (timeout)                   ← uncertain
```

**What to do with results:**

- Visit every `[+]` link
- Screenshot the profile (public info, bio, profile picture, posts)
- Note: follower count, following count, join date, location, links in bio
- Archive each URL at archive.ph

### 2.3 Maigret

An advanced fork of Sherlock that provides more detail — it extracts info from
found profiles automatically and shows relationships between accounts.

```bash
# Install
pip3 install maigret

# Basic search
maigret johndoe

# Save HTML report
maigret johndoe --html

# Save PDF report
maigret johndoe --pdf

# Search on specific sites only
maigret johndoe --site twitter --site github

# Use Tor for anonymity
maigret johndoe --tor
```

Maigret generates a structured report with a visual graph of platform connections.

### 2.4 WhatsMyName

A community-maintained database of usernames and the patterns used to check them.

```
Web version:     whatsmyname.app
GitHub repo:     github.com/webbreacher/whats-my-name

# CLI usage
git clone https://github.com/webbreacher/whats-my-name.git
cd whats-my-name
python3 wmn-data.py -u johndoe
```

WhatsMyName is often more up to date than Sherlock because it is community-maintained.
Run both — they cover different platforms.

### 2.5 Namechk & KnowEm

```
Namechk:  namechk.com    → Checks username on 100+ social media + domains
KnowEm:   knowem.com     → Checks 500+ social networks + USPTO trademark search
```

These are web-based — no installation needed. Use them for quick cross-platform
checks when you do not need CLI output.

### 2.6 Username Analysis

When you find accounts, extract more usernames to search:

```
Example analysis of username "johndoe_1990":
├── "1990" → likely birth year → narrows age
├── "johndoe" → real name or alias
├── Variations to also search:
│   ├── johndoe1990
│   ├── john.doe.1990
│   ├── j.doe1990
│   ├── jdoe1990
│   └── john_doe
└── Check if username appears in email addresses via breach search
```

---

## 3. Email Address Investigation

An email address is one of the richest starting points in person OSINT.
It links to breach data, social accounts, GitHub, domain registrations, and more.

### 3.1 Extracting the Username from an Email

```
Email: john.doe1990@gmail.com
                ↓
Username hint:  john.doe1990  or  johndoe1990
                ↓
Search on: Sherlock, WhatsMyName
```

### 3.2 Email Format Discovery (When You Have a Name + Company)

If you know someone works at a company and you know the company's email format,
you can construct their email address.

**Hunter.io**
```
URL: hunter.io

Features:
  - Enter a domain → see the email format (firstname@, first.last@, etc.)
  - Shows actual employee emails found publicly
  - Verifies whether an email address is deliverable

Free tier: 25 searches/month
```

**Email Format Patterns:**
```
Most common formats:
  firstname@company.com               → john@acme.com
  firstname.lastname@company.com      → john.doe@acme.com
  f.lastname@company.com              → j.doe@acme.com
  flastname@company.com               → jdoe@acme.com
  firstname_lastname@company.com      → john_doe@acme.com
  lastname.firstname@company.com      → doe.john@acme.com
```

**Phonebook.cz**
```
URL: phonebook.cz
Search: Enter a domain → get a list of all emails and domains found publicly
No account required
```

**theHarvester** (covered in depth in Level 4 — Organisation OSINT):
```bash
theHarvester -d targetcompany.com -b google,linkedin,hunter
# Returns emails associated with the domain
```

### 3.3 Email Validation

Before acting on a guessed email, verify it exists.

```
Hunter.io email verifier:   hunter.io/email-verifier
Verify-email.org:           verify-email.org
MailTester:                 mail-tester.com
NeverBounce:                neverbounce.com (bulk verification, paid)
```

**How email verification works:**
```
Tool connects to the domain's mail server (MX record)
→ Simulates sending an email without actually sending it
→ Server responds whether the address exists
→ VALID   = address exists and accepts mail
→ INVALID = address does not exist
→ RISKY   = server cannot be verified (catch-all domains)
```

### 3.4 Breach & Leak Checking

Checking whether an email appears in a known data breach is one of the most
powerful OSINT moves — it may reveal usernames, passwords, IP addresses,
and other accounts associated with that email.

**Have I Been Pwned (HIBP)**
```
URL:      haveibeenpwned.com
Purpose:  Find which data breaches contain an email address
Free:     Yes — shows breach names, dates, and what data was exposed
API:      api.haveibeenpwned.com (free for personal use, key required)

What it shows:
  ✅ Which services were breached
  ✅ When the breach occurred
  ✅ What data types were exposed (email, password, phone, IP, etc.)
  ❌ Does NOT show actual passwords (they are hashed or redacted)
```

**DeHashed**
```
URL:      dehashed.com
Purpose:  Full breach data search — returns actual leaked data
Free:     Limited (shows entries but blurs values)
Paid:     Full access to leaked passwords, hashes, usernames, IP, phone

What it shows (paid):
  ✅ Plaintext passwords (when leaked as plaintext)
  ✅ Password hashes (MD5, SHA1, bcrypt)
  ✅ Associated usernames and IDs
  ✅ IP addresses at time of breach
  ✅ Phone numbers
```

**IntelligenceX (IntelX)**
```
URL:      intelx.io
Purpose:  Searches breaches, dark web, Tor, I2P, public paste sites
Free:     Limited searches per day
Paid:     Full access

Unique feature: Indexes dark web content, leaked databases, and historical web
```

**BreachDirectory**
```
URL:      breachdirectory.org
Purpose:  Free breach lookup; returns password hash for found entries
Free:     Yes — shows partial plaintext or hash
```

**Leakpeek**
```
URL:      leakpeek.com
Purpose:  Breach search with partial password reveal on free tier
```

**H8mail — Automated Breach Search**
```bash
# Install
pip3 install h8mail

# Basic search
h8mail -t target@email.com

# With API keys (more results)
# Configure ~/.h8mail_config.yml with keys from: hibp, intelx, dehashed

# Search multiple targets
h8mail -t target1@email.com target2@email.com

# Search from a file of emails
h8mail -t emails.txt

# Output to JSON
h8mail -t target@email.com -o results.json
```

### 3.5 Email → Google Account (GHunt)

If the target uses Gmail, GHunt extracts information from their Google account:
profile photo, Google Maps reviews, YouTube activity, Google Calendar events (if public),
and linked services.

```bash
# Install
pip3 install ghunt

# Authenticate (first time only)
ghunt login

# Investigate a Gmail address
ghunt email target@gmail.com

# What it returns (when data is public):
#   - Profile name and picture
#   - Google account ID
#   - Last profile edit date
#   - Google Maps reviews (with locations)
#   - YouTube channel (if linked)
#   - Google Photos albums (if public)
#   - Calendar events (if public)
#   - Linked app reviews (Play Store, etc.)
```

**Manual "Forgot Password" technique:**
```
1. Go to accounts.google.com/signin/recovery
2. Enter the target email
3. Google shows a partial phone number: "We'll send a code to +91 ••••••5678"
4. This confirms the email is valid AND reveals the last 4 digits of their phone
5. On Instagram/Facebook: the same technique reveals a partial email or phone

LEGAL NOTE: This is observation of publicly visible UI — not unauthorized access.
            Do not attempt actual account recovery. Observe only.
```

### 3.6 Reverse Email Search

Search the email address directly on Google and other platforms:

```
Google:   "john.doe1990@gmail.com"          → may find forum posts, registrations
Google:   site:linkedin.com "john.doe@"     → LinkedIn profile
GitHub:   Search the email in GitHub        → linked commits and accounts
Gravatar: en.gravatar.com/emails/?email=X   → profile picture linked to that email
```

---

## 4. Phone Number Investigation

### 4.1 What a Phone Number Can Reveal

```
Phone number
    │
    ├── Carrier (mobile, VoIP, landline)
    ├── Country and region
    ├── Owner's name (crowd-sourced apps like Truecaller)
    ├── Linked social accounts (via "forgot password")
    ├── Registration on messaging apps (WhatsApp, Telegram, Signal)
    └── Spam/fraud history
```

### 4.2 Truecaller

The most widely used reverse phone lookup globally, especially strong in India.

```
URL:    truecaller.com
App:    Available on Android and iOS
Web:    truecaller.com/search/in/<number>   (web lookup — limited)

What it shows:
  ✅ Name registered with the number (from crowd-sourced contacts)
  ✅ Operator / carrier
  ✅ Location (state / city)
  ✅ Spam score (if reported by users)

Limitations:
  - Requires an account to view full results
  - Name may be wrong if the person was saved differently by others
  - VoIP / international numbers have less data
```

### 4.3 Phoneinfoga

A framework specifically for phone number OSINT — queries multiple sources.

```bash
# Install (Docker recommended)
docker pull sundowndev/phoneinfoga
docker run -it sundowndev/phoneinfoga scan -n "+91XXXXXXXXXX"

# Or install from binary
# Download from: github.com/sundowndev/phoneinfoga/releases

# Scan a number
phoneinfoga scan -n "+919876543210"

# Start web UI
phoneinfoga serve -p 8080
# → Open browser: http://localhost:8080

# What it queries:
#   - Number format and carrier (via NumVerify API)
#   - Google search results for the number
#   - Truecaller (web scrape)
#   - HLR lookup (carrier + roaming status, if API configured)
```

**Configure API keys** in `~/.config/phoneinfoga/config.yaml`:
```yaml
numverify_api_key: "YOUR_KEY"
# Get free key at: numverify.com
```

### 4.4 NumVerify

```
URL:    numverify.com
API:    Free tier = 100 requests/month

Returns:
  ✅ Country and country code
  ✅ Carrier / network operator
  ✅ Line type (mobile, landline, VoIP)
  ✅ Valid / invalid status
  ✅ Location (country, city for some numbers)

curl "http://apilayer.net/api/validate?access_key=KEY&number=+919876543210"
```

### 4.5 Messaging App Presence

Check if a phone number is registered on messaging platforms:

```
WhatsApp:
  Add the number to your contacts (from your sock puppet phone/account)
  Open WhatsApp → check if the profile exists and has a picture / status
  Profile picture = account confirmed active
  "Last seen" info = usage pattern

Telegram:
  Search by phone number in Telegram
  If registered, their username and name appear
  telegram.me/<username> to view public profile

Signal:
  Add to contacts on Signal → it will show if they are on Signal
  No profile info is revealed beyond the fact they exist
```

### 4.6 India-Specific Resources

```
TRAI DND (Do Not Disturb) lookup:  trai.gov.in/dnd-check
  → Check if a number is registered on DND

Sanchar Saathi:  sancharsaathi.gov.in
  → Check how many SIMs are registered to an identity (govt portal)
  → Report unauthorized SIM cards

Truecaller India:  truecaller.com/search/in/<10-digit-number>

RTO vehicle lookup (from phone number → owner is indirect; usually VIN needed)
```

---

## 5. Social Media OSINT

### 5.1 Core Principles

```
1. NEVER interact — no likes, follows, comments, or DMs
2. ALWAYS use your OSINT browser profile (not your personal browser)
3. ALWAYS archive before the post gets deleted
4. Use a VPN — your IP may be logged when viewing some profiles
5. Notes: post time, location tags, tagged people, hashtags, device info in posts
```

### 5.2 What to Extract from Any Social Profile

```
From the profile page:
  ├── Username / handle
  ├── Display name (may differ from legal name)
  ├── Profile picture → run reverse image search
  ├── Bio / "About" → location, employer, website, phone, email
  ├── Join / creation date → context for the account
  ├── Follower / following count → reach and network size
  ├── Links → personal website, other platforms
  └── Location (if listed)

From posts:
  ├── Posting times → infer time zone and schedule
  ├── Recurring locations → home area, workplace, gym, frequent spots
  ├── Language patterns → writing style for comparison / attribution
  ├── Tagged people → known associates
  ├── Hashtags → communities and interests
  ├── Equipment and EXIF → camera model, sometimes GPS (if not stripped)
  └── Background details → vehicles, landmarks, license plates, signs
```

### 5.3 Twitter / X

**Advanced Search (without account):**
```
URL: twitter.com/search-advanced

Operators:
  from:username         → all tweets from this account
  to:username           → tweets mentioning this account
  @username             → mentions of this account
  "exact phrase"        → tweets containing this phrase
  since:YYYY-MM-DD      → tweets after this date
  until:YYYY-MM-DD      → tweets before this date
  near:"city"           → tweets posted near a location
  within:25km           → radius (used with near:)
  geocode:lat,long,25km → tweets within radius of coordinates
  filter:media          → only tweets with images/video
  filter:links          → only tweets with links
  -filter:retweets      → exclude retweets

Example:
  from:johndoe since:2024-01-01 until:2024-06-01 filter:media
  → All media tweets from johndoe in the first half of 2024
```

**Twint (Twitter scraping without API):**
```bash
# Install
pip3 install twint

# Scrape all tweets from a user
twint -u johndoe

# Scrape tweets from a date range
twint -u johndoe --since "2024-01-01" --until "2024-06-01"

# Scrape tweets containing a keyword
twint -u johndoe -s "company name"

# Save to CSV
twint -u johndoe -o results.csv --csv

# Search by location
twint -g "19.0760,72.8777,10km"     # tweets near Mumbai

# Save to JSON
twint -u johndoe -o results.json --json

# Note: Twitter/X has aggressively blocked scraping tools since 2023.
#       Results may be inconsistent. Use Twitter's own advanced search as primary.
```

**Useful third-party Twitter tools:**
```
Social Bearing:     socialbearing.com  → tweet analysis, posting time heatmap
Sleeping Time:      sleepingtime.io    → infer when the user is asleep from tweet times
TweetBeaver:        tweetbeaver.com    → follower/following analysis
```

### 5.4 Instagram

**Viewing without an account:**
```
Direct URL access (limited):
  https://www.instagram.com/<username>/
  → Public profiles are viewable without login
  → Private profiles show nothing

Picuki (picuki.com):
  → View and download public Instagram profiles without logging in
  → Search by username, hashtag, or location

StoriesIG (storiesig.com):
  → View public stories and highlights without an account

Imginn (imginn.com):
  → View and download public profile photos and videos
```

**Instaloader (bulk download and metadata):**
```bash
# Install
pip3 install instaloader

# Download public profile (no login needed)
instaloader profile johndoe

# What it downloads:
#   - All posts (images + captions)
#   - Post metadata (date, likes, comments count)
#   - Profile picture
#   - Followers / following lists (requires login for private)

# Download only metadata (no images)
instaloader --no-pictures --no-videos profile johndoe

# Download stories (requires login)
instaloader --login=your_account stories johndoe

# Download posts from a hashtag
instaloader "#osint"

# Download posts tagged at a location
instaloader "%location/12345678"
# Get location ID from the URL when browsing a location on Instagram
```

**Location tags:**
```
Every Instagram location tag points to a Facebook Place ID
  → clicking the location shows ALL public posts tagged there
  → search the location for other users who posted nearby
  → useful for building a physical location profile
  
URL pattern: instagram.com/explore/locations/<location_id>/
```

**What post metadata reveals:**
```
Caption:   mentions, hashtags, text content
Date/Time: posting time → infer time zone
Location:  GPS tag (not always present)
Tagged:    other accounts tagged in the post → network mapping
Comments:  associated accounts, relationship types
```

### 5.5 Facebook

**Searching without being logged in:**
```
Most Facebook content requires login. Without login:
  → Some public pages and group posts are viewable
  → Google indexes many Facebook posts:
    site:facebook.com "john doe" "company name"
    site:facebook.com "john.doe.1990"
```

**Finding Facebook ID:**
```
URL method:
  https://www.facebook.com/johndoe
  → In page source, search for "userID" → reveals numeric ID
  → OR use: findmyfbid.com (enter profile URL → returns ID)

  Numeric ID is permanent even if username changes
  → Use it for Graph API queries
```

**Who Posted What:**
```
URL:  whopostedwhat.com
Purpose: Facebook keyword search with date range (non-public posts excluded)
Usage:
  Enter a keyword + date range
  Returns public Facebook posts matching that keyword in that time
  Useful for finding posts about an event, location, or person
```

**Sowdust Graph Search:**
```
URL: sowdust.github.io/fb-search/
Purpose: Build custom Facebook Graph Search URLs
  → People who live in [city]
  → People who work at [company]
  → People named [name] who like [page]
```

### 5.6 LinkedIn

**Searching without login:**
```
Google search:
  site:linkedin.com/in/ "john doe" "company name"
  site:linkedin.com/in/ "john doe" "mumbai"

Direct URL:
  linkedin.com/in/johndoe
  → Some profile info is visible without login (name, title, company, location)
  → Full profile requires login
```

**What LinkedIn reveals:**
```
✅ Current employer and job title
✅ Job history (previous employers, dates)
✅ Education history
✅ Skills and endorsements
✅ Posts and activity (if public)
✅ Connections (mutual connections if logged in)
✅ Certifications and awards

Pivot uses:
  → Job history → which tech they use → look for exposed infra (Shodan)
  → Education → find classmates → network mapping
  → Employer → pivot to organisation OSINT (Level 4)
  → Skills → infer knowledge of specific systems
```

**CrossLinked (email harvesting from LinkedIn):**
```bash
# Install
pip3 install crosslinked

# Harvest emails from a company's LinkedIn employees
# Format: pattern of the company's email format
crosslinked -f "{first}.{last}@company.com" "Company Name"

# Common patterns to try:
crosslinked -f "{first}@company.com" "Company Name"
crosslinked -f "{f}{last}@company.com" "Company Name"

# Output goes to names.txt and emails.txt
```

### 5.7 Reddit

Reddit contains extremely candid personal disclosures — people often share
things on Reddit they would not post anywhere else, under usernames they believe
are anonymous.

**Searching deleted posts (critical):**
```
Official Reddit search only shows recent, non-deleted content.
To access ALL Reddit history including deleted posts:

Pushshift / PullPush:
  search.pullpush.io
  → Search deleted posts, comments, and threads
  → Filter by: subreddit, author, keyword, date range

URL pattern:
  https://api.pullpush.io/reddit/search/comment/?author=johndoe
  https://api.pullpush.io/reddit/search/submission/?author=johndoe

Unddit (unddit.com):
  → Paste a Reddit thread URL → see deleted comments in that thread
  
Reveddit (reveddit.com):
  → See all removed content by a specific user
  → reveddit.com/v/u/johndoe
```

**What Reddit history reveals:**
```
Subreddits a user posts in → interests, hobbies, problems they are dealing with
Post times → time zone inference
Mentioned locations → city, neighborhood, local landmarks
Writing style → consistent across platforms for attribution
Personal disclosures → health, family, employment, relationships
Username → may match usernames on other platforms
```

### 5.8 TikTok

```
Direct URL:   tiktok.com/@johndoe

What to look for:
  ├── Background details in videos → location, landmarks, license plates
  ├── Location tags on videos
  ├── Other accounts tagged
  ├── Bio links → other platforms
  ├── Audio → original audio? → same audio elsewhere?
  └── Comments → associated accounts

Tools:
  savett.cc / snaptik.app → download TikTok videos for offline analysis
  
Metadata extraction:
  Download video → run ExifTool → may reveal device model and sometimes location
  exiftool video.mp4
```

### 5.9 Cross-Platform Correlation

This is where person OSINT becomes powerful — linking accounts across platforms.

```
Technique 1: Profile picture matching
  Download the profile picture from Platform A
  Run reverse image search (Yandex, Google Lens)
  → Finds same picture on other platforms

Technique 2: Bio text matching
  Copy a distinctive phrase from the bio
  Search it in Google with quotes: "available for freelance and collabs 📸"
  → Finds other profiles using the exact same bio text

Technique 3: Writing style (stylometry)
  Consistent: capitalization, punctuation, emoji use, phrases, typos
  → Used to attribute anonymous posts to a known person
  Tools: JGAAP (Java Graphical Authorship Attribution Program)

Technique 4: Posting time patterns
  Collect 50+ posts with timestamps
  Plot on a heatmap → identify active hours
  Active hours → infer time zone (compare against stated location)

Technique 5: Username variations
  From johndoe_1990 on Instagram → search johndoe1990, johndoe90, john.doe on other platforms

Technique 6: Mutual connections
  Platform A: account follows X, Y, Z
  Platform B: account follows X, Y, Z (same subset)
  → Same person with high probability
```

---

## 6. People Search Engines

These databases aggregate public records — voter rolls, property records, court
records, phone directories, and social data — into searchable profiles.

### 6.1 Global / US-Focused

| Tool | URL | Notes |
|---|---|---|
| **Spokeo** | spokeo.com | US people search — address, relatives, social |
| **Pipl** | pipl.com | Deep people search — paid; very thorough |
| **BeenVerified** | beenverified.com | US records — address history, relatives |
| **Intelius** | intelius.com | US people search — court records, background |
| **TruePeopleSearch** | truepeoplesearch.com | US, completely free |
| **FastPeopleSearch** | fastpeoplesearch.com | US, free |
| **Radaris** | radaris.com | US + some international |
| **ZabaSearch** | zabasearch.com | US public records |
| **Whitepages** | whitepages.com | US phone, address, background |

### 6.2 India-Focused Resources

```
Truecaller:         truecaller.com         → best for phone → name lookup in India
LinkedIn:           linkedin.com           → professional profiles
MCA21:              mca.gov.in             → company directors (if target is a director)
eCourts:            ecourts.gov.in         → court case lookup by name
RTO:                parivahan.gov.in       → vehicle registration (VIN / plate lookup)
Voter ID search:    voters.eci.gov.in      → voter registration (name, address, photo)
```

### 6.3 How to Use People Search Engines

```
Step 1: Enter the full name + state/city if known
Step 2: Review all returned profiles — there may be multiple people with the same name
Step 3: Cross-reference with known info (age, location, relatives) to identify correct match
Step 4: Extract: current address, previous addresses, phone, relatives, associated emails
Step 5: Each piece of new info → new pivot (relatives' names → search them → extended network)

Note: Most meaningful data requires a paid subscription.
      Free tiers usually show that a profile exists but blur the details.
```

---

## 7. Breach Data Analysis

### 7.1 The Value of Breach Data

When a company's database is hacked, the leaked data often contains:
- Email addresses
- Usernames
- Password hashes or plaintext passwords
- Phone numbers
- IP addresses (at time of registration)
- Physical addresses
- Purchase history

This data, when combined with other OSINT, builds a rich profile.

### 7.2 Workflow

```
Step 1: Find breach entries for target email / username
  → HIBP, DeHashed, IntelX, H8mail

Step 2: Analyze what was exposed
  → Email: confirms the account exists on that service
  → Username: new username to search across platforms
  → Password (if plaintext): reveals their password habits / patterns
  → IP: may reveal ISP, approximate location at time of breach
  → Phone: new number to investigate

Step 3: Password pattern analysis (DO NOT TEST CREDENTIALS — observation only)
  Plaintext password: "Johndoe@Mumbai2020"
  Pattern analysis:
    ├── Uses real name → confirmed alias = real name
    ├── Uses city → location confirmed or hinted
    ├── Uses year → may update year annually (→ current password: Johndoe@Mumbai2026?)
    └── Pattern → apply to other services for credential stuffing analysis
                  (LEGAL NOTE: Testing = unauthorized access. Analysis only.)

Step 4: Service name from breach → new pivot
  Breached on "ClothingStore2019" → they shopped there → address may be in that data
  Breached on "GamingForum2021" → they game → look for gaming platform accounts
```

### 7.3 Password Habit Analysis (Legal / Passive)

```
If you find a plaintext password from a public breach:
  DO:
    ✅ Analyze the pattern (structure, format, personal info embedded)
    ✅ Use it to learn their naming conventions (= useful for username guessing)
    ✅ Document it as a finding
  DO NOT:
    ❌ Use the password to attempt to log into any account
    ❌ Share the password
    ❌ Store breach databases (receiving stolen data may be illegal)
```

---

## 8. Dark Web Mentions

### 8.1 What to Look For

```
On dark web forums and markets, personal data appears in:
  ├── Data dump postings (email, password, SSN, credit card)
  ├── Forum discussions mentioning the person
  ├── Listings (fraudulent services, identity documents for sale)
  └── Doxing posts (if target has been doxed by someone else)
```

### 8.2 Tools (Accessible Without Tor)

**Ahmia**
```
URL:     ahmia.fi
Purpose: Tor search engine — accessible without Tor
Usage:   Search for the target's name, email, or username
         Returns publicly indexed .onion pages mentioning the term
```

**IntelligenceX**
```
URL:     intelx.io
Purpose: Indexes dark web, Tor, I2P, leaked databases, and paste sites
Free:    Limited daily searches
Paid:    Full access with more results and historical data
Usage:   Search email, name, domain → returns any dark web mentions
```

**DarkSearch**
```
URL:     darksearch.io
Purpose: Dark web search engine, no Tor needed
Free:    Yes — API available
```

**OnionSearch**
```bash
# Install
pip3 install onionsearch

# Search multiple dark web search engines simultaneously
onionsearch "john doe 1990"
```

### 8.3 Safety Note

```
The dark web contains illegal content. When conducting searches:
  ✅ Use the surface-web tools listed above (Ahmia, IntelX) — no Tor needed for initial sweep
  ✅ If you must use Tor: use Tails OS + Tor Browser; never reveal identity
  ❌ Do NOT download files from dark web during OSINT
  ❌ Do NOT engage with any marketplace or forum
  ❌ Do NOT access CSAM, weapons, or drug content (criminal offense)
  
   For most person OSINT, IntelX and HIBP cover dark web exposure adequately.
   Full Tor investigation is rarely needed and carries significant risk.
```

---

## 9. Building the Profile — Putting It Together

After running through all the above steps, consolidate your findings:

### 9.1 The Person Profile Template

```markdown
## Person OSINT Report

### Subject Overview
- Full Name:         [confirmed / likely / unknown]
- Aliases / Usernames:
- Age / DOB:
- Gender:
- Current Location:
- Previous Locations:
- Languages:

### Contact
- Email addresses (list all found + source)
- Phone numbers (list all found + source)
- Physical address (list all found + source)

### Employment
- Current employer:
- Previous employers:
- Job title / role:
- Tech stack / skills:

### Online Presence
| Platform | Username | URL | Key findings |
|---|---|---|---|
| Twitter | | | |
| Instagram | | | |
| LinkedIn | | | |
| Reddit | | | |
| GitHub | | | |

### Breach Data
| Service | Year | Data Exposed |
|---|---|---|
| | | |

### Associates / Network
- Known connections:

### Timeline of Key Events
- [Date] → [Finding and source]

### Evidence
- [Screenshot filename] → [What it shows] → [Archived URL]

### Gaps & Open Questions
- What is still unknown?
- What pivot leads remain unexplored?

### Sources
- List all URLs with dates accessed
```

### 9.2 Visualization in Maltego

```
Create entities for each data point:
  Person entity → email → username → social profiles → phone → address

Run transforms on each:
  Email entity → "Find breaches" → "Find associated usernames"
  Domain entity → "Find subdomains" → "Find employees"

The graph shows relationships visually — useful for presenting findings
and identifying connections you missed in text analysis
```

---

## 10. Level 2 Practice Tasks

### Task 1 — Username Hunt (Public Figure)
- Pick a public figure (journalist, YouTuber, public author)
- Run Sherlock and WhatsMyName against their known username
- Find at least 3 platforms they use that are not their primary/public ones
- Document what extra information those lesser-known profiles reveal

### Task 2 — Email Investigation
- Use Hunter.io to find the email format for a large company (Google, Infosys, etc.)
- Construct 5 likely email addresses for named employees using that format
- Verify one using an email validation tool
- Check one in HIBP to see if it appears in any breach

### Task 3 — Social Media Profile Analysis
- Pick any public Instagram or Twitter profile
- Extract all data listed in Section 5.2 without interacting with the account
- Identify: likely time zone, approximate location, 3 associates, primary interests
- Run reverse image search on their profile picture

### Task 4 — Reddit History
- Pick a Reddit username from a public thread
- Use pullpush.io to find their complete post history
- Find a post that reveals personal information they may not have intended to share
- Note: which subreddits they post in, time zone, interests

### Task 5 — Cross-Platform Correlation
- Starting from one social profile, chain to at least 3 other platforms
- Document each pivot: what data on Platform A led you to Platform B
- Build a simple relationship diagram in Obsidian or draw it by hand

---

## 11. Level 2 Summary

```
✅ Start with what you have — every starting point has a pivot path
✅ Usernames: Sherlock + WhatsMyName + manual variations
✅ Emails: Hunter.io → breach check (HIBP, DeHashed) → GHunt (Gmail)
✅ Phone: Truecaller + Phoneinfoga + messaging app presence
✅ Social: extract everything from profiles, NEVER interact
✅ Reddit: pullpush.io for full history including deleted posts
✅ Breach data: analyze patterns, never test credentials
✅ Cross-platform: same photo / bio / username → link accounts
✅ Document everything with source, timestamp, and archive link

When you are done here → Proceed to Level 3: Image & Media OSINT
```

---

## 12. Tools Quick Reference

| Tool | Purpose | Install | Command |
|---|---|---|---|
| **Sherlock** | Username search (300+ sites) | `pip3 install sherlock-project` | `sherlock johndoe` |
| **Maigret** | Advanced username + report | `pip3 install maigret` | `maigret johndoe --html` |
| **WhatsMyName** | Username search (community DB) | `git clone` repo | `python3 wmn-data.py -u johndoe` |
| **GHunt** | Gmail account OSINT | `pip3 install ghunt` | `ghunt email x@gmail.com` |
| **H8mail** | Breach email search | `pip3 install h8mail` | `h8mail -t x@email.com` |
| **Phoneinfoga** | Phone number OSINT | Docker / binary | `phoneinfoga scan -n +91XXX` |
| **Instaloader** | Instagram download | `pip3 install instaloader` | `instaloader profile johndoe` |
| **CrossLinked** | LinkedIn email harvest | `pip3 install crosslinked` | `crosslinked -f "{f}{last}@co.com" "Co"` |
| **Twint** | Twitter scraping | `pip3 install twint` | `twint -u johndoe` |
| **HIBP** | Email breach check | Web | haveibeenpwned.com |
| **DeHashed** | Full breach data | Web (paid) | dehashed.com |
| **IntelX** | Dark web + breach search | Web | intelx.io |
| **Truecaller** | Phone → name (India) | Web / App | truecaller.com |
| **Hunter.io** | Email format / discovery | Web | hunter.io |
| **pullpush.io** | Reddit full history | Web | search.pullpush.io |
| **Ahmia** | Dark web search (no Tor) | Web | ahmia.fi |
