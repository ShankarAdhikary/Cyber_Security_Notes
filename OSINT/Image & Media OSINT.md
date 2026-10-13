# OSINT — Level 3: Image & Media OSINT
> Images and videos are among the richest sources of unintentional intelligence.
> A single photograph can reveal GPS coordinates, the time it was taken, the device
> used, the location via landmarks, the people present, and the direction the camera
> was pointing. This level teaches you to extract every layer of information from
> visual media.

---

## 1. The Image OSINT Mindset

Most people look AT an image. An OSINT investigator looks INTO it.

```
What a regular person sees:
  "A photo of a person standing in front of a building."

What an investigator extracts:
  ├── Person: face → reverse image search → other profiles
  ├── Clothing: brand, style, weather appropriateness → season / climate clue
  ├── Building: architecture style → country / region
  ├── Signs: language, script, store names → location confirmation
  ├── Vehicles: license plate format, make/model → country, region
  ├── Vegetation: plant species → climate zone
  ├── Sky: sun angle, cloud type → time of day, hemisphere
  ├── Shadows: direction and length → compass bearing + time of day
  ├── Reflections: in windows, glasses, puddles → what is behind the camera
  └── Metadata: GPS coordinates, device, timestamp (if not stripped)
```

### 1.1 Two Layers of Image Intelligence

```
Layer 1 — METADATA (invisible, technical)
  Data embedded in the file itself: GPS, date, camera, software
  Extracted with: ExifTool, Jeffrey's Exif Viewer

Layer 2 — CONTENT (visible, analytical)
  Information visible in the image: landmarks, text, faces, objects
  Analyzed with: reverse image search, manual analysis, geolocation tools
```

---

## 2. Image Metadata (EXIF Data)

### 2.1 What is EXIF?

**EXIF (Exchangeable Image File Format)** is metadata stored inside image files
(JPG, TIFF, HEIC, PNG partially). It is written by the camera or device automatically
and contains technical and contextual information about the photo.

```
EXIF fields and their intelligence value:

Field                   │ Example Value              │ Intelligence Value
────────────────────────┼────────────────────────────┼──────────────────────────
GPS Latitude            │ 19.0760° N                 │ Exact location of photo
GPS Longitude           │ 72.8777° E                 │ Exact location of photo
GPS Altitude            │ 14 m                       │ Floor / elevation clue
Date/Time Original      │ 2024-03-15 14:32:11        │ When the photo was taken
Make                    │ Apple                      │ Device manufacturer
Model                   │ iPhone 15 Pro              │ Exact device model
Software                │ iOS 17.3.1                 │ OS version → security profile
Focal Length            │ 6.86 mm                    │ Lens type
Flash                   │ Flash did not fire         │ Lighting conditions
Image Width/Height      │ 4032 x 3024                │ Full resolution (not cropped?)
Artist / Copyright      │ John Doe                   │ Creator name
Comment                 │ (custom text)              │ Sometimes personal notes
Thumbnail               │ (embedded image)           │ May differ from main image
```

### 2.2 ExifTool — The Standard Tool

ExifTool is the most comprehensive metadata extraction tool. It reads virtually
every metadata format from images, videos, audio, and documents.

**Installation:**
```bash
# Debian / Kali / Ubuntu
sudo apt install exiftool

# macOS
brew install exiftool

# Windows
# Download from: exiftool.org → ExifTool-XX.XX.exe → rename to exiftool.exe
```

**Basic Commands:**
```bash
# Extract all metadata from a single image
exiftool image.jpg

# Extract specific fields only
exiftool -GPSLatitude -GPSLongitude -DateTimeOriginal image.jpg

# Extract metadata from all images in a folder
exiftool /path/to/folder/

# Recursive (includes subfolders)
exiftool -r /path/to/folder/

# Output as JSON
exiftool -json image.jpg

# Output as CSV (useful for bulk analysis)
exiftool -csv *.jpg > metadata.csv

# Extract and display GPS coordinates in decimal format
exiftool -gpslatitude -gpslongitude -n image.jpg

# Strip all metadata from an image (defensive)
exiftool -all= image.jpg

# Extract thumbnail image embedded in file
exiftool -b -ThumbnailImage image.jpg > thumbnail.jpg

# Show only GPS-related fields
exiftool -gps:all image.jpg

# Identify all files in a folder that contain GPS data
exiftool -if '$GPSLatitude' -filename -GPSLatitude -GPSLongitude /folder/
```

