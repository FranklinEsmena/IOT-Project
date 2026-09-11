# IoT Crowd Sensor Demo

A live, in-browser demo built for BAANEL4 (Pair 2: IoT Analytics). Classmates scan a QR code, their phone becomes a live motion sensor, and a shared dashboard aggregates everyone's readings in real time.

## What this actually is

One single HTML file (`index.html`) with no build step, no frameworks, no npm install — just plain HTML/CSS/JavaScript that runs directly in a browser. It talks to a Firebase Realtime Database in the cloud so multiple phones can share live state with each other and with a dashboard screen.

Opening the file shows a role picker with two choices:
- **I'm a sensor** — turns that phone into a live motion sensor, sending an activity score (0–100) and a step count to the shared database every couple of seconds.
- **Show the live dashboard** — shows everyone currently connected, aggregate stats, a live trend line, and a per-phone breakdown.

## Live demo

https://iotdemov7.netlify.app

A matching QR code (`qr-code.png`) is included in this repo, pointing to the URL above. **If this URL ever changes** (redeploying via Netlify Drop tends to generate a new one — see "Deploying" below), this line and the QR code image both need to be updated together, or they'll point to different things.

## How it works, briefly

- **Sensor page:** reads the phone's motion/orientation sensors (with a graceful fallback to a manual slider if a phone can't or won't share sensor data), and writes a small JSON reading to Firebase every ~2 seconds under its own unique key.
- **Dashboard page:** subscribes live to all sensor readings in Firebase (push-based, not polling) and renders the aggregate.
- **Firebase config is already baked into `index.html`.** You don't need to create your own Firebase project to run or test this — it's already wired up. (If you ever do want your own separate project, see "Firebase notes" below.)

## Running it locally

Just open `index.html` in any browser. The UI and Firebase connection both work fine opened directly as a local file — the one thing that *won't* work locally is the motion sensor itself, since phones only grant sensor permissions to pages served over HTTPS, not local files. To test the actual sensor behavior, it needs to be hosted (see below).

## Deploying

Currently deployed via **Netlify Drop** (app.netlify.com/drop) — drag `index.html` onto the page and it gives you a public URL in seconds. No account required.

**Known annoyance, worth knowing up front:** dropping a file into Netlify Drop this way tends to generate a brand-new random URL almost every time, rather than updating the same one. That means every time either of us changes the code and redeploys, we've been generating a fresh QR code to match. If this gets annoying, GitHub Pages is a better long-term option since it gives one stable URL for the whole repo — worth switching to once the core features stop changing so often.

## Firebase notes

- The database is currently in **test mode**, meaning open read/write with no login required — that's intentional, it's what lets any classmate's phone write to it without an account. Test mode auto-expires 30 days after the database was created; after that it locks down and writes will start failing silently until the rules are reopened in the Firebase console.
- Every sensor phone writes to its own key under `sensors/<random-id>`, so multiple phones never overwrite each other.
- A phone's entry automatically deletes itself when that tab closes (`onDisconnect`), so the dashboard's "connected" count reflects who's actually still there.
- To manually clear all current readings back to zero: open the Firebase console → Realtime Database → Data tab → delete the `sensors` node. There's also a "Reset" button built into the dashboard itself that does the same thing.

## Known limitations (please read before changing the step counter)

- **Step counting is experimental and roughly calibrated, not precise.** It went through several iterations trying to reject false positives (random shaking, horizontal swinging) using increasingly specific signals — timing rhythm, then gravity direction, then phone rotation rate — and each round fixed the previous failure mode while occasionally introducing new edge cases. Treat any further accuracy complaints as expected territory, not necessarily a new bug.
- **Network type detection was tried and removed.** The browser API for this doesn't reliably distinguish Wi-Fi from cellular in current browsers, so this feature was cut rather than shipped broken.
- **Microphone/ambient noise was tried and removed.** It didn't work reliably on iOS during testing, so it was cut rather than shipped as a dead button.
- **Network resilience:** built for a small class (roughly a dozen phones); hasn't been tested at larger scale.

## Version history

| Version | What changed |
|---|---|
| v1 | Original design: shared storage, tilt-based activity score, iOS-safe permission flow with manual fallback |
| v2 | Switched from Claude's built-in artifact storage to Firebase Realtime Database, so it works on any external host |
| v3 | Added peak activity, connected time, room status, sustained-activity flag, reset button |
| v4 | Added network type + microphone/noise tracking |
| v5 | Removed network type and microphone (didn't work reliably) |
| v6 | Added step counter (rhythm-based detection) |
| v7 | Step counter fix #1: gravity-direction check (rejects horizontal swinging) |
| v7 | Step counter fix #2: rotation-rate check (rejects hand-swinging in any direction) — **current version** |

## If you're picking this up for the first time

1. Open `index.html` locally first, just to see the UI (sensor prompts won't fully work locally, that's expected).
2. Deploy it via Netlify Drop to get a real URL.
3. Test on your own phone — sensor page, dashboard, and disconnect behavior — before trusting it for anything live.
4. If you're testing on iOS specifically, read the "Known limitations" section above first — several rounds of iteration already happened there, so check the version history before assuming something's newly broken.
