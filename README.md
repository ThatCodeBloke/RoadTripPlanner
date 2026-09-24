# Roadbook

A trip planner for the road trip — route legs, day-by-day plan, budget. One file, no build step.

## 1. Put it on GitHub Pages

1. Create a new **public** GitHub repo (e.g. `roadbook`).
2. Add `index.html` to the root of the repo (drag-and-drop upload on github.com works fine, or `git push` it).
3. Go to **Settings → Pages**. Under "Build and deployment", set **Source: Deploy from a branch**, **Branch: main**, folder **/ (root)**. Save.
4. Wait a minute, then your site is live at:
   `https://<your-github-username>.github.io/<repo-name>/`

That's it — anyone with the link can open it and plan a trip. By default, each person's trips are saved only in their own browser (`localStorage`), so you and your mate won't see each other's edits unless you add live sync (below).

## 2. Optional: live sync between you and your mate

This uses [Firebase](https://firebase.google.com)'s free tier (no credit card needed) as a shared database.

1. Go to the [Firebase console](https://console.firebase.google.com), click **Add project**, give it any name, and skip Google Analytics if you don't want it.
2. In the project, open **Build → Firestore Database → Create database**. Choose **Start in test mode**, pick a region close to you, and create it.
3. Click the **gear icon → Project settings**, scroll to **Your apps**, click the **`</>`** (web) icon, register an app with any nickname (no hosting needed). Firebase shows you a `firebaseConfig` object — copy it.
4. Open `index.html` in your repo, find this near the top of the `<script>` block:
   ```js
   const FIREBASE_CONFIG = null;
   ```
   Replace `null` with the config object you copied, e.g.:
   ```js
   const FIREBASE_CONFIG = {
     apiKey: "AIza...",
     authDomain: "your-project.firebaseapp.com",
     projectId: "your-project",
     storageBucket: "your-project.appspot.com",
     messagingSenderId: "...",
     appId: "..."
   };
   ```
5. Commit and push the change. GitHub Pages redeploys automatically within a minute or two.
6. Send your mate the same GitHub Pages link — you'll now both see the same trip list, live.

### About Firestore's "test mode"

Test mode leaves the database open to *anyone who has your config values* (which are visible in your page's source — that's normal for Firebase web apps, but worth knowing). It auto-expires 30 days after creation as a safety net. Since this only ever holds trip-planning data, the simplest fix when that happens (or straight away) is:

1. In the Firebase console, go to **Firestore Database → Rules**.
2. Replace the rule with:
   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /{document=**} {
         allow read, write: if true;
       }
     }
   }
   ```
3. Click **Publish**. This keeps it open indefinitely — fine for two people planning a road trip, but don't reuse this project for anything sensitive.

## What's in it

- **Currencies** — add as many as the trip needs (chip list at the top); the first one you add is "home" and is what the grand total is shown in. Add a second currency (e.g. EUR) and an exchange-rate row appears, pre-filled from a live rate where possible (via [Frankfurter](https://frankfurter.dev), free/keyless), editable by hand.
- **Roadbook** — the route, leg by leg, with mode, distance, duration, notes and cost. Each leg has its own currency — auto-detected from the destination country once it's been geocoded (via the Map's "Update map" or the leg's own ↻ button), overridable any time. The ↻ button also looks up real road distance/time for drive legs via [OSRM](https://project-osrm.org) (free, keyless), or a straight-line distance for ferry/tunnel/flight legs, since real crossing/flight duration isn't something a free API can tell you. Ferry/Eurotunnel legs also get a "Check current prices" link to a live search for that crossing.
- **Map** — every stop on the route plotted on an OpenStreetMap map (via [Leaflet](https://leafletjs.com), free, no key needed), connected in order. Click "Update map" after adding or renaming legs to look up any new places and auto-assign currencies — locations are cached on the trip itself, so it's instant after the first lookup, including for whoever you're syncing with.
- **Day by day** — generated from your start/end dates, with an overnight city and free-form notes per day.
- **Weather** — per day, click "check" and it geocodes the overnight city and fetches from [Open-Meteo](https://open-meteo.com) (free, no API key, works straight from the browser). If the date's within 15 days it shows an actual forecast; further out, it averages the same calendar date across the last 5 years so you get a feel for typical conditions.
- **Fuel calculator** — enter your car's efficiency (mpg or L/100km) and a price per litre for each currency in use — one for UK fuel, one for European, say — and it estimates each drive leg's fuel cost in that leg's own currency, applied with one click.
- **Budget** — fuel, ferry/tunnel, tolls, accommodation, food, activities, misc — each with its own currency selector — rolled up into a grand total converted to your home currency, plus a raw per-currency spend breakdown and a per-person split.
- **Packing list** — a shared, tickable list per trip, seeded with a few things that matter for a winter drive into mainland Europe (headlamp beam converters, warning triangle + hi-vis, breakdown cover, etc.) — edit or remove any of it.

## Notes

- No backend, no npm install — it's a single static HTML file.
- If `FIREBASE_CONFIG` is left as `null`, the app works standalone with per-browser storage — good for trying it out solo before setting up sync.
- The weather feature makes requests directly to Open-Meteo's public API from the visitor's browser — no keys or accounts needed, and nothing routes through a server of ours.
