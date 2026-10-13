# OSINT — Level 5: Geospatial Intelligence (GeoINT)
> GeoINT is the discipline of extracting location intelligence from
> images, videos, social media, signals, and open data. It answers
> the question: WHERE? — where a photo was taken, where a person
> lives or works, where an event occurred, and where infrastructure
> is physically located. GeoINT is the backbone of investigative
> journalism, conflict verification, and missing persons work.

---

## 1. What is GeoINT?

### 1.1 Definition

**Geospatial Intelligence (GeoINT)** is the collection and analysis of imagery,
signals, and data tied to physical locations on Earth. In OSINT, GeoINT means:

```
→  Locating where a photo or video was taken
→  Verifying the location of an event
→  Mapping the physical footprint of a person or organization
→  Tracking movement patterns from social media and signals
→  Identifying physical infrastructure from satellite imagery
→  Confirming or debunking location claims in media
```

### 1.2 GeoINT in Practice

```
Investigative Journalism:
  Bellingcat geolocated MH17 missile launch site using social media images
  and satellite imagery — purely from publicly available sources.

Conflict Verification:
  War footage verified (or debunked) by matching landmarks, sun angles,
  vehicle types, and terrain with satellite imagery.

Missing Persons:
  Last known location from social post → narrow search area →
  Street View matching → physical search team deployment.

Infrastructure OSINT:
  Data center location confirmed from satellite imagery and
  power infrastructure visible in delivery driver photos.

Security Research:
  Employee's home location inferred from consistent background
  details across multiple video calls and social media posts.
```

### 1.3 The GeoINT Process

```
┌──────────────────────────────────────────────────────────┐
│                    GEOINTO PROCESS                        │
│                                                          │
│  SOURCE          →  EXTRACT       →  ANALYZE  →  LOCATE │
│                                                          │
│  Image/Video        EXIF GPS          Landmarks          │
│  Social Post        Sun angle         Vegetation         │
│  Wi-Fi SSID         Shadows           Architecture       │
│  IP Address         License plate     Signs & text       │
│  Signal data        Vegetation        Satellite match    │
│                     Sound clues       Street View match  │
│                                                          │
│                              ↓                           │
│                       COORDINATES                        │
│                    + TIMESTAMP + DIRECTION               │
└──────────────────────────────────────────────────────────┘
```

---

## 2. Mapping & Satellite Tools

### 2.1 Google Maps

The primary tool for any GeoINT investigation.

```
URL: maps.google.com

Key features for GeoINT:
  ├── Satellite view         →  overhead imagery for landmark matching
  ├── Street View            →  ground-level imagery (360°)
  ├── Historical Street View →  past images (useful for old photos)
  ├── Indoor maps            →  airports, malls, stadiums
  ├── Terrain view           →  elevation, topography
  └── Measure distance       →  right-click → Measure distance

Street View tips:
  → Press left/right arrows to navigate along a road
  → Drag the orange figure to ANY blue-highlighted road
  → Scroll the mini-map in the corner to keep track of location
  → Click the clock icon (top-left in Street View) → historical imagery
  → Press Shift+Click → add a waypoint; useful to measure angles

Keyboard shortcuts in Street View:
  Arrow keys    →  move forward/backward/turn
  Page Up/Down  →  zoom in/out
  + / -         →  zoom
```

### 2.2 Google Earth Pro

More powerful than Google Maps for GeoINT — has historical satellite imagery
going back to the 1980s in some areas, and measurement tools.

```
Download: earth.google.com/web/ (browser) or Pro desktop (free)

Key features:
  ├── Historical imagery slider    →  see how a location looked in any year
  ├── Sun position tool            →  simulate sun angle at any date/time
  ├── Measurement tools            →  distance, area, elevation profile
  ├── KML/KMZ import               →  overlay custom data layers
  ├── Flight simulator             →  useful for perspective matching
  └── 3D buildings                 →  match 3D structure from an image

Historical imagery workflow:
  1. Navigate to location
  2. Click the clock icon in the toolbar → slider appears
  3. Drag slider left → older imagery
  4. Compare building construction stages, vegetation growth, infrastructure

Sun position in Google Earth Pro:
  1. Tools → Sun → Show Sun
  2. View → Sun → adjust date and time
  3. Sun casts real shadows at the correct angle for that date/time/location
  → Match shadow angle from your subject image to confirm time/date
```

### 2.3 Bing Maps

```
URL: bing.com/maps

Why use it alongside Google:
  ├── Different satellite image sources → sometimes more recent
  ├── Better oblique aerial imagery (bird's-eye view at 45°)
  ├── Bird's eye view available in many cities → see building sides
  └── Sometimes higher resolution than Google in certain regions

Bird's eye view (45° oblique):
  → Switch to "Bird's eye" in Bing Maps
  → Rotate N/S/E/W to see all four sides of a building
  → Useful for matching side-on building features from a photo
```