**Reading the output:**
```
$ exiftool photo.jpg

File Name                       : photo.jpg
File Size                       : 4.2 MB
MIME Type                       : image/jpeg
Make                            : Apple
Camera Model Name               : iPhone 15 Pro
Software                        : 17.3.1
Date/Time Original              : 2024:03:15 14:32:11   ← when taken
GPS Latitude                    : 19 deg 4' 33.60" N    ← location
GPS Longitude                   : 72 deg 52' 39.72" E   ← location
GPS Altitude                    : 14.2 m Above Sea Level
GPS Speed                       : 0 km/h                ← stationary
GPS Img Direction               : 247.5                 ← camera pointed SW
GPS Date/Time                   : 2024:03:15 09:02:11   ← UTC time
```

### 2.3 Converting GPS Coordinates

ExifTool returns GPS in degrees/minutes/seconds (DMS) format.
Convert to decimal for use in Google Maps:

```
DMS:     19 deg 4' 33.60" N,  72 deg 52' 39.72" E

Convert:
  Decimal = Degrees + (Minutes/60) + (Seconds/3600)
  19 + (4/60) + (33.60/3600) = 19.0760° N
  72 + (52/60) + (39.72/3600) = 72.8777° E

Google Maps URL:
  https://www.google.com/maps?q=19.0760,72.8777

OR use -n flag in exiftool for direct decimal output:
  exiftool -GPSLatitude -GPSLongitude -n image.jpg
```

### 2.4 Web-Based EXIF Tools

```
Jeffrey's Exif Viewer:   exifdata.com
  → Upload or provide URL → full EXIF display + map

Pic2Map:                  pic2map.com
  → Extracts GPS → shows on Google Maps automatically

Exif.tools:               exif.tools
  → Clean interface; good for quick checks

Forensically:             29a.ch/photo-forensics/
  → Advanced: ELA (Error Level Analysis), clone detection, noise analysis
  → Detects if an image has been edited or composited
```

### 2.5 What Platforms Strip EXIF

```
STRIPS EXIF (GPS removed):      RETAINS EXIF (may keep GPS):
  Instagram ✅                    Telegram (file send, not photo) ✅
  Facebook ✅                     Discord (some contexts) ✅
  Twitter/X ✅                    WhatsApp → strips in most cases
  Reddit ✅                       Direct email attachments ✅
  LinkedIn ✅                     Shared via Dropbox/Drive links ✅
  TikTok ✅                       AirDrop ✅
  Snapchat ✅                     Bluetooth transfer ✅
```

> **Key insight:** Always check EXIF on images received directly (email, messaging,
> file transfer). Do not expect EXIF from social media downloads — it will be stripped.

---

## 3. Reverse Image Search

Reverse image search finds where else an image (or a visually similar image) appears
on the internet. It is used to:

```
✅ Find the original source of an image
✅ Find other accounts using the same profile picture
✅ Verify if an image has been used out of context (misinformation)
✅ Find higher-resolution versions of an image
✅ Identify a person by their face
✅ Find images taken at the same location
✅ Track how an image has spread across the web
```

### 3.1 Google Lens

```
URL: images.google.com → click the camera icon
  OR: drag and drop an image
  OR: right-click → "Search image with Google"

Strengths:
  ✅ Best for object identification (buildings, products, logos)
  ✅ Good for landmark matching
  ✅ Strong for finding visually similar images
  ✅ Integrated with Google's Knowledge Graph

Weaknesses:
  ❌ Weaker than Yandex for face matching
  ❌ May not find older or low-traffic pages
```

**Using Google Lens from CLI (for bulk search):**
```bash
# No direct CLI — use the web interface
# For automation, use SerpAPI (paid) or custom Python with selenium
```

### 3.2 Yandex Images — The Most Powerful for Faces

