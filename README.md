# SakayKita 🚍

**An offline, on-device AI commute companion for blind and low-vision Filipino commuters.**
ParaPo watches the street, reads the route signboards of approaching jeepneys, and tells you out loud which one is yours. On board, it buzzes when you're near your stop. It all runs on your phone, with no internet needed.

> Built for **AppBuildersPH Hackathon 2026: Local AI** ("Build an AI product that remains genuinely useful when the cloud disappears").

---

## Table of Contents

- [Why ParaPo](#why-parapo)
- [What it does](#what-it-does)
- [Why the AI has to run on the phone](#why-the-ai-has-to-run-on-the-phone)
- [How it works](#how-it-works)
- [Scope](#scope)
- [Tech stack](#tech-stack)
- [Getting started](#getting-started)
- [Demo script](#demo-script)
- [Safety and limitations](#safety-and-limitations)
- [Roadmap](#roadmap)
- [Disclosures (hackathon rules)](#disclosures-hackathon-rules)
- [References](#references)
- [Team](#team)
- [License](#license)

---

## Why ParaPo

To ride a jeepney, you first have to flag it down, and to flag it down, you have to know it's the right one. Jeepneys have no fixed stops, and their routes are shown only on **visual signboards**. For a blind or low-vision commuter, that means relying on strangers, guessing, or waiting.

- An ABS-CBN News series followed visually impaired workers commuting along Commonwealth Avenue. One was nearly hit by a motorcycle while waiting for a jeepney, and it took them about **half an hour** to hail one. She described having to ask strangers for help to get a ride.
- In US research on blind and vision-impaired transit users, **70%** said finding where to board a bus was "somewhat difficult" or harder, and those are fixed-stop buses.
- A Philippine study cites roughly **272,000 visually impaired Filipinos**.

Getting off is the second problem: knowing when you've reached your stop when you can't see the landmarks outside.

"Para po!" is what Filipino commuters say to stop a jeep. ParaPo helps with both moments: **getting on the right jeep, and getting off at the right place.**

## What it does

### 1. Jeepney route reader ("Is this my jeep?")
- Point the phone's camera toward the street (hand-held, lanyard, or chest mount).
- ParaPo detects approaching jeepneys, reads the **route signboard**, and matches it against a list of known routes.
- It speaks the result: *"Philcoa jeep, approaching. This is your route."*
- If you've set a destination, it only announces **matching** jeeps, so you're not flooded with every vehicle that passes.
- When it's unsure, it says so (*"Jeep approaching, route unclear"*) instead of guessing.

### 2. Stop alert ("Tell me when I'm near")
- Say or type your destination once (e.g., *"Philcoa"*).
- During the ride, ParaPo tracks your location using the phone's GPS. GPS does **not** need mobile data.
- As you approach your stop, the phone **vibrates and plays a sound**: an early "get ready" alert, then an "arriving now" alert, so you have time to say *"Para po."*

## Why the AI has to run on the phone

| Reason | Why it matters for this user |
|---|---|
| **Speed** | A jeepney passes in seconds. The alert is only useful *before* it passes, and a cloud round trip on roadside mobile data is too slow and unreliable. |
| **No data needed** | Many users are on prepaid plans or have weak signal. ParaPo works in airplane mode. |
| **Privacy** | The camera continuously sees the street, other people, and plate numbers. That video never leaves the device. |
| **Cost** | No per-request API fees, so the app can stay free for the people who need it. |

## How it works

```
Camera frames
     │
     ▼
[1] Vehicle detection ── is a jeepney in view? (on-device object detection)
     │
     ▼
[2] Signboard text reading ── read the route text on the placard (on-device OCR)
     │
     ▼
[3] Route matching ── fuzzy-match the text against a known route list
     │                 (e.g., "CUB_O" → "Cubao"), with a confidence score
     ▼
[4] Decision ── does it match the user's destination? Is confidence high enough?
     │
     ▼
[5] Feedback ── speech (Filipino/English) + vibration

Destination (voice/text) ──► on-device speech recognition ──► match to stop list
GPS location ──► offline geofence around the stop ──► "get ready" / "arriving" alerts
```

**Why route matching instead of free-form reading:** jeepney signboards are cluttered, and text on a moving vehicle is often blurry or partly hidden. Matching the noisy reading against a fixed list of route names makes the system far more tolerant of errors. Published work on bus-route recognition for visually impaired users uses the same idea, correcting OCR results against a predefined route list with edit distance.

## Scope

### Goals for the hackathon MVP
1. Correctly announce matching jeepneys from a **fixed corridor route list** in a live demo, with the network fully disabled.
2. Give an audible and vibration "near your stop" alert from GPS, offline.
3. Never announce a confident **wrong** match. When unsure, say "unclear."

### Non-goals (for now)
- **Replacing the white cane or mobility training.** ParaPo is an aid, not a navigator.
- **Obstacle or traffic-danger detection.** Safety-critical; out of scope.
- **All jeepney routes nationwide.** We start with one corridor (e.g., Commonwealth Avenue).
- **Live jeepney tracking or arrival times.** Requires infrastructure most jeepneys don't have.
- **Buses, UV Express, tricycles.** Possible later, using the same pipeline.

### User stories
- As a **blind commuter waiting at the roadside**, I want to hear which jeep is approaching so that I can flag down the right one without asking strangers.
- As a **low-vision commuter**, I want ParaPo to stay quiet about jeeps that aren't mine so that I'm not overwhelmed by announcements in traffic.
- As a **rider already on board**, I want my phone to vibrate before my stop so that I have time to say "Para po" and get off safely.
- As a **commuter with no mobile data**, I want everything to work offline so that I can use it anywhere, anytime.

### Requirements

**P0: must have for the demo**
- [ ] On-device jeepney detection from the camera feed
- [ ] On-device OCR of the signboard region
- [ ] Fuzzy matching against a bundled route list, with a confidence threshold
- [ ] Spoken announcement plus vibration on a match
- [ ] "Route unclear" fallback below the confidence threshold
- [ ] Destination entry (text or voice) matched to a bundled stop list
- [ ] Offline GPS geofence alert near the chosen stop
- [ ] Works fully in airplane mode after first load

**P1: nice to have**
- [ ] Saved "My routes" (e.g., home ↔ work)
- [ ] Filipino and English voice options
- [ ] Large-button, screen-reader-friendly UI (TalkBack-tested)
- [ ] Repeat-last-announcement gesture

**P2: future**
- [ ] More corridors and route packs (downloadable)
- [ ] Buses and UV Express
- [ ] Wearable/earbud-only mode

## Tech stack

> **Proposed** stack, chosen because the team is web-first. Update this as you build.

| Layer | Choice | Notes |
|---|---|---|
| App shell | **Progressive Web App** (installable, offline via service worker) | Android Chrome as the primary target |
| Vehicle detection | Small object-detection model (e.g., YOLO-family, ONNX) via **ONNX Runtime Web / Transformers.js** on **WebGPU/WASM** | Generic "bus/truck" classes as a starting proxy; fine-tune on jeepney photos if time allows |
| OCR | On-device OCR (e.g., **PaddleOCR via ONNX** or **Tesseract.js**) on the cropped signboard | Run only on detected vehicle crops to save compute |
| Route matching | Fuzzy string matching (Levenshtein / token similarity) against `routes.json` | Deterministic and fast; no LLM needed in the hot path |
| Destination input | On-device speech-to-text (e.g., **Whisper tiny** via Transformers.js), with a typed fallback | Browser built-in speech recognition may use the cloud, so don't rely on it offline |
| Speech output | Web Speech `speechSynthesis`, with **pre-recorded route-name audio** as fallback | Installed voices vary by device |
| Haptics | Vibration API | Supported on Android Chrome, not on iOS Safari |
| Location | Geolocation API + bundled stop coordinates (`stops.geojson`) | GPS works without data; the first fix can be slower offline |

## Getting started

> ⚠️ Placeholder commands. Replace them once the repo structure is final.

```bash
# 1. Clone
git clone https://github.com/<your-org>/parapo.git
cd parapo

# 2. Install dependencies
npm install

# 3. Download model files into /public/models (see models/README.md)
npm run fetch-models

# 4. Run locally (HTTPS is needed for camera, GPS, and service workers)
npm run dev
```

Then:
1. Open the app on an Android phone in Chrome and **Install** it (Add to Home Screen).
2. Open it once while online so models and route data are cached.
3. Turn on **airplane mode**. Everything should keep working.

### Project structure (planned)

```
parapo/
├── public/
│   ├── models/          # ONNX detection + OCR models (cached offline)
│   ├── audio/           # pre-recorded route-name clips (TTS fallback)
│   └── manifest.json
├── data/
│   ├── routes.json      # corridor route names + aliases (e.g., "Cubao", "CUBAO", "Cubao Ali Mall")
│   └── stops.geojson    # stop names + coordinates for the stop alert
├── src/
│   ├── detect/          # vehicle detection
│   ├── ocr/             # signboard OCR
│   ├── match/           # fuzzy route/stop matching + confidence
│   ├── speech/          # speech-to-text + speech output
│   ├── geo/             # GPS geofence + alerts
│   └── ui/              # accessible UI
└── sw.js                # service worker for offline use
```

### Adding a route

Add an entry to `data/routes.json`:

```json
{
  "id": "philcoa-quiapo",
  "display": "Philcoa – Quiapo",
  "aliases": ["PHILCOA", "QUIAPO", "PHILCOA QUIAPO", "QUIAPO PHILCOA"]
}
```

## Demo script

**Live, about 90 seconds**

1. Turn on **airplane mode** and show it on screen.
2. Set the destination by voice: *"Philcoa."*
3. Teammates walk past the camera holding printed jeepney signboards (or play street video on a laptop).
   - A non-matching jeep passes: ParaPo stays quiet, or gives a short "not yours."
   - The matching jeep passes: *"Philcoa jeep, approaching. This is your route."*, plus vibration.
4. Show the **"route unclear"** behavior with a partly covered sign.
5. Switch to ride mode with a simulated GPS track: the phone buzzes with *"Get ready, Philcoa is next,"* then *"Arriving now."*

## Safety and limitations

- **ParaPo supports the white cane. It never replaces it.** It does not detect obstacles, traffic, or danger.
- OCR on moving vehicles can fail in rain, at night, at odd angles, or with hand-painted signs. When confidence is low, ParaPo says so instead of guessing.
- Route and stop data cover **one corridor** in this version.
- GPS accuracy varies in dense areas and inside vehicles, so stop alerts include an early warning, not just a final one.
- This is a **hackathon prototype**. It has not yet been tested with blind and low-vision users, and that is our first next step.

## Roadmap

- [ ] Test with blind and low-vision commuters (via disability organizations and school student-services offices)
- [ ] Build a labeled jeepney signboard photo set for one corridor
- [ ] Measure and publish accuracy on that set (we will not claim numbers we haven't measured)
- [ ] Downloadable route packs for more cities
- [ ] Earbud-first, hands-free mode

## Disclosures (hackathon rules)

> Fill this in before submission. The rules require disclosure of every model, framework, API, AI coding tool, and any pre-existing code.

| Type | Name | Version / Source | Used for |
|---|---|---|---|
| Model | _e.g., YOLO-family detector (ONNX)_ | _link_ | Vehicle detection |
| Model | _e.g., PaddleOCR / Tesseract.js_ | _link_ | Signboard OCR |
| Model | _e.g., Whisper tiny_ | _link_ | Destination speech-to-text |
| Framework | _e.g., ONNX Runtime Web / Transformers.js_ | _link_ | On-device inference |
| API | Web Speech, Vibration, Geolocation | Browser built-in | Speech, haptics, location |
| AI coding tools | _list any (e.g., Claude, Copilot, Devin)_ | | Development assistance |
| Pre-existing code/assets | _none / list_ | | |

**Cloud usage:** none in the core flow. Any optional cloud feature must be listed here and must not be required for the app to work.

## References

- ABS-CBN News: [How's your commute? A journey from the perspective of a visually impaired person](https://news.abs-cbn.com/spotlight/multimedia/slideshow/07/21/22/sidewalks-ped-xing-overpasses-commuting-from-the-perspective-of-a-blind-person) (2022)
- Marston, J. (UCSB): [Research on barriers to transit for vision-impaired travelers](https://people.geog.ucsb.edu/~marstonj/DIS/CH1_1.html)
- Bacalla et al.: [Braille signage wayfinding study, Cebu Normal University](https://www.jhe.cnu.edu.ph/ojs3/article/download/322/49) (cites ~272,527 visually impaired Filipinos)
- [Bus route number and destination recognition for visually impaired individuals](https://api.crossref.org/works/10.3390%2FA18100616), *Algorithms* (MDPI): OCR with route-list correction
- [Google Lookout overview](https://ixd.prattsi.org/2023/09/assistive-technology-google-lookout-assisted-vision/): a general-purpose reading app for comparison

## Team

| Name | Role |
|---|---|
| _Name_ | _e.g., ML / on-device inference_ |
| _Name_ | _e.g., Frontend / accessibility_ |
| _Name_ | _e.g., Data (routes, stops, test set)_ |
| _Name_ | _e.g., Pitch / user research_ |

## License

_TBD (e.g., MIT)._ Make sure third-party model licenses are compatible and listed in [Disclosures](#disclosures-hackathon-rules).

---

*Para po!* 🙋