### 2.4 Sentinel Hub — Free Satellite Imagery

```
URL: sentinel-hub.com/explore/eobrowser/

What it is:
  ESA (European Space Agency) Sentinel satellites provide free
  multispectral imagery of the entire Earth updated every 5 days.

Account: Free (registration required)

Key uses in GeoINT:
  ├── Track changes over time (construction, deforestation, flooding)
  ├── Identify infrastructure not visible in Google (recent builds)
  ├── Agricultural analysis (NDVI bands show crop health)
  ├── Wildfire / flood mapping
  └── Ship tracking (SAR imagery mode)

EO Browser usage:
  1. Open EO Browser → search for a location
  2. Select: Sentinel-2 L2A (optical, 10m resolution)
  3. Adjust date range → select clear (cloud-free) image
  4. Compare: click the "Compare" split screen to see two dates side by side
  5. Download the image for annotation

Resolution reference:
  Sentinel-2:   10 metre resolution (free, 5-day revisit)
  Planet Labs:  3 metre resolution (commercial, daily)
  Maxar:        30 cm resolution (commercial, used by intelligence agencies)
  Google Maps:  varies; major cities often at 15-50 cm
```

### 2.5 Mapillary & KartaView — Crowd-Sourced Street Images

```
Mapillary:  mapillary.com
KartaView:  kartaview.org

What they are:
  Crowd-sourced street-level imagery databases. Contributors upload
  photos/videos taken while walking, driving, or cycling. Coverage
  extends to areas Google Street View has never visited.

GeoINT uses:
  ├── Match images from areas with no Google Street View
  ├── Find imagery from less-developed regions
  ├── Find recent imagery (uploaded days or weeks ago)
  └── Match imagery from rural roads, trails, and paths

Usage:
  1. Navigate to the area on the map
  2. Green lines = coverage available
  3. Click any green line → view images along that path
  4. Filter by date for most recent images
```

### 2.6 OpenStreetMap & Overpass Turbo

```
OpenStreetMap (OSM): openstreetmap.org
Overpass Turbo:      overpass-turbo.eu

What OSM is:
  The open-source, community-maintained world map.
  Contains detailed data on roads, buildings, amenities,
  natural features, and infrastructure — all queryable.

Overpass Turbo — querying OSM data:
  Run structured queries to find specific features in an area.

Example queries:

  # Find all McDonald's in Mumbai
  [out:json][timeout:25];
  area["name"="Mumbai"]->.searchArea;
  node["name"="McDonald's"](area.searchArea);
  out body;

  # Find all mosques in a city
  [out:json][timeout:25];
  area["name"="Delhi"]->.searchArea;
  node["amenity"="place_of_worship"]["religion"="muslim"](area.searchArea);
  out body;

  # Find all ATMs of a specific bank
  [out:json][timeout:25];
  area["name"="Bangalore"]->.searchArea;
  node["amenity"="atm"]["operator"="SBI"](area.searchArea);
  out body;

  # Find all railway stations
  node["railway"="station"](19.0,72.8,19.2,72.9);
  out body;
  # (bounding box: south,west,north,east)

GeoINT application:
  Image shows: a distinctive orange-coloured petrol station
  → Overpass query: nodes with "brand"="HP" or "Indian Oil" in the area
  → Narrows the location to specific fuel stations
  → Cross-reference with satellite imagery
```

---

## 3. IP Geolocation

### 3.1 What IP Geolocation Can and Cannot Do

```
CAN DO:
  ✅ Identify the country with ~99% accuracy
  ✅ Identify the city with ~70-80% accuracy (major ISPs)
  ✅ Identify the ISP / organization
  ✅ Identify the ASN
  ✅ Identify if IP is a VPN / proxy / Tor exit node
  ✅ Identify if IP is a data center or residential

CANNOT DO:
  ❌ Give a precise street address (for residential IPs)
  ❌ Work accurately through VPN / Tor (shows VPN server location)
  ❌ Identify mobile users accurately (carrier NAT masks real location)
  ❌ Work for satellite internet (Starlink shows random US locations)
```

### 3.2 IP Geolocation Tools

**ip-api.com (free, no account)**
```bash
# Single IP lookup
curl http://ip-api.com/json/93.184.216.34

# Returns JSON:
{
  "status": "success",
  "country": "United States",
  "countryCode": "US",
  "region": "MA",
  "regionName": "Massachusetts",
  "city": "Norwell",
  "zip": "02061",
  "lat": 42.1596,
  "lon": -70.8217,
  "timezone": "America/New_York",
  "isp": "Edgecast Inc.",
  "org": "MCI Communications Services d/b/a Verizon Business",
  "as": "AS15133 MCI Communications Services d/b/a Verizon Business",
  "query": "93.184.216.34"
}

# Batch lookup (up to 100 IPs)
curl -X POST http://ip-api.com/batch \
  -H "Content-Type: application/json" \
  -d '["93.184.216.34", "8.8.8.8", "1.1.1.1"]'
```