```
URL: yandex.com/images → click the camera icon

Strengths:
  ✅ BEST for face matching — finds faces across different photos
  ✅ Finds images that Google misses
  ✅ Strong for Eastern European and Russian web content
  ✅ Often finds the earliest version of an image

Weaknesses:
  ❌ Results may include non-English sites
  ❌ Interface less intuitive than Google

Usage:
  1. Go to yandex.com/images
  2. Click the camera icon in the search bar
  3. Upload the image or paste a URL
  4. Results show: similar images, pages containing the image, other sizes
  5. Click "Similar images" for face/object matches
  6. Click "Sites where this image was found" for source tracking
```

### 3.3 TinEye

```
URL: tineye.com

Strengths:
  ✅ BEST for finding exact copies of an image
  ✅ Shows the OLDEST occurrence of an image on the web
  ✅ Tracks how an image has spread over time
  ✅ Can filter results by oldest/newest/most changed

Usage:
  1. Go to tineye.com
  2. Upload image or paste URL
  3. Sort by "Oldest" → find where the image originally appeared
  4. "Best Match" → find highest-quality version

Weaknesses:
  ❌ Weak for face matching (not designed for it)
  ❌ Smaller index than Google or Yandex
```

### 3.4 Bing Visual Search

```
URL: bing.com/visualsearch

Strengths:
  ✅ Good alternative to Google — different index
  ✅ Strong for product identification
  ✅ Finds pages Google may not index

Usage:
  1. Go to bing.com
  2. Click the camera icon in the search bar
  3. Upload image or paste URL
```

### 3.5 PimEyes — Face Search Engine

```
URL: pimeyes.com

What it does:
  Searches specifically for FACES across a large index of public web images.
  Upload a photo of a face → find other photos of that same person online.

Free tier:    Blurred results; shows that matches exist
Paid tier:    Full results with source URLs

Accuracy:     Very high for clear, frontal face photos
Legal note:   Use only on public figures or within your explicit authorization
              EU: GDPR restricts face recognition data use significantly
```

### 3.6 FaceCheck.ID

```
URL: facecheck.id

Similar to PimEyes — face search across public web images.
Free tier available with limited results.
Often finds matches on social media profiles.
```

### 3.7 Multi-Engine Approach

Never rely on a single engine. Use all of them — each has a different index.

```
Step 1: Google Lens     → objects, landmarks, similar images
Step 2: Yandex Images   → face matching, broader web coverage
Step 3: TinEye          → find original source, oldest occurrence
Step 4: Bing Visual     → alternative index
Step 5: PimEyes         → face-specific deep search (for public figures)

If any engine finds a match → follow it to the source page → extract new data
```

### 3.8 Reverse Image Search from the CLI

```bash
# Using Python + requests to submit to Yandex (basic)
# Most investigators use the web interface — no CLI standard exists

# Bulk approach: use gallery-dl to download images, then manually submit key ones
pip3 install gallery-dl
gallery-dl https://instagram.com/p/POSTID/  # Download post image for analysis
```

---

## 4. Image Content Analysis (GeoINT from Images)

When EXIF data is stripped or missing, you can still geolocate an image by
analyzing its visual content. This is called **GeoINT** (Geospatial Intelligence).

### 4.1 The GeoINT Methodology

```
Step 1: ELIMINATE
  What can you rule out immediately?
  → Snow-covered mountains → not a tropical region
  → Right-hand traffic → not UK, Australia, India (left-hand drive)
  → Arabic script → Middle East / North Africa region
  → Specific flag visible → country confirmed

Step 2: NARROW
  What narrows the location to a country or region?
  → Architecture style → Soviet-era? Colonial? Modern Asian?
  → Vegetation type → tropical? desert? temperate?
  → Vehicle type and license plate format
  → Road markings and signage style
  → Electrical infrastructure (pole style, transformer type)

Step 3: PINPOINT
  What gives the exact location?
  → Visible business names → Google Maps search
  → Distinctive landmarks → landmarks database
  → Street signs with full names
  → Google Street View matching
```

### 4.2 Visual Clues Reference

**Traffic and Roads:**
```
Left-hand traffic countries:  UK, India, Australia, Japan, South Africa
Right-hand traffic countries: Most of the world (US, Europe, China)

Road markings:
  Yellow center line     →  USA, Canada
  White center line      →  Most of Europe, India
  No center line         →  Rural roads worldwide

Speed limit signs:
  km/h (red circle)      →  Europe, India, most of world
  mph (white rectangle)  →  USA, UK
```

