# Snowed-In Bus Stop

> AI-powered snow detection tool that uses NYC traffic cameras to proactively identify blocked bus stops before they're reported to 311.

## The Problem

Every winter, NYC bus riders — especially elderly and disabled New Yorkers — are stranded at stops buried under plowed snow. The MTA has no real-time visibility into which of its 16,000+ bus stops are blocked after a storm. Dispatching crews is guesswork, and riders have no way to report conditions at scale.

## What It Does

An MTA operator or 311 dispatcher opens an interactive map showing every NYC bus stop with a nearby traffic camera. The system automatically analyzes live camera feeds for snow accumulation at curb level and flags each stop as blocked, clear, or obscured. Dispatchers can click any flagged stop to see the source camera image with snow areas highlighted, then route a plow crew without waiting on rider complaints.

## Key Features

| Feature | What It Does |
|---------|-------------|
| Live Camera Feed | Pulls real-time images from 900+ NYC traffic cameras via the NYCTMC API |
| AI Snow Detection | Color-based image analysis detects snow accumulation on sidewalks and curbs |
| Interactive Map | Leaflet-powered map with color-coded markers for all monitored bus stops |
| Bus Stop Matching | Automatically pairs each camera to nearby MTA bus stops within a 100m radius |
| Snowplow Tracking | Cross-references NYC OpenData snowplow activity to show recent plow passes |
| Visual Annotations | Red box overlays highlight blocked areas for MTA dispatchers |

## Tech Stack

| Layer | Technology | Why |
|-------|-----------|-----|
| Frontend | HTML/CSS/JS | Lightweight UI with no build step — fast to prototype and deploy |
| Backend | Python, Flask | Simple REST API to proxy camera feeds and run image analysis server-side |
| Image Analysis | Pillow (PIL), NumPy | Efficient pixel-level color analysis without heavy ML dependencies |
| Maps | Leaflet.js | Open-source, lightweight mapping with easy marker customization |
| APIs | NYCTMC API, MTA Bus Time API, NYC OpenData | Official city/transit data ensures accurate, real-time camera and stop info |
| Deployment | Local / Python HTTP Server | Zero-config local demo; easily containerizable for production |

## Architecture

```
NYCTMC API → Flask Server → PIL/NumPy Analysis → Status Classification → Leaflet Map
```

The Flask server pulls live images from the NYCTMC camera feed, hands them to PIL/NumPy for pixel-level color analysis (HSV thresholds on the bottom 40% of each frame), classifies each camera view as blocked, clear, or obscured, and pushes the results to a Leaflet map that renders color-coded markers for every bus stop within 100m of a camera.

## Technical Decisions

**Color-based detection over an ML model**
An ML segmentation model would need labeled training data we didn't have and adds per-image latency. HSV color thresholds — high brightness (>200) with low saturation (<25) — run in milliseconds and are surprisingly accurate for fresh snow on dark pavement. For a hackathon-scoped project, accuracy-per-hour-of-build-time favored the simple approach.

**Bottom 40% image crop over full-frame analysis**
Traffic cameras point at intersections — the top 60% is sky, buildings, and moving traffic that would trigger false positives (white vehicles, bright signage, overcast sky). Cropping to the bottom 40% focuses analysis on the curb and sidewalk where snow actually blocks riders.

**100m radius bus stop matching over exact address lookup**
MTA stop data and NYCTMC camera data use different coordinate systems and naming conventions, so string-matching street names fails on abbreviations and formatting differences. A 100m radius spatial match is more resilient and balances precision (narrow enough to tie a camera to the right stop) with coverage (wide enough that most stops near a camera get monitored).

## Getting Started

### Prerequisites
- Python 3.8+
- MTA Bus Time API key (free — register at the [MTA Bus Time Developer Portal](http://bustime.mta.info/wiki/Developers/Index))

### Installation
```bash
git clone https://github.com/jonelrichardson/SnowedInMTABusStop.git
cd SnowedInMTABusStop
pip install -r requirements.txt
```

### Environment Variables
Copy the template and add your keys:
```bash
cp .env.example .env
```
```
MTA_API_KEY=your_key_here
HF_API_KEY=your_token_here  # optional, for future ML features
```

The NYCTMC and NYC OpenData endpoints are public and require no key. The HF token is only needed if you swap the color-threshold detector for a Hugging Face segmentation model.

### Run Locally
```bash
python3 server.py
# In another terminal:
python3 -m http.server 8080
# Visit http://localhost:8080/app.html
```

## What I'd Build Next

- **ML model upgrade**: Replace HSV thresholds with a fine-tuned segmentation model (e.g., Hugging Face SegFormer) trained on labeled winter traffic-camera frames to cut false positives from shadows and wet pavement
- **Historical snow pattern tracking**: Log per-stop blockage frequency across the season to surface chronic problem stops and inform pre-positioning of plow crews before a storm hits
- **MTA/311 API integration for automated dispatch**: Push detected blockages directly into the MTA dispatch system and NYC 311 so crews are routed without a human in the loop

## About This Project

Built during Pursuit's AI-Native Builder Fellowship (January 2026). Team of five.

**My role:** Team lead. Led project scoping, task delegation, and coordinated the five-person team from idea to working prototype.

---

Built by [Jonel Richardson](https://linkedin.com/in/jonel-richardson-09a399382)