**ipinfo.io**
```bash
# Full info
curl https://ipinfo.io/93.184.216.34

# Just the org
curl https://ipinfo.io/93.184.216.34/org

# Just the city
curl https://ipinfo.io/93.184.216.34/city

# Check if it's a VPN/proxy/hosting
curl "https://ipinfo.io/93.184.216.34?token=YOUR_TOKEN"
# Returns "privacy" object if it's a VPN/datacenter
```

**Shodan for IP intelligence**
```bash
shodan host 93.184.216.34
# Returns: org, ISP, open ports, banners, geo, hostnames
```

**ipqualityscore.com**
```bash
# Detects: VPN, proxy, Tor, bot, abuse history, fraud score
curl "https://ipqualityscore.com/api/json/ip/YOUR_KEY/93.184.216.34"
```

### 3.3 Geolocation from Email Headers

When someone sends you an email, the email headers may contain their
sending IP address — which can be geolocated.

```
How to extract email headers:
  Gmail:    Open email → three dots → "Show original"
  Outlook:  File → Properties → Internet headers
  Apple Mail: View → Message → All Headers

What to look for in headers:
  Received: from mail.example.com (93.184.216.34) by ...

Extract IPs from "Received:" headers:
  → The LAST "Received:" header (bottom) = originating server
  → IPs in earlier "Received:" headers = mail relay hops
  → Ignore IPs belonging to Google/Microsoft (they are mail servers)
  → Geolocate the ORIGINATING IP

Tools for header analysis:
  MXToolbox Header Analyzer:   mxtoolbox.com/EmailHeaders.aspx
  Mail Header Analyzer:        mailheader.org
  Google Admin Toolbox:        toolbox.googleapps.com/apps/messageheader/
```

---

## 4. Wi-Fi & Signal Intelligence

### 4.1 WiGLE — Wi-Fi Network Geolocation

```
URL: wigle.net

What it is:
  A massive crowd-sourced database of Wi-Fi networks (SSIDs) and their
  approximate physical locations, collected by volunteers (wardriving).
  Contains over 1 billion networks worldwide.

GeoINT use cases:
  ├── An SSID name appears in a photo or social post → look it up → location
  ├── A BSSID (MAC address) from a photo → exact access point location
  ├── Home network named after a person → pivot to address
  └── Confirm that a person is in a specific location from their Wi-Fi

Free account: allows basic map browsing and limited searches
Paid account: full API access, bulk queries

Web search:
  1. Create account at wigle.net
  2. Go to: wigle.net/search
  3. Search by:
     - SSID (network name): e.g., "JohnDoeHomeNetwork"
     - BSSID (MAC address): e.g., "AA:BB:CC:DD:EE:FF"
  4. Results show: approximate coordinates, last seen date, frequency

API search:
curl -u "username:password" \
  "https://api.wigle.net/api/v2/network/search?ssid=JohnDoeHomeNetwork"
```

**Real-world example:**
```
A photo shows a laptop with a Wi-Fi popup visible: "SSID: Smith_Family_Home"
→ Search WiGLE: "Smith_Family_Home"
→ Match found at: 40.7128° N, 74.0060° W (approximate)
→ Narrows location to a specific block in New York City
→ Cross-reference with known information about the subject
```

### 4.2 Bluetooth & Device Signals

```
Bluetooth device names sometimes appear in screenshots:
  Phone screenshot → Bluetooth menu visible → device names shown
  "John's iPhone", "BMW 5 Series", "HP LaserJet P1006"
  → Device names can confirm identity or organization

Bluetooth MAC address databases:
  Not as comprehensive as WiGLE for Wi-Fi
  OpenCelliD has some Bluetooth data

Practical use:
  Corporate equipment often has names like "COMPANY-LAPTOP-001"
  Visible in screenshots → confirms organization
```

### 4.3 Cell Tower Geolocation

```
OpenCelliD:   opencellid.org
  The largest open database of cell tower locations.
  Contains: MCC, MNC, LAC, CellID → GPS coordinates

When it is useful:
  Old Android screenshots show "Cell ID" in developer options
  Some apps expose cell tower data
  Malware / spyware analysis reveals tower IDs

API example:
curl "https://opencellid.org/cell/get?key=YOUR_KEY&mcc=404&mnc=20&lac=2183&cellid=11174"
# Returns: lat, lon, range (accuracy)

Providers in India (MCC=404, MNC=XX):
  MNC 20 = Vodafone India
  MNC 10 = Airtel
  MNC 45 = Aircel
  MNC 01 = BSNL
```

---

## 5. Flight & Maritime Tracking

### 5.1 Flight Tracking

Civilian aircraft are required to broadcast their position via ADS-B
(Automatic Dependent Surveillance–Broadcast). This data is received by
ground stations and aggregated into public databases.

