# SakayKita 🚍

**An offline, on-device AI commute companion for blind and low-vision Filipino commuters.**
You choose your commute. SakayKita watches the street, reads the route signboards of approaching jeepneys, ignores the ones that aren't yours, and tells you out loud when your jeep is coming, early enough to flag it down. It all runs on the phone, with no internet needed.

> Built for **AppBuildersPH Hackathon 2026: Local AI** ("Build an AI product that remains genuinely useful when the cloud disappears").
> **Status: hackathon prototype.** Not yet tested with blind or low-vision users.

---

## Table of Contents

- [Why SakayKita](#why-sakaykita)
- [What it does](#what-it-does)
- [What's different from existing tools](#whats-different-from-existing-tools)
- [Why the AI has to run on the phone](#why-the-ai-has-to-run-on-the-phone)
- [How it works](#how-it-works)
- [Scope](#scope)
- [How we measure it](#how-we-measure-it)
- [Tech stack](#tech-stack)
- [Getting started](#getting-started)
- [Pre-demo test checklist](#pre-demo-test-checklist)
- [Demo plan](#demo-plan)
- [Known limitations](#known-limitations)
- [Roadmap](#roadmap)
- [Disclosures (hackathon rules)](#disclosures-hackathon-rules)
- [References](#references)
- [Team](#team)
- [License](#license)

---

## Why SakayKita

To ride a jeepney, you first have to flag it down, and to flag it down you have to know it's the right one. Jeepneys have no fixed stops, and their routes are shown only on **visual signboards**.

- An ABS-CBN News series followed visually impaired workers commuting along Commonwealth Avenue. One was nearly hit by a motorcycle while waiting for a jeepney, and it took them about **half an hour** to hail one. She described having to ask strangers for help to get a ride.
- In US research on blind and vision-impaired transit users, **70%** said finding where to board a bus was "somewhat difficult" or harder, and those are fixed-stop buses.
- A Philippine study cites roughly **272,000 visually impaired Filipinos**.

"Sakay" is Filipino for "ride." SakayKita helps you get on the right one.

## What it does

### Core: "Is my jeep coming?"
1. **Choose your commute** without looking at the screen: a boarding spot, a direction, and your route (from 3–5 verified routes for that spot).
2. **Start scanning.** SakayKita confirms out loud that the camera is working and keeps a soft periodic tick so you know it's still scanning.
3. When a jeepney on **your** route approaches, SakayKita announces it (*"Philcoa jeep, approaching"*) with a distinct tone.
4. Jeeps on other routes are ignored, or get a short low tone if you turn that on.
5. If SakayKita can't read a sign clearly, it says *"Jeep approaching, unclear"* instead of guessing.
6. **Repeat** the last announcement anytime with one large gesture.

### Optional (only if it passes on-phone testing): stop alert
Once on board, SakayKita uses GPS to alert you as you near your stop. Haptics work on phones only. See [Known limitations](#known-limitations) for the screen-on requirement.

## What's different from existing tools

Reading signs is **not** new. Google Lookout already reads text and identifies objects for blind and low-vision users, and published research covers bus-route recognition. We don't claim sign reading as our innovation.

SakayKita's difference is the **decisions** around reading:

| | General reader (e.g., Lookout) | SakayKita |
|---|---|---|
| What it reads | Any text in view (ads, plates, stickers) | Only jeepney route signboards |
| What it says | The raw text | "This is your jeep" / "unclear" / silence |
| Commute context | None | Knows your boarding spot, direction, and route |
| Local route knowledge | None | Verified routes for a specific boarding area |
| Refusing to guess | Not applicable | Says "unclear" rather than risk a wrong alert |
| Timing | On demand | Designed to alert **before** the jeep passes |

In the demo, we show these decisions, **including when SakayKita refuses to guess.**

## Why the AI has to run on the phone

| Reason | Why it matters |
|---|---|
| **Timing** | At 30 km/h a vehicle covers 15 metres in about 1.8 seconds. Reading, deciding, speaking and the user reacting must all fit before it passes, so there's no room for a cloud round trip on roadside mobile data. |
| **No data needed** | Works in airplane mode, for users on prepaid plans or with weak signal. |
| **Privacy** | The camera continuously sees the street, other people, and plate numbers. That video never leaves the device. |
| **Cost** | No per-request API fees, so it can stay free. |

On-device text recognition plus route matching is the Local AI core. **No language model is needed** in the main pipeline.

## How it works

```
Camera frames
     │
     ▼
[1] Vehicle detection ── jeepney-like vehicle in view?  (on-device object detection)
     │   └─ FAIL-OPEN: if nothing is detected, still OCR a fixed region
     ▼      (the area where signboards appear from the boarding spot)
[2] Signboard text reading ── on-device OCR on the crop / region
     │
     ▼
[3] Route matching ── fuzzy-match text to the 3–5 verified routes for this
     │                 boarding spot + direction, with a confidence score
     ▼
[4] Agreement check ── require the same route across several consecutive frames;
     │                 reject readings that are ambiguous between two routes
     ▼
[5] Decision ── YOUR ROUTE / OTHER ROUTE / UNCLEAR
     │
     ▼
[6] Feedback ── spoken announcement + distinct tones (+ vibration on phones)
                + large high-contrast visual cue
```

**Why fail-open:** if detection is the gate and the detector misses a jeepney (they are not a standard object-detection class), OCR never runs and the whole pipeline goes silent. Fail-open keeps the reader running. The route-list match plus multi-frame agreement then filters out stray text such as ads or other signage, so fail-open doesn't turn into false alerts.

**Why a route list, not free reading:** signboards are cluttered and text on a moving vehicle is blurry or partly hidden. Matching a noisy reading against a small verified list is far more robust. Published bus-route recognition work uses the same idea (OCR plus route-list correction).

**Direction matters:** a sign containing "Philcoa" doesn't prove the jeep is going *your* way from *where you are*. So routes are tied to a specific boarding spot and direction, not matched on a single word.

## Scope

### MVP goals
1. At **one boarding location, one direction, with 3–5 verified routes**, announce the user's route early enough to act on, on the actual demo phone, fully offline.
2. Prefer "unclear" over a wrong alert, and **report wrong alerts honestly** (see [How we measure it](#how-we-measure-it)).
3. The whole core flow can be completed **without looking at the screen**.

### Non-goals (for now)
- **Replacing the white cane or mobility training.** SakayKita is an aid, not a navigator.
- **Obstacle or traffic-danger detection.** Safety-critical; out of scope.
- **Voice input for the destination.** Postponed: small on-device speech models are likely to garble Filipino place names such as "Philcoa."
- **Multiple corridors or nationwide routes.** One boarding location first.
- **Custom model training.** Postponed until we've tested off-the-shelf detection and OCR on real footage.
- **Language models.** Not needed for the core flow.

### User stories
- As a **blind commuter at my usual boarding spot**, I want to hear when my jeep is approaching, early enough to flag it, so that I don't have to ask strangers.
- As a **low-vision commuter**, I want SakayKita to stay quiet about jeeps that aren't mine so that I'm not overwhelmed in traffic.
- As a **screen-reader user**, I want to set up my commute and start scanning entirely with TalkBack so that I never need to see the screen.
- As a **user holding the phone**, I want to know whether the camera is actually aimed at the street so that I can trust silence.

### Requirements

**P0: must have for the demo**
- [ ] **Accessible setup with TalkBack:** choose boarding spot → direction → route, start/stop scanning, all with large targets and spoken labels
- [ ] **Scanning status by audio:** spoken "scanning started," periodic tick while running, and a spoken warning if the camera sees no street or road (e.g., aimed at the ground or sky)
- [ ] **Camera positioning guidance:** a spoken setup step plus a recommended mount (lanyard or chest strap, rear camera facing the road)
- [ ] On-device detection, with **fail-open** to a fixed OCR region
- [ ] On-device OCR
- [ ] Route matching against the verified list for the chosen spot and direction, with multi-frame agreement and rejection of ambiguous readings
- [ ] Three outcomes with distinct audio: **your route / other route / unclear**
- [ ] **Repeat last announcement** with one large gesture
- [ ] Audio + large visual cue for every alert (haptics as a phone-only extra)
- [ ] Works fully in airplane mode after first load
- [ ] Runs on WebGPU **and** falls back to WASM

**P1: nice to have**
- [ ] Stop alert via GPS (only if it passes on-phone testing; see limitations)
- [ ] Saved commutes (e.g., home → work)
- [ ] Filipino and English voices
- [ ] Voice destination input

**P2: future**
- [ ] More boarding locations and route packs
- [ ] Buses and UV Express
- [ ] Earbud-first, hands-free mode
- [ ] Custom jeepney detection model

## How we measure it

We report each outcome **separately**. An app that always says "unclear" could have zero wrong alerts and still be useless, so no single number tells the story.

For each test pass (recorded footage and live camera), we count:

| Metric | Definition |
|---|---|
| **Correct alerts** | Announced "your route" and it was your route |
| **Wrong alerts** | Announced "your route" but it wasn't |
| **Missed vehicles** | Your route passed and SakayKita said nothing or "unclear" |
| **Unclear results** | Said "unclear" (for any vehicle) |
| **Lead time** | Seconds between the "your route" announcement and the jeep passing the user, measured on the actual phone |

We will only publish numbers we have measured, along with how many vehicles were in the test.

## Tech stack

> **Proposed** stack (web-first team). Items marked 🔬 must pass the [pre-demo tests](#pre-demo-test-checklist) before we rely on them.

| Layer | Choice | Notes |
|---|---|---|
| App shell | **Progressive Web App** (installable, offline via service worker) | Android Chrome is the primary target |
| Vehicle detection 🔬 | Small COCO-trained detector (e.g., YOLO-family, ONNX) via **ONNX Runtime Web** | Jeepneys are **not** a COCO class. We test whether `bus`/`truck` fire on jeepneys; if not, rely on fail-open OCR |
| OCR 🔬 | On-device OCR (e.g., **PaddleOCR via ONNX** or **Tesseract.js**) | Run on detected crops or the fixed fail-open region |
| Route matching | Fuzzy string matching (Levenshtein / token similarity) + multi-frame agreement | Deterministic and fast |
| Inference backend 🔬 | WebGPU with **WASM fallback** | Venue laptops and phones may lack WebGPU |
| Speech output | **Pre-recorded audio clips** for route names and outcomes, with Web Speech `speechSynthesis` as a backup | Bundled clips don't depend on installed voices |
| Haptics | Vibration API | **Phone only.** Not available on desktop browsers or iOS Safari |
| Screen-on | Screen Wake Lock API | Keeps scanning running while the screen is on |
| Location (P1) 🔬 | Geolocation API + bundled stop coordinates | Only if background/locked-screen behavior is acceptable |

## Getting started

> ⚠️ Placeholder commands. Replace them once the repo structure is final.

```bash
git clone https://github.com/<your-org>/sakaykita.git
cd sakaykita
npm install
npm run fetch-models   # downloads ONNX models into /public/models
npm run dev            # HTTPS needed for camera, GPS, wake lock, service worker
```

1. Open on an Android phone in Chrome and **Install** (Add to Home Screen).
2. Open once while online so models and route data are cached.
3. Turn on **airplane mode**. Everything in the core flow should keep working.

### Adding a boarding spot and its routes

`data/spots.json`:

```json
{
  "id": "commonwealth-philcoa-southbound",
  "name": "Commonwealth Ave near Philcoa, southbound",
  "direction": "southbound",
  "ocr_region": { "x": 0.1, "y": 0.15, "w": 0.8, "h": 0.35 },
  "routes": [
    {
      "id": "philcoa-quiapo",
      "display": "Philcoa – Quiapo",
      "required_terms": ["QUIAPO"],
      "aliases": ["PHILCOA QUIAPO", "QUIAPO"],
      "verified": true
    }
  ]
}
```

- `required_terms` is what must be read for a match, chosen so that this route can't be confused with the other routes **at this spot and direction**.
- Only include routes the team has **verified in person** at that spot.

## Pre-demo test checklist

Run these **in this order**. If step 1 fails, change the plan before building further.

1. [ ] **Detector on jeepneys (do this first).** Run the COCO detector on 20+ real jeepney photos and video clips. Record whether `bus`, `truck`, or nothing fires. If it rarely fires, rely on fail-open OCR and say so in the pitch.
2. [ ] **OCR on real signboards.** Same footage: can OCR read the route terms at all? At what distance?
3. [ ] **Lead time on moving footage, on the actual phone.** Measure seconds between the announcement and the jeep passing. If it only works close up, narrow to a boarding area where jeeps slow down or stop.
4. [ ] **WebGPU vs WASM.** Run on the actual demo phone and **every teammate's laptop tonight**, not tomorrow. Confirm the WASM fallback works.
5. [ ] **Eyes-free run.** A teammate completes setup → scan → hear alert → repeat, **without looking at the screen**, using TalkBack. (A useful check, but not a substitute for testing with blind users.)
6. [ ] **Airplane mode.** Full core flow with all networks off.
7. [ ] **(P1) Stop alert with the screen locked or app in the background.** Chrome can suspend background pages. If the alert only works with the screen on, document that and keep it out of the main claim.

## Demo plan

The demo must prove the **full claim**, not just the reading step.

1. **Airplane mode on**, shown on screen. The phone screen is mirrored to the projector (e.g., via `scrcpy`).
2. **Eyes-free setup:** choose spot → direction → route using TalkBack, with audio on.
3. **Recorded jeepney footage, clearly labelled as recorded,** played into the phone camera or the app's video input. Live inference runs on it. It includes:
   - a **wrong-route** jeep → SakayKita stays quiet / low tone
   - an **unclear** sign → "Jeep approaching, unclear"
   - the **correct route** → "Philcoa jeep, approaching" + tone + big visual cue
4. **Live camera example:** a printed signboard in front of the phone camera to show real-time reading. We say openly that a hand-held sign may not trigger the vehicle detector, which is why fail-open exists.
5. **Show the measured results** (correct / wrong / missed / unclear / lead time) from our test footage.
6. **(If kept) Stop alert:** a simulated GPS track, **clearly labelled as simulated**, plus a separate screen recording of a real offline GPS test.
7. **Haptics:** say out loud that vibration is phone-only. On the projector, the alert appears as an audio countdown plus a large visual cue.

## Known limitations

- **SakayKita supports the white cane. It never replaces it.** It does not detect obstacles, traffic, or danger.
- **Coverage:** one boarding location, one direction, 3–5 verified routes.
- **Wrong alerts are possible.** We reduce them with route-list matching, multi-frame agreement, and rejecting ambiguous readings, and we report them. We do not claim zero.
- **OCR conditions:** rain, night, glare, angles, and hand-painted signs reduce accuracy.
- **Detection:** jeepneys are not a standard detection class. Detection may miss them, which is why the pipeline is fail-open.
- **Haptics** are phone-only (not desktop browsers or iOS Safari).
- **Background behavior:** Chrome can suspend background pages. Scanning (and any stop alert) requires the app open with the screen on (wake lock) unless testing shows otherwise.
- **Not yet tested with blind or low-vision users.**

## Roadmap

- [ ] Test with blind and low-vision commuters (via disability organizations and school student-services offices)
- [ ] Build a labelled signboard dataset for the first boarding location
- [ ] Publish measured correct / wrong / missed / unclear rates and lead times
- [ ] Add boarding locations, then route packs
- [ ] Evaluate a custom jeepney detector
- [ ] Re-evaluate voice input with a stronger on-device Filipino speech model

## Disclosures (hackathon rules)

> Fill this in before submission. The rules require disclosure of every model, framework, API, AI coding tool, and any pre-existing code.

| Type | Name | Version / Source | Used for |
|---|---|---|---|
| Model | _e.g., YOLO-family detector (ONNX)_ | _link + license_ | Vehicle detection |
| Model | _e.g., PaddleOCR / Tesseract.js_ | _link + license_ | Signboard OCR |
| Framework | _e.g., ONNX Runtime Web_ | _link_ | On-device inference |
| API | Vibration, Wake Lock, Geolocation, Web Speech | Browser built-in | Haptics, screen-on, location, speech fallback |
| Footage | _source of recorded jeepney video_ | _own recording / permission_ | Demo + testing |
| AI coding tools | _list any (e.g., Claude, Copilot, Devin)_ | | Development assistance |
| Pre-existing code/assets | _none / list_ | | |

**Cloud usage:** none in the core flow.

## References

- ABS-CBN News: [How's your commute? A journey from the perspective of a visually impaired person](https://news.abs-cbn.com/spotlight/multimedia/slideshow/07/21/22/sidewalks-ped-xing-overpasses-commuting-from-the-perspective-of-a-blind-person) (2022)
- Marston, J. (UCSB): [Research on barriers to transit for vision-impaired travelers](https://people.geog.ucsb.edu/~marstonj/DIS/CH1_1.html)
- Bacalla et al.: [Braille signage wayfinding study, Cebu Normal University](https://www.jhe.cnu.edu.ph/ojs3/article/download/322/49) (cites ~272,527 visually impaired Filipinos)
- [Bus route number and destination recognition for visually impaired individuals](https://api.crossref.org/works/10.3390%2FA18100616), *Algorithms* (MDPI)
- [Google Lookout overview](https://ixd.prattsi.org/2023/09/assistive-technology-google-lookout-assisted-vision/)

## Team

| Name | Role |
|---|---|
| _Name_ | _e.g., On-device inference (detection + OCR)_ |
| _Name_ | _e.g., Accessibility + TalkBack flow_ |
| _Name_ | _e.g., Route data, footage, measurements_ |
| _Name_ | _e.g., Pitch + demo_ |

## License

_TBD (e.g., MIT)._ Make sure third-party model licenses are compatible and listed in [Disclosures](#disclosures-hackathon-rules).

---

*Para po!* 🙋
