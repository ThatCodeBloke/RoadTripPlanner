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
- **Roadbook** — the route, leg by leg, with mode, distance, duration, notes and cost. Places are looked up automatically in the background as you type them (and cached on the trip), and a new leg with no distance gets one filled in via [OSRM](https://project-osrm.org) (free, keyless) for drive legs, or a straight-line distance for ferry/tunnel/flight legs. The ↻ button re-does that on demand.
  - **Drive legs price their own fuel.** Cost = distance × your car's efficiency × the pump price for that leg's currency, and it updates live when you change the distance, efficiency or price.
  - **Ferry and Eurotunnel legs get an estimated fare.** There's no free, keyless API for live ferry prices, so this is a typical one-way fare for a car (Dover–Calais, Folkestone–Calais tunnel, Newhaven–Dieppe, the Portsmouth/Poole/Plymouth routes to France and Spain), adjusted for the time of year from the leg's date (Christmas/New Year and summer are dearer, Nov–Mar cheaper). Routes it doesn't recognise get a rough guess from the distance. Treat it as a placeholder and use the leg's "Check current prices" link for the real fare.
  - Automatic figures show an **auto** tag (hover for how it was worked out). Type your own number into a leg and it switches to **manual · reset**; click that to go back to automatic. Trips saved before this change keep any number you'd typed by hand.
  - Each leg has its own currency: ferries and the tunnel default to your home currency (UK operators sell in GBP), other legs follow the country you end up in. Overridable any time.
- **Map** — every stop on the route plotted on an OpenStreetMap map (via [Leaflet](https://leafletjs.com), free, no key needed). Drive legs follow the real road: the route shape is fetched once per leg from [OSRM](https://project-osrm.org) and stored (compactly) on the trip, so it's instant afterwards and shared with whoever you sync with. Ferries, the tunnel and trains are drawn as dotted lines between the two places, with a waypoint so the long Biscay ferries to Spain don't cut across Brittany. Arrows show direction; outbound is the thick line and the return is a thinner line on top, so a road driven both ways shows both. A dashed line means a road hasn't loaded yet (click "Update map" to retry). Ferry routes aren't real shipping lanes — there's no free source for those — so treat the dotted lines as "this leg goes by sea", not the exact track.
- **Day by day** — generated from your start/end dates, with an overnight city and free-form notes per day. Above the day tabs there's a rough guide to **which countries you'll be in on each day**, worked out by spreading the legs' travel time across your dates. Fill in "Overnight in" on any day to pin the route to it and the other days adjust around it. It uses each leg's start and end country (with the border assumed halfway), so a country you only drive through on a long leg won't be listed.
- **Itinerary** — a timed plan for each day, built from the legs and the day-by-day split: leave time, drives (with a break every couple of hours), ferry/tunnel check-in, sailing, arrival and overnight. Set a default start time and break/check-in lengths at the top, or override the start for any single day. On a ferry or tunnel leg you can enter the sailing/train you've actually booked ("Sailing" field); leave it blank and the itinerary uses the earliest one after check-in, and warns you if a booked time is too tight to make check-in. Clocks are adjusted when you cross between the UK and mainland Europe. Add your own items to any day (dinner, a booking, a sight) and they slot in by time. "Copy as text" gives you a plain-text version to paste into a message, and "Print / save as PDF" prints just the itinerary. Times are estimates: real sailings run to the operator's timetable.
- **Weather** — per day, click "check" and it geocodes the overnight city and fetches from [Open-Meteo](https://open-meteo.com) (free, no API key, works straight from the browser). If the date's within 15 days it shows an actual forecast; further out, it averages the same calendar date across the last 5 years so you get a feel for typical conditions.
- **Fuel calculator** — enter your car's efficiency (mpg or L/100km) and a price per litre for each currency in use — one for UK fuel, one for European, say. Every drive leg's cost is then filled in automatically in that leg's own currency. "Recalculate all drive legs" hands any hand-edited legs back to the automatic figure.
- **Budget** — fuel, ferry/tunnel, tolls, accommodation, food, activities, misc — each with its own currency selector — rolled up together with the leg costs into a grand total converted to your home currency, plus a raw per-currency spend breakdown and a per-person split. Fuel and ferry/tunnel for the legs are counted automatically under "Route costs", so the Fuel and Ferry rows here are labelled "extra" and are only for spend that isn't tied to a leg.
- **Packing list** — a shared, tickable list per trip, seeded with a few things that matter for a winter drive into mainland Europe (headlamp beam converters, warning triangle + hi-vis, breakdown cover, etc.) — edit or remove any of it.

## Notes

- No backend, no npm install — it's a single static HTML file.
- If `FIREBASE_CONFIG` is left as `null`, the app works standalone with per-browser storage — good for trying it out solo before setting up sync.
- The weather feature makes requests directly to Open-Meteo's public API from the visitor's browser — no keys or accounts needed, and nothing routes through a server of ours.