**FlightAware**
```
URL: flightaware.com

Features:
  ├── Real-time flight position tracking
  ├── Flight history (where has this aircraft been?)
  ├── Aircraft registration lookup (tail number → owner)
  ├── Airport activity (arrivals/departures)
  └── Route and speed data

Registration lookup:
  flightaware.com/resources/registration/N12345
  → Returns: aircraft type, registered owner, registration date

GeoINT use:
  Photo shows private jet tail number (e.g., N12345)
  → FlightAware → owner → associated person or company
  → Flight history → where have they been and when
  → Confirms presence in a location at a specific time
```

**FlightRadar24**
```
URL: flightradar24.com

Best for:
  ├── Live global flight map
  ├── Playback of historical flights (go back in time)
  ├── Military and private aircraft (limited)
  └── Filtering by airline, type, altitude, speed

Playback workflow:
  1. Go to flightradar24.com
  2. Click the clock icon (top menu)
  3. Set date and time
  4. Press play → see all flights that were active at that time
  5. Click any aircraft → see its exact route
```

**ADS-B Exchange**
```
URL: adsbexchange.com

Why it is unique:
  ├── Does NOT filter out military aircraft
  ├── Does NOT honor "privacy" removal requests
  ├── Shows aircraft that choose to be hidden from FlightAware/FR24
  └── Full historical data via API

Used by:
  Investigators tracking private jets, military movements,
  government aircraft, and VIP travel
```

**OpenSky Network**
```
URL: opensky-network.org

Free API:
  curl "https://opensky-network.org/api/states/all?lamin=18.5&lomin=72.7&lamax=19.3&lomax=73.0"
  # Returns all aircraft currently over Mumbai

Historical data:
  python3:
  from pyopensky.rest import OpenSky
  opensky = OpenSky()
  flights = opensky.get_flights_by_aircraft("N12345", begin=1672531200, end=1672617600)
```

### 5.2 Maritime Tracking

Ships transmit their position via AIS (Automatic Identification System).

**MarineTraffic**
```
URL: marinetraffic.com

Features:
  ├── Real-time ship positions globally
  ├── Vessel history (past routes)
  ├── Port activity (arrivals and departures)
  ├── Vessel details (flag, type, owner, dimensions)
  └── Photo gallery of vessels

Vessel lookup:
  Search by: name, MMSI (unique ship ID), IMO number, call sign
  → View: current position, recent route, port calls, owner

GeoINT use:
  News photo shows a ship → read the name on the hull
  → MarineTraffic → current or last known position
  → History → where has it been
```

**VesselFinder**
```
URL: vesselfinder.com
Alternative to MarineTraffic — sometimes has different coverage
```

**Global Fishing Watch**
```
URL: globalfishingwatch.org
Tracks commercial fishing vessels — useful for illegal fishing investigation
```

### 5.3 Vehicle Tracking

```
India vehicle registration lookup:
  URL: vahan.nic.in/vahanservice/
  Enter: vehicle registration number (plate)
  Returns: make, model, registration date, owner name (in some cases), state

  Alternative: parivahan.gov.in/rcdlstatus/

International plate format identification:
  platesmania.com → identify country/region from plate format
  Wikipedia: "Vehicle registration plates by country"
  → Compare plate format in image to known formats

License plate reading in images:
  If plate is partially visible → enhance with image tools:
    GIMP → Colors → Curves → boost contrast
    Forensically (29a.ch) → magnifier tool
    Let's Enhance (letsenhance.io) → AI upscaling (web)
    Topaz Gigapixel AI → AI upscaling (desktop, paid)
```

---

## 6. Sun, Shadow & Time Analysis

### 6.1 SunCalc — Detailed Usage

```
URL: suncalc.org

Interface elements:
  ├── Yellow line    →  sun's path through the day
  ├── Orange marker  →  sun's current position
  ├── Time slider    →  drag to change time
  ├── Date picker    →  change date
  └── Azimuth/elevation display → exact sun angle

How to use for GeoINT:

  Step 1: Set approximate location on the map
          (from context clues — city or region level)

  Step 2: Set the date
          (from EXIF, event context, or seasonal clues like vegetation)

  Step 3: Look at shadows in your image
          → Note the DIRECTION they point
          → Note the LENGTH relative to the object casting them

  Step 4: Drag the time slider on SunCalc
          → Match when the shadow direction on the map
            matches the shadow direction in your image

  Step 5: The time shown = approximate time the photo was taken

  Step 6: Refine the location by testing different spots
          → The sun angle changes with latitude, not longitude
          → Shadow direction helps confirm N/S hemisphere
          → Shadow length helps confirm latitude range
```

### 6.2 Shadow Calculator

