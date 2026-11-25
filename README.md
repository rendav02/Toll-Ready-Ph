# TollReady PH

Philippine Expressway Route Planner & Toll Fee Calculator 
Next.js (App Router) · React 19 · TypeScript · Tailwind CSS v4 · Lucide · Leaflet

## Run

```bash
npm install
npm run dev          # http://localhost:3000
npm run build && npm start
npm run lint         # type-check
npm run test:tolls   # checks the toll engine against known-correct fares (fails if a total changes)
```

Needs an internet connection (map tiles, routing and place search come from free public servers).

## How a trip is calculated

1. **Place search** — typing in a field calls `/api/places`, which queries **Photon** (OpenStreetMap data, Philippines only).
2. **Routes** — `/api/route` asks the **Valhalla** routing server (FOSSGIS, OpenStreetMap data) for the fastest route + alternatives, a toll-avoiding route and, if needed, a middle-ground route.
3. **Toll engine** (`lib/tolls/engine.ts`) — reads each route's toll sections, works out which expressway each one is (by road name, then geometry), snaps entry/exit to the nearest toll plaza, and prices it from the **TollReady rate database** (`data/tolls/expressways.ts`).
4. **Labels** (`lib/routing/labels.ts`) — cheapest / fastest / balanced are chosen for the selected vehicle class. Balanced is the recommendation.
5. **Open in Google Maps** — builds a standard Google Maps directions link (no key needed).

## Coverage (rates compiled Nov 2026)

| Road | Network | Rates as of |
| --- | --- | --- |
| Skyway Stage 3 | Autosweep | Feb 2026 |
| Skyway Stage 1 & 2 (elevated) | Autosweep | Nov 2024 (TRB) |
| SLEX incl. Skyway at-grade | Autosweep | Jul 2026 |
| STAR Tollway | Autosweep | Jul 2026 |
| NAIAX | Autosweep | Feb 2026 |
| MCX | Autosweep | Sep 2025 |
| CAVITEX (R-1, R-1 Ext.) | Easytrip | Oct 2026 |
| CAVITEX–C5 Link | Easytrip | May 2026 |
| CALAX | Easytrip | Aug 2026 |
| NLEX (open + closed system) | Easytrip | Jan 2026 |
| NLEX Connector | Easytrip | Oct 2024 |
| NLEX Harbor Link | Easytrip | Feb 2026 (unconfirmed) |
| SCTEX | Easytrip | Sep 2025 |
| TPLEX | Autosweep | Sep 2025 (TRB) |

Not yet priced: SFEX (Subic Freeport Expressway). Routes using an unpriced road show “not priced” and a “+” on the total. Every road lists its sources in the data file and in the app's Toll Breakdown → Rate sources.

**Updating rates:** edit the tables in `data/tolls/expressways.ts`, change `ratesAsOf`, add the source, then run `npm run test:tolls`.

## Structure

```
app/
  api/route/            POST trip points → priced route options
  api/places/           GET place search, /reverse for “use my location”
  layout.tsx, page.tsx, globals.css
components/
  layout/               AppShell, Sidebar, MobileNav, BottomSheet, Logo
  map/                  MapView (search, controls, framing), LeafletMap, MapControls, MapSearch, types
  panels/               RoutePlanner, RouteComparison, TollBreakdown, Rfid, Settings
  route/                LocationInput (live search), RouteCard
  ui/primitives.tsx
data/
  tolls/expressways.ts  Toll rate database + plaza coordinates (OpenStreetMap)
  tolls/types.ts
  rfid.ts               RFID descriptions
hooks/useTollReady.tsx  App state + calculate flow
lib/
  tolls/engine.ts       Route → toll charges
  routing/valhalla.ts   Routing client (server)
  routing/photon.ts     Place search client (server)
  routing/client.ts     Browser helpers (API calls, geolocation, Google Maps link)
  routing/labels.ts     Cheapest/fastest/balanced + totals
  rfid.ts, geo.ts, format.ts, constants.ts, types.ts
scripts/test-engine.ts  Regression check with recorded routes
```

## Free services and long-term use

| Job | Default (no key) | Backup (free key, no card) |
| --- | --- | --- |
| Routing | Valhalla — FOSSGIS public server | OpenRouteService (non-profit HeiGIT) |
| Place search | Photon — Komoot public server | OpenRouteService geocoder |
| Map tiles | CARTO basemaps (OpenStreetMap data) | OpenStreetMap standard tiles (Layers menu) |

To turn on the backup: sign up at https://openrouteservice.org/dev/#/signup, create a token, copy `.env.example` to `.env.local` and set `ORS_API_KEY=...`, then restart `npm run dev`. If Valhalla or Photon fail, the app switches automatically. Set `ROUTING_PROVIDER=ors` to use OpenRouteService first.

All of these run on free, open-source software and OpenStreetMap data, so you can also self-host them later (`VALHALLA_URL`, `PHOTON_URL`) without changing the app.

## Limits of the free setup

- Travel times are typical driving times, **no live traffic**.
- Public servers are for light use (demos, testing). For heavier use, self-host Valhalla/Photon and set `VALHALLA_URL` / `PHOTON_URL`.
- Toll amounts are **estimates** from published matrices. Always confirm with the TRB or operator.

Credits: map data © OpenStreetMap contributors · tiles © CARTO · routing by Valhalla (FOSSGIS) · search by Photon (Komoot).