**License Plates:**
```
Blue strip on left + letters/numbers   →  European Union (EU member state)
Yellow plate (rear)                    →  UK
White/yellow with state code           →  India (state code in 2 letters)
White plate, 3 letters + 4 numbers     →  USA (varies by state)
White plate with 4 numbers             →  China
```

**Vegetation:**
```
Palm trees                             →  Tropical / subtropical
Pine/conifer forests                   →  Northern hemisphere, high altitude
Eucalyptus trees                       →  Australia, parts of Africa
Baobab trees                           →  Africa, Madagascar
Cherry blossom (sakura)                →  Japan, South Korea
Paddy fields (rice)                    →  South/Southeast Asia
```

**Architecture:**
```
Soviet-era concrete blocks             →  Former Soviet states (Russia, Central Asia, Eastern Europe)
Wooden houses on stilts                →  Southeast Asia, flood-prone areas
Adobe / mud brick structures           →  North Africa, Middle East, Central America
Victorian terraced houses              →  UK
Pagoda-style roofs                     →  China, Japan, Korea, Vietnam
Red-tiled roofs on white walls         →  Mediterranean (Spain, Italy, Greece)
Colorful colonial architecture         →  Latin America, parts of Southeast Asia
```

**Utility Infrastructure:**
```
Wooden utility poles (single crossbar) →  USA, Canada
Concrete poles with multiple wires     →  South/Southeast Asia
Underground utilities (no poles)       →  Western Europe (urban)
Yellow fire hydrant                    →  USA
Red fire hydrant                       →  Canada, UK variants
```

**Language and Script:**
```
Devanagari script           →  Hindi (India, Nepal)
Cyrillic script             →  Russia, Eastern Europe, Central Asia
Arabic script               →  Middle East, North Africa
Thai script                 →  Thailand
Hangul script               →  Korea
Kanji / Hiragana / Katakana →  Japan
Traditional Chinese         →  Taiwan, Hong Kong
Simplified Chinese          →  Mainland China
```

### 4.3 Sun Angle Analysis

The position of the sun in an image reveals:
- Time of day (roughly)
- Direction the camera was pointing
- Hemisphere (northern or southern)

**SunCalc:**
```
URL: suncalc.org

Usage:
  1. Click to set the location on the map (approximate)
  2. Set the date (from EXIF or context clues)
  3. Drag the time slider to match the shadow direction in the image
  4. The tool shows sun azimuth (compass direction) and elevation
  5. Compare the shadow direction in the image to the sun position on the map

What it tells you:
  Shadow points NORTH  →  Sun is in the south  →  northern hemisphere photo
  Shadow points SOUTH  →  Sun is in the north  →  southern hemisphere photo
  Sun directly above   →  Tropical region near equator, midday
```

**Manual shadow analysis:**
```
1. Identify the longest shadow in the image
2. The sun is in the OPPOSITE direction of the shadow
3. At solar noon, shadows point exactly true north (NH) or true south (SH)
4. Shadow length relative to object height → sun elevation angle → time of day

Example:
  Shadow points roughly west → sun is in the east → morning photo
  Shadow points roughly east → sun is in the west → afternoon photo
  Short shadow → sun is high → midday
  Long shadow → sun is low → morning or evening
```

### 4.4 Google Street View Matching

Once you have narrowed the location to a neighborhood or street:

```
Step 1: Open Google Maps → switch to Street View
Step 2: Navigate to the approximate area
Step 3: Compare the following between the image and Street View:
  ├── Building facades (windows, balconies, materials)
  ├── Shop signage and names
  ├── Road markings and curb style
  ├── Utility poles and power lines
  ├── Trees and planters
  ├── Parked vehicles (if in multiple images)
  └── Ground texture (tile, asphalt, cobblestone)

Tips:
  → Use historical Street View (clock icon) to match older photos
  → Check ALL four directions from a location — target may be behind the camera
  → Shadows in Street View confirm direction even without compass
```