```
URL: shadowcalculator.eu

More precise than SunCalc for shadow length analysis.

Input:
  ├── Location (lat/lon)
  ├── Date
  ├── Time
  └── Object height

Output:
  ├── Shadow length
  ├── Shadow azimuth (direction)
  └── Sun elevation and azimuth

Usage:
  Measure a shadow in the image relative to a known object
  (a person of average height, a car of known length, a building)
  → Input object height → target shadow length
  → Solver gives you the date/time/location combination that produces it
```

### 6.3 Deriving Direction from Shadows

```
Northern Hemisphere:
  Shadow points NORTH  →  Photo taken at solar noon (sun in south)
  Shadow points NE     →  Photo taken in afternoon (sun in SW)
  Shadow points NW     →  Photo taken in morning (sun in SE)
  Long shadow          →  Early morning or late afternoon
  Short shadow         →  Around midday

Southern Hemisphere (reverse):
  Shadow points SOUTH  →  Photo taken at solar noon (sun in north)

Verification:
  If shadow in a street photo points north along the road
  → Road runs E-W
  → Cross-reference with satellite map: does the road run E-W?
  → If yes → location is consistent
  → If road runs N-S → shadow direction is wrong → revisit assumptions
```

---

## 7. Social Media Geolocation

### 7.1 Geotagged Posts

```
Many social media posts contain embedded location data.

Instagram:
  → Click the location tag above the photo
  → Shows all public posts tagged at that location
  → Build a timeline of activity at a specific place
  → URL: instagram.com/explore/locations/<location_id>/

Twitter / X:
  → Advanced search: near:"city name" within:25km
  → geocode:lat,lon,radius (e.g., geocode:19.0760,72.8777,5km)
  → Returns tweets posted near those coordinates

Snapchat Map:
  → web.snapchat.com/map
  → Shows public "Snaps" posted in a geographic area in real time
  → Useful for event verification and crowd intelligence

Facebook:
  → Search: "posts near [city]"
  → Check-ins at a specific location

Flickr:
  → Flickr has a geolocation map of public photos
  → flickr.com/map → browse geotagged photos by area
  → Search for photos tagged at a specific location: flickr.com/places
```

### 7.2 Twitter Geosearch Automation

```bash
# Using Twint for location-based search
twint -g "19.0760,72.8777,5km" -o mumbai_tweets.json --json

# With a keyword
twint -g "19.0760,72.8777,5km" -s "explosion" --since "2024-01-01"

# Results include: username, tweet content, timestamp, coordinates (if shared)
```

### 7.3 Reverse Geocoding

Converting GPS coordinates (from EXIF) into a human-readable address.

```bash
# Using OpenStreetMap Nominatim (free, no key required)
curl "https://nominatim.openstreetmap.org/reverse?lat=19.0760&lon=72.8777&format=json"

# Returns:
{
  "display_name": "Gateway of India, Apollo Bunder, Colaba, Mumbai, Maharashtra, India",
  "address": {
    "tourism": "Gateway of India",
    "neighbourhood": "Apollo Bunder",
    "suburb": "Colaba",
    "city": "Mumbai",
    "state": "Maharashtra",
    "country": "India"
  }
}

# Google Maps API reverse geocoding
curl "https://maps.googleapis.com/maps/api/geocode/json?latlng=19.0760,72.8777&key=YOUR_KEY"
```

---

## 8. Physical Location Profiling

### 8.1 Building a Location Profile

When investigating where a person lives or works:

```
Data sources that reveal location:
  ├── Social media posts (location tags, background details, local events)
  ├── Check-ins on Facebook, Foursquare, Google
  ├── LinkedIn → "Mumbai Metropolitan Area"
  ├── Truecaller → carrier info → region
  ├── Email header IP → ISP + approximate city
  ├── WHOIS → domain registrant address
  ├── Company registration → registered office address
  ├── Court records → address of parties
  ├── Voter registration → address (some countries public)
  └── Property records → some countries make these public
```

### 8.2 Inferring Home Location from Social Media

```
Method 1: Morning and evening posts
  Posts at 7-8 AM and 6-8 PM are typically from home
  → Background details in morning/evening posts = home area

Method 2: Local context
  Posts about: local restaurants, local events, local sports teams
  → Narrows to a city or neighborhood

Method 3: Delivery/service mentions
  "Waiting for Swiggy delivery" → uses Indian delivery apps → India
  "Ordered from Domino's on MG Road" → Bangalore, MG Road area

Method 4: Travel posts vs. home posts
  Consistent background for non-travel posts = home area
  Sudden exotic backgrounds = travel period

Method 5: Tagged locations over time
  Plot all tagged locations on a map
  → Cluster analysis → home and work locations emerge
  → One cluster at the same residential area = likely home

Method 6: Commute references
  "Stuck in traffic on NH8" → near NH-8 (Delhi-Gurugram highway)
  "Late train from CST" → commutes into Mumbai CST
```

### 8.3 Confirming a Business Address

```
Step 1: Google Maps search → company name → see listed address
Step 2: Google Street View → does it match the company's branding?
Step 3: Check company website → "Contact Us" / "Our Office" pages
Step 4: WHOIS → registrant address (may differ from HQ)
Step 5: Company registry (MCA21, Companies House) → registered address
Step 6: LinkedIn → "Office" photos posted by employees
Step 7: Glassdoor → office photos uploaded by employees
Step 8: Google Maps "See inside" → 360° interior if available
Step 9: Job postings → "Location: [address]"

Verify the address:
  → Does Street View show a matching building?
  → Does the building exist? Is it occupied?
  → Is there a physical signage / branding visible?
```

---

## 9. GeoINT for Event Verification

### 9.1 Verifying That an Event Occurred at a Claimed Location

```
Step 1: Get all media from the event (photos, videos, social posts)

Step 2: Identify geolocatable features in each:
  ├── Background landmarks
  ├── Road signs and street names
  ├── Building facades
  ├── Vegetation type
  └── Vehicle types and plates

Step 3: Geolocate at least 3 separate media items
  → If they all point to the same location → location confirmed

Step 4: Cross-reference with satellite imagery
  → Was the event large enough to be visible from space?
    (protests, fires, floods, construction)
  → Use Sentinel Hub to look at imagery from the claimed date
  → If visible in satellite → strong confirmation

Step 5: Cross-reference with witness accounts
  → Social media posts from other people near that location
  → Local news reports
  → Geolocation data from multiple independent sources

Step 6: Check for inconsistencies
  → Does the sun angle in the photos match the claimed date/time?
  → Do weather conditions match local weather records?
  → Are there any anachronisms (cars, signage, clothing not from that era)?
```

### 9.2 Weather Verification

```
Verify weather conditions at a claimed location and time:

Wunderground:    wunderground.com/history
  → Historical weather by city and date
  → Shows: temperature, conditions, precipitation, wind, visibility

Windy:           windy.com
  → Historical weather map visualization
  → Cloud cover data useful for confirming outdoor conditions

Wolfram Alpha:
  → "weather in Mumbai on March 15 2024"
  → Returns: conditions, temperature, sunrise/sunset times

Meteomanz:       meteomanz.com
  → Historical METAR (aviation weather) data
  → Very accurate for airports worldwide

Usage:
  Claimed date: "This photo is from March 15, 2024 in Mumbai"
  Image shows: bright sunshine, no clouds
  Weather records for Mumbai March 15, 2024: heavy rain all day
  → Conclusion: date is wrong OR location is wrong
```

### 9.3 Google Street View Date Matching

```
Street View images are dated. If your subject photo shows:
  A building that was demolished in 2022 → photo is from before 2022
  A new building that was built in 2023 → photo is from after 2023
  Street View from 2021 shows the same building → consistent

How to check Street View dates:
  1. Navigate to the location in Street View
  2. The date is shown at the bottom-left (e.g., "May 2023")
  3. Click the clock icon → see all available historical captures
  4. Navigate through dates to find when features appeared/disappeared
```

---

## 10. Terrain & Elevation Analysis

### 10.1 Identifying Terrain Type

```
Terrain clues and what they tell you:
  ├── Flat agricultural land          →  Plains region (India: Punjab, Haryana, UP)
  ├── Red laterite soil               →  Deccan plateau, Goa, parts of Kerala
  ├── Rocky terrain with sparse brush →  Arid zone (Rajasthan, parts of AP)
  ├── Dense tropical forest           →  Western Ghats, Northeast India, SE Asia
  ├── Snow-capped mountains           →  Himalayan region or high altitude
  ├── Sand dunes                      →  Desert (Rajasthan, Middle East, N. Africa)
  ├── Terraced hillside agriculture   →  Himalayan foothills, SE Asia, China
  └── River delta / mangroves         →  Coastal estuary (Sundarbans, Kerala backwaters)
```

### 10.2 Elevation Tools

```
Google Earth Pro:
  → Right-click any point → "What's here?" → shows elevation
  → Tools → Ruler → Path → shows elevation profile along a route

Open Elevation API (free):
  curl "https://api.open-elevation.com/api/v1/lookup?locations=19.0760,72.8777"
  # Returns elevation in meters

Terrain.party:  terrain.party
  → Download elevation heightmaps for any area
  → Useful for terrain matching in mountainous photos

Peakvisor:      peakvisor.com
  → Identify mountain peaks visible in a photo
  → Enter approximate location → shows peaks visible from that point
  → Match peaks in your image to the database
```

---

## 11. Geolocation Challenges & Training

### 11.1 GeoGuessr — Primary Training Tool