**Overpass Turbo (OpenStreetMap querying):**
```
URL: overpass-turbo.eu

Use it to find specific features in an area:
  → All McDonald's in a city → if golden arches visible in image
  → All gas stations of a specific brand
  → All schools, hospitals, or distinctive buildings

Example query (find all Starbucks in Mumbai):
  [out:json][timeout:25];
  area["name"="Mumbai"]->.searchArea;
  node["name"="Starbucks"](area.searchArea);
  out body;
```

### 4.5 Identifying Specific Objects

```
Aircraft:
  → ADS-B Exchange (adsbexchange.com) → tail number → owner → route
  → FlightAware → historical flights

Ships:
  → MarineTraffic → vessel name or MMSI number

Vehicles:
  → License plate visible → country/state confirmed
  → Plate format → narrow to region
  → India: vahan.nic.in for vehicle lookup (VIN or plate)

Military equipment:
  → oryx blog (oryxspioenkop.com) → visual military equipment identification
  → Armament Research Services

Buildings:
  → Google Maps Satellite + Street View
  → Bing Maps (sometimes better aerial coverage)
  → Google Earth Pro (historical imagery)
```

---

## 5. Video OSINT

### 5.1 Video as a Source of Intelligence

Videos contain everything images do — and more:
```
  ├── Multiple frames → geolocate from any frame
  ├── Audio → language, accent, background sounds, music
  ├── Movement → time of day, weather, crowd behavior
  ├── Embedded metadata → creation date, device, software
  └── Context → events, dates, identities
```

### 5.2 Extracting Frames for Analysis

```bash
# Install ffmpeg
sudo apt install ffmpeg

# Extract 1 frame per second
ffmpeg -i video.mp4 -vf fps=1 frames/frame_%04d.jpg

# Extract every 5 seconds
ffmpeg -i video.mp4 -vf fps=1/5 frames/frame_%04d.jpg

# Extract a specific frame at 00:01:23
ffmpeg -i video.mp4 -ss 00:01:23 -vframes 1 frame_at_83sec.jpg

# Extract ALL frames (warning: large output for long videos)
ffmpeg -i video.mp4 frames/frame_%06d.jpg

# Then: run each key frame through reverse image search
# Focus on frames with visible landmarks, text, or faces
```

### 5.3 Video Metadata

```bash
# Extract video metadata with ExifTool
exiftool video.mp4

# Key fields to look for:
#   Create Date       → when the video was recorded
#   Make / Model      → recording device
#   GPS Coordinates   → if recorded with GPS (GoPro, some phones)
#   Duration          → total length
#   Software          → editing software (reveals post-processing)

# For YouTube videos, metadata via yt-dlp:
pip3 install yt-dlp
yt-dlp --get-description https://www.youtube.com/watch?v=VIDEOID
yt-dlp --write-info-json https://www.youtube.com/watch?v=VIDEOID
# The .info.json file contains: upload date, description, tags, channel, view count
```

### 5.4 InVID / WeVerify — Video Verification

The standard tool for verifying video authenticity and finding original sources.

```
URL: invid-project.eu (browser extension for Chrome and Firefox)
     Also available at: weverify.eu

Features:
  ├── Keyframe extraction → submit each frame to reverse image search
  ├── Metadata analysis → extract video metadata without downloading
  ├── YouTube reverse search → find where the video or its frames appear
  ├── Twitter/Facebook video download for offline analysis
  ├── Magnifier tool → zoom into specific parts of a frame
  └── Geolocation assistant → structured geoINT workflow

Usage (browser extension):
  1. Install the InVID/WeVerify extension
  2. Right-click any video on a webpage → "Analyse with InVID"
  3. OR: open the extension → paste a video URL
  4. Click "Keyframes" → get 8-16 frames
  5. Submit each frame to Google Lens and Yandex Images
```

### 5.5 Audio Analysis from Video