```
URL: geoguessr.com

What it is:
  A game that drops you in a random Google Street View location.
  Your goal: identify the location and drop a pin on the world map.
  Scored by distance from the actual location.

Training value:
  Builds rapid visual recognition of:
  → Country-level cues (traffic direction, language, architecture)
  → Region-level cues (terrain, vegetation, infrastructure)
  → City-level cues (street layout, building density, signage)

Strategy for better scores:
  1. Check traffic direction FIRST (left/right hand)
  2. Read any visible script → narrows to a script region immediately
  3. Look at road surface and markings → infer development level
  4. Check vegetation → climate zone
  5. Look for utility poles → style varies by country
  6. Walk along the road → find a sign with a place name
  7. Google the place name → confirm location

Modes to practice:
  → World (hardest — truly random)
  → Country streaks (identify country only — good for beginners)
  → City streaks
  → Custom maps (India only, Europe only, etc.)
```

### 11.2 Bellingcat Quizzes

```
URL: bellingcat.com/category/resources/quizzes/

Real-world OSINT challenges:
  → Geolocate photos from conflict zones
  → Identify vehicles from partial views
  → Verify social media claims with GeoINT

These are harder than GeoGuessr because:
  → No Street View available at location
  → Images are often low quality
  → Subject may be remote or deliberately obscured
```

### 11.3 Sofia Santos OSINT Exercises

```
URL: gralhix.com/list-of-osint-exercises/

Structured GeoINT exercises with solutions:
  → Each exercise has an image and a question
  → Solutions published → you can check your work
  → Difficulty ranges from beginner to advanced

Highly recommended for systematic skill building.
```

### 11.4 Quiztime

```
Twitter/X: #Quiztime

A community of OSINT practitioners who post daily geolocation challenges.
  → Follow #Quiztime on Twitter
  → A challenge image is posted
  → Community members post their solutions and methods
  → Learn from others' approaches to the same image

Also: @Geolocation_OSINT, @sector035, @OSINTCurious
```

---

## 12. Advanced GeoINT Techniques

### 12.1 Triangulation from Multiple Images

```
If you have multiple images of the same subject from different angles:

Step 1: Geolocate each image independently
  → Get a direction (bearing) the camera was facing in each

Step 2: Draw lines on a map
  → From location A, draw a line in the camera's facing direction
  → From location B, draw a line in the camera's facing direction

Step 3: The intersection = subject's location
  → Used in conflict OSINT to find where artillery fire came from
  → Used to find where a person is standing in a photo

Tools:
  Google Maps (manual line drawing)
  QGIS (free GIS software) → import multiple bearing lines → find intersection
  GeoJSON.io → draw and visualize lines on a map
```

### 12.2 Building Footprint Matching

```
Technique: Match the shape of a building from a satellite photo
           to a building visible in a ground-level image.

Step 1: From the ground-level image, estimate the building's shape:
  → Number of corners visible
  → Relative proportions
  → Distinctive features (curved edges, internal courtyard, etc.)

Step 2: Search satellite imagery for matching footprints
  → OpenStreetMap shows building footprints
  → Google Earth Pro shows detailed outlines
  → overpass-turbo can query building shapes

Step 3: Match roof features
  → Roof color, shape, equipment (HVAC, water tanks, solar panels)
  → These are distinctive and change rarely

Step 4: Confirm with Street View or Mapillary from all four sides
```

### 12.3 Linguistic Analysis for Geolocation

```
When text is visible in images:

Script → region mapping:
  Devanagari   →  Hindi/Marathi/Nepali → India/Nepal
  Cyrillic     →  Russia/Eastern Europe/Central Asia
  Arabic       →  Middle East/North Africa
  Thai         →  Thailand
  Hangul       →  Korea
  Traditional Chinese → Taiwan/Hong Kong
  Simplified Chinese  → Mainland China

Specific language identifiers:
  Language of street signs → national language → country
  Bilingual signs → border regions or federal territories
  English + local script → tourist area or major city

Brand names:
  "पेट्रोल" (Petrol in Hindi) on a pump → India
  "BPCL / HPCL / IOC" branding → India (PSU fuel companies)
  "Petrobras" → Brazil
  "TotalEnergies" → Europe/Africa
  "Texaco" → Americas / some European countries

Phone number formats visible in photos:
  "+91" prefix → India
  "+1" prefix  → US / Canada
  "+44" prefix → UK
  "1800-XXX-XXXX" → likely India or US toll-free
```

---

## 13. Level 5 Practice Tasks

### Task 1 — IP Geolocation
- Take 5 IP addresses from any public source (Shodan results, WHOIS records)
- Run each through ip-api.com and ipinfo.io
- Note: country, city, ISP, and whether it is a data center or residential IP
- Cross-reference with Shodan: `shodan host <IP>` — do the results agree?

### Task 2 — WiGLE Search
- Create a free WiGLE account
- Search for your home Wi-Fi network name (SSID) — see if it appears
- Try searching for 3 other SSIDs from different locations
- Note: what privacy implications does this reveal?

### Task 3 — Flight Investigation
- Pick a major news event that involved a notable aircraft (private jet, cargo plane)
- Use FlightAware + FlightRadar24 to find the tail number
- Trace: where did it come from, where did it go, who owns it
- Use ADS-B Exchange to check if the same aircraft appears with different data

### Task 4 — GeoINT Challenge
- Complete 5 challenges from gralhix.com (Sofia Santos OSINT exercises)
- For EACH challenge, document:
  - First visual clue that narrowed the location
  - All subsequent clues used
  - The tool used to confirm
  - Final coordinates and confidence level
- Check the solution and note what you missed

### Task 5 — GeoGuessr Country Mastery
- Play 20 rounds of GeoGuessr (World map)
- Keep a log: for each round, write the first visual clue that suggested the country
- After 20 rounds, identify your weakest regions
- Play the specific country/region maps for those weak areas for 10 more rounds

### Task 6 — Sun Angle Verification
- Find any outdoor photo online that includes a shadow and a date/time claim
- Use SunCalc to verify:
  - Set the claimed location and date
  - Adjust time until shadow direction matches
  - Does the confirmed time match the claimed time?
- Document your findings

### Task 7 — Multi-Source Event Verification
- Find any social media post claiming to show an event at a specific location
- Use at least 3 of these to verify or refute:
  - Landmark identification
  - Sun angle / shadow analysis
  - Street View matching
  - Satellite imagery (Sentinel Hub or Google Earth)
  - Weather records
  - Local news cross-reference

---

## 14. Level 5 Summary

```
✅ GeoINT answers WHERE — from images, social media, signals, and open data
✅ Google Earth Pro historical imagery reveals changes over time
✅ Sentinel Hub (ESA) provides free satellite imagery updated every 5 days
✅ Overpass Turbo queries OpenStreetMap for specific features in an area
✅ IP geolocation: accurate to city level; useless through VPN/Tor
✅ WiGLE maps Wi-Fi SSIDs to physical locations — powerful pivot from social posts
✅ FlightAware + ADS-B Exchange track aircraft tail numbers and routes
✅ MarineTraffic tracks ships by name, MMSI, or IMO number
✅ SunCalc + shadow analysis reveals time, direction, and hemisphere from any photo
✅ Social media location tags + Snapchat Map = real-time crowd intelligence
✅ Weather records verify or debunk claimed photo dates/locations
✅ GeoGuessr + Sofia Santos exercises = best way to build visual recognition skills
✅ Triangulation from multiple images → pinpoint a subject's exact location
✅ Script analysis → instant country-level shortlist from any text in an image

When you are done here → Proceed to Level 6: Network & Infrastructure OSINT
```

---

## 15. Tools Quick Reference

| Tool | Purpose | URL / Install |
|---|---|---|
| **Google Maps** | Primary geolocation + Street View | maps.google.com |
| **Google Earth Pro** | Historical satellite + sun simulation | earth.google.com |
| **Bing Maps** | Oblique aerial / bird's eye view | bing.com/maps |
| **Sentinel Hub EO Browser** | Free ESA satellite imagery | sentinel-hub.com |
| **Mapillary** | Crowd-sourced street images | mapillary.com |
| **KartaView** | Crowd-sourced street images | kartaview.org |
| **Overpass Turbo** | Query OpenStreetMap features | overpass-turbo.eu |
| **SunCalc** | Sun angle + shadow analysis | suncalc.org |
| **Shadow Calculator** | Precise shadow length analysis | shadowcalculator.eu |
| **WiGLE** | Wi-Fi SSID geolocation | wigle.net |
| **OpenCelliD** | Cell tower geolocation | opencellid.org |
| **FlightAware** | Flight tracking + aircraft lookup | flightaware.com |
| **FlightRadar24** | Live + historical flight map | flightradar24.com |
| **ADS-B Exchange** | Unfiltered aircraft tracking | adsbexchange.com |
| **OpenSky Network** | Free flight API | opensky-network.org |
| **MarineTraffic** | Ship tracking | marinetraffic.com |
| **VesselFinder** | Ship tracking (alternative) | vesselfinder.com |
| **ip-api.com** | IP geolocation (free) | ip-api.com |
| **ipinfo.io** | IP geolocation + org | ipinfo.io |
| **Nominatim** | Reverse geocoding (free OSM) | nominatim.openstreetmap.org |
| **Wunderground** | Historical weather | wunderground.com/history |
| **Windy** | Weather map + history | windy.com |
| **GeoGuessr** | Geolocation training game | geoguessr.com |
| **gralhix.com** | OSINT GeoINT exercises | gralhix.com |
| **Bellingcat quizzes** | Real-world GeoINT challenges | bellingcat.com |
| **Peakvisor** | Mountain peak identification | peakvisor.com |
| **GeoJSON.io** | Draw and visualize map data | geojson.io |
| **vahan.nic.in** | India vehicle registration | vahan.nic.in |