```
What audio can reveal:
  ├── Language and dialect → region / country
  ├── Accent → more specific region
  ├── Background sounds:
  │   ├── Call to prayer (Azan) → Muslim-majority area
  │   ├── Church bells → Christian community
  │   ├── Traffic sounds → urban area
  │   ├── Ocean waves → coastal
  │   ├── Crickets / insects → rural, warm climate
  │   ├── Train announcements → specific station / country
  │   └── Military equipment sounds → tank, aircraft type
  ├── Music → Shazam can identify → origin, date
  └── Speech content → names, places, events mentioned

Tools:
  Shazam:       shazam.com → identify music in video
  Google Search: hum or describe the music to find it
  Adobe Podcast: podcast.adobe.com → AI audio enhancement + transcription
  Whisper (OpenAI):
    pip3 install openai-whisper
    whisper video.mp4 --language auto   # Transcribes audio to text
```

---

## 6. Document Metadata OSINT

### 6.1 What Documents Reveal

Office documents (Word, Excel, PowerPoint, PDF) contain metadata that reveals:

```
Field               │ Example                    │ Intelligence Value
────────────────────┼────────────────────────────┼────────────────────────────
Author              │ John Doe                   │ Real name of creator
Last Modified By    │ jane.doe                   │ Username of last editor
Creator Software    │ Microsoft Word 2019        │ Software version
Company             │ Acme Corp                  │ Organization name
Created Date        │ 2024-01-15 09:23:00        │ When document was created
Modified Date       │ 2024-03-10 16:45:22        │ Last edit date
Template            │ \\server1\templates\corp   │ Internal server path!
Revision Number     │ 47                         │ Document has been edited 47 times
Total Editing Time  │ 3h 22m                     │ Active work duration
Comments            │ (hidden comments in doc)   │ Internal notes
Embedded objects    │ (linked files, images)     │ May reference internal paths
Printer name        │ \\server2\HP-LaserJet-4F   │ Internal printer / server name
```

### 6.2 ExifTool for Documents

```bash
# Extract metadata from a PDF
exiftool document.pdf

# Extract from a Word document
exiftool document.docx

# Extract from Excel
exiftool spreadsheet.xlsx

# Extract from PowerPoint
exiftool presentation.pptx

# Look specifically for author and company
exiftool -Author -Creator -Company -LastModifiedBy document.docx

# Batch process all PDFs in a folder
exiftool -csv *.pdf > document_metadata.csv
```

### 6.3 FOCA — Focused Document Metadata Extraction

FOCA is a Windows tool specifically designed for mass document metadata extraction
from a target domain. It finds all public documents, downloads them, and extracts
metadata automatically.

```
Download:    github.com/ElevenPaths/FOCA/releases
Platform:    Windows only
Cost:        Free

Workflow:
  1. Open FOCA → New Project → enter target domain (e.g., example.com)
  2. Select file types to search for: PDF, DOC, DOCX, XLS, PPT, etc.
  3. Click "Search All" → FOCA uses Google, Bing, Exalead to find all public documents
  4. Download selected documents
  5. Extract metadata → view in the interface

What FOCA reveals:
  ├── All unique author names (list of employees)
  ├── All unique usernames (IT account names)
  ├── All software versions used
  ├── Internal server paths and network shares (\\server\path)
  ├── Printer names
  └── Email address patterns (sometimes in document properties)
```

### 6.4 Metagoofil — Linux Alternative to FOCA

```bash
# Install
sudo apt install metagoofil
# OR
git clone https://github.com/laramies/metagoofil.git

# Basic usage
metagoofil -d example.com -t pdf,doc,xls -o output_folder/

# Flags:
# -d   target domain
# -t   file types (comma-separated)
# -l   number of results (default: 200)
# -n   number of files to download (default: 5)
# -o   output folder for downloaded files

# What it does:
#   Google searches for target domain + filetype
#   Downloads found documents
#   Runs metadata extraction on each
#   Outputs: list of usernames, software, paths
```

---

## 7. Forensic Image Analysis

### 7.1 Error Level Analysis (ELA)

ELA detects whether an image has been digitally altered. JPEG images are
compressed. When an area is edited and re-saved, it compresses at a different
rate — ELA makes this difference visible.

```
URL: 29a.ch/photo-forensics/#error-level-analysis
     fotoforensics.com

Usage:
  1. Upload the image
  2. ELA re-saves the image at a known compression level
  3. The difference between original and re-saved is shown
  4. Areas that look BRIGHTER in the ELA view = more recently modified
  5. Areas that look DARKER = less modified (original content)

Interpretation:
  Uniform texture throughout → likely unmodified
  Bright patches in specific areas → those areas were edited
  Text or objects with much higher ELA value than background → added/composited
```

### 7.2 Metadata Inconsistency

```
Check for inconsistencies between metadata fields:

EXIF says:   Date taken = 2024-03-15 (winter in northern hemisphere)
Image shows: Trees in full bloom (spring/summer)
→ Date is wrong, or image is from southern hemisphere

EXIF says:   Camera = iPhone 15 Pro
Image shows: File size of 4.2 MB, resolution 4032x3024
→ Plausible for an iPhone 15 Pro (consistent)

EXIF says:   GPS = Mumbai, India
Image shows: European-style road signs and vehicles
→ GPS data may have been faked or image is misattributed

Software field says: GIMP 2.10 (photo editor)
→ Image has been opened and saved in an editor — may have been modified
```

### 7.3 Clone Detection

Clone detection finds if parts of an image have been copied and pasted within
the same image (a common forgery technique).

```
Tool:    29a.ch/photo-forensics/#clone-detection
         fotoforensics.com

Usage:
  Upload image → run Clone Detection
  Tool highlights regions that appear multiple times in the image
  Circular patterns of clones = strong evidence of manipulation
```

---

## 8. Geolocation Workflow — Full Example

A structured example of geolocating an image step by step:

```
Image: A photo of a street scene, EXIF stripped

Step 1: Initial Analysis
  ├── Left-hand traffic visible → India, UK, Australia, Japan (etc.)
  ├── Auto-rickshaws visible → India, Bangladesh, Sri Lanka, some SE Asia
  ├── License plates: white with state code "MH" → Maharashtra, India
  ├── Script on signs: Devanagari → Hindi → confirms India
  └── Yellow-topped vehicles (BEST buses) → Mumbai specifically

→ CONCLUSION: Mumbai, India. Narrowed to city level.

Step 2: Narrow to Area
  ├── Visible landmark: Gateway of India arch in distant background
  ├── Wide waterfront road with palm trees
  └── Colonial-style buildings on right

→ CONCLUSION: Colaba area, Mumbai. Narrowed to neighborhood level.

Step 3: Pinpoint
  ├── Open Google Maps → navigate to Colaba, Mumbai
  ├── Enable Street View → navigate along Apollo Bunder road
  ├── Match: road width, building facades, palm tree positions, road markings
  └── Exact match found at: 18.9220° N, 72.8347° E (near Gateway of India)

Step 4: Verify
  ├── SunCalc: set date to estimated date, location to 18.9220° N, 72.8347° E
  ├── Shadow direction in image matches sun azimuth at approximately 10:00 AM
  └── Consistent with the image content (morning, not harsh midday light)

→ FINAL: Photo taken at approximately 18.9220° N, 72.8347° E,
          facing north, approximately 10:00 AM local time.
```

---

## 9. Useful GeoINT Reference Tools

| Tool | URL | Purpose |
|---|---|---|
| **Google Maps / Street View** | maps.google.com | Primary geolocation matching |
| **Google Earth Pro** | earth.google.com | Historical satellite imagery (free desktop app) |
| **Bing Maps** | bing.com/maps | Alternative aerial/satellite view |
| **Sentinel Hub** | sentinel-hub.com | ESA satellite imagery (free account) |
| **SunCalc** | suncalc.org | Sun angle and shadow analysis |
| **ShadowCalculator** | shadowcalculator.eu | Shadow length analysis |
| **Overpass Turbo** | overpass-turbo.eu | OpenStreetMap feature querying |
| **WiGLE** | wigle.net | Wi-Fi SSID geolocation database |
| **GeoGuessr** | geoguessr.com | Geolocation training (game) |
| **Mapillary** | mapillary.com | Crowd-sourced street-level imagery |
| **KartaView** | kartaview.org | Alternative crowd-sourced street images |
| **Pic2Map** | pic2map.com | EXIF GPS → Map |
| **Forensically** | 29a.ch/photo-forensics | ELA, clone detection, metadata |
| **FotoForensics** | fotoforensics.com | ELA and metadata analysis |
| **InVID/WeVerify** | weverify.eu | Video verification and keyframe extraction |

---

## 10. Level 3 Practice Tasks

### Task 1 — EXIF Extraction
- Find 3 images online (news photos, blog posts) that were NOT uploaded via social media
- Run ExifTool on each
- Note what metadata is present: date, device, GPS (if any), software
- Convert any GPS coordinates to a Google Maps URL

### Task 2 — Reverse Image Chain
- Take a profile picture from any public social media account (public figure)
- Run it through: Google Lens → Yandex → TinEye → Bing Visual
- Document what each engine found differently
- Follow any match to its source page and extract new intelligence

### Task 3 — GeoINT Challenge
- Find a geoINT challenge image from:
  - Bellingcat quizzes: bellingcat.com/category/resources/quizzes/
  - Sofia Santos OSINT exercises: gralhix.com/list-of-osint-exercises/
- Attempt to geolocate the image step by step using the methodology in Section 4
- Document every clue you used and the final coordinates

### Task 4 — Document Metadata
- Download any public PDF from a government website or corporate website
  (e.g., an annual report, a policy document)
- Run ExifTool on it
- Note: author name, software used, creation date, any internal paths
- What does the metadata reveal about the organization?

### Task 5 — Video Frame Analysis
- Find a news video on YouTube from any major news event
- Use InVID/WeVerify to extract keyframes
- Run 3 keyframes through reverse image search
- Verify: is the video being used in the correct context or has it been misattributed?

### Task 6 — GeoGuessr Training
- Play 10 rounds of GeoGuessr (geoguessr.com)
- For each round, write down the visual clues that led to your guess
- Score yourself and improve your visual cue recognition

---

## 11. Level 3 Summary

```
✅ Images have two layers: EXIF metadata (technical) and visual content (analytical)
✅ ExifTool extracts everything — GPS, device, date, software
✅ Social media strips EXIF; direct transfers often retain it
✅ Reverse image search: Google Lens + Yandex + TinEye + Bing (use all four)
✅ Yandex is the best engine for face matching
✅ TinEye is the best for finding the original/oldest source
✅ GeoINT: eliminate → narrow → pinpoint → verify
✅ Traffic direction, license plates, scripts, vegetation → country-level clue
✅ Street View + shadow analysis → exact location
✅ Videos: extract frames with ffmpeg, analyze audio, use InVID
✅ Documents: ExifTool + FOCA/metagoofil reveal employee names, paths, software
✅ ELA detects image manipulation

When you are done here → Proceed to Level 4: Organisation OSINT
```

---

## 12. Tools Quick Reference

| Tool | Purpose | Install | Command |
|---|---|---|---|
| **ExifTool** | Extract all file metadata | `apt install exiftool` | `exiftool image.jpg` |
| **ffmpeg** | Extract video frames | `apt install ffmpeg` | `ffmpeg -i video.mp4 -vf fps=1 frames/frame_%04d.jpg` |
| **yt-dlp** | YouTube metadata + download | `pip3 install yt-dlp` | `yt-dlp --write-info-json URL` |
| **Whisper** | Audio transcription | `pip3 install openai-whisper` | `whisper video.mp4` |
| **metagoofil** | Document metadata from domain | `apt install metagoofil` | `metagoofil -d example.com -t pdf,doc` |
| **Google Lens** | Image + object reverse search | Web | images.google.com |
| **Yandex Images** | Face + reverse image search | Web | yandex.com/images |
| **TinEye** | Find original/oldest image source | Web | tineye.com |
| **PimEyes** | Face search engine | Web (paid) | pimeyes.com |
| **InVID/WeVerify** | Video verification + keyframes | Browser extension | weverify.eu |
| **SunCalc** | Sun angle + shadow analysis | Web | suncalc.org |
| **Forensically** | ELA + clone detection | Web | 29a.ch/photo-forensics |
| **FotoForensics** | ELA + metadata analysis | Web | fotoforensics.com |
| **FOCA** | Mass document metadata (Windows) | Download | github.com/ElevenPaths/FOCA |
| **Overpass Turbo** | Query OpenStreetMap features | Web | overpass-turbo.eu |
| **Sentinel Hub** | Satellite imagery | Web | sentinel-hub.com |
