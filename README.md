# Picker Tracker

A Progressive Web App (PWA) for tracking daily picker harvest weights on a farm. Built for multiple contractors working simultaneously, with a live admin dashboard and Excel export.

**Live app:** https://nashys-pickers.web.app

---

## Features

- **Multi-contractor support** — multiple contractors can run simultaneous sessions; pickers are locked to one session at a time to prevent double-entry
- **Live tracking** — add bucket weights per picker in real time; entries sync instantly via Firestore
- **Bin tracking** — track which bin is currently being filled, close bins and open new ones, see per-bin totals
- **Admin live dashboard** — Anna's admin account sees all active sessions and grand totals across all contractors in real time
- **History & search** — filter past sessions by picker name and/or date range
- **Excel export** — exports Daily, Weekly, Monthly, and Picker Summary sheets with gross kg per picker; individual session export also available
- **Offline support** — Firestore persistence keeps the app functional offline; data syncs when connection returns
- **PWA installable** — can be installed on iOS/Android/desktop as a standalone app

## User Roles

| Role | Access |
|---|---|
| Contractor | Setup, Track, History, Manage Pickers |
| Admin (`anna@nashysfarm.com`) | Live dashboard, History |

## Tech Stack

- **Frontend:** React 18 (via CDN, no build step), Babel standalone, JetBrains Mono font
- **Database:** Firebase Firestore (real-time sync, offline persistence)
- **Auth:** Firebase Authentication (email/password)
- **Hosting:** Firebase Hosting
- **Export:** SheetJS (xlsx)
- **PWA:** Service Worker + Web App Manifest

## Firestore Collections

| Collection | Purpose |
|---|---|
| `appData/pickers` | Master list of all registered pickers |
| `sessions/{id}` | Active day sessions (live, deleted on end-day) |
| `history/{id}` | Completed sessions (permanent record) |

A session document contains: `date`, `activePickers[]`, `entries{}`, `pickerNames{}`, `contractorEmail`, `contractorName`, `currentBin`, `bins[]`.

## Project Structure

```
picker_tracker/
├── index.html          # Entire app (React + Firebase, single file)
├── sw.js               # Service worker for PWA offline support
├── manifest.json       # PWA manifest
├── 404.html            # Firebase Hosting 404 page
├── firebase.json       # Firebase Hosting config
├── .firebaserc         # Firebase project alias (nashys-pickers)
├── seed_data.html      # Dev tool to reset Firestore test data
└── public/             # Firebase Hosting public assets
```

## Deployment

```bash
firebase deploy --only hosting
```

No build step required — the app is a single `index.html` with CDN dependencies.

## Development Notes

- All app logic lives in `index.html` inside a `<script type="text/babel">` block
- Stale sessions from previous days are automatically moved to history on next load
- `takenPickers` only counts picker IDs that exist in the current pickers list, preventing ghost warnings after data resets
- Gross weights only are displayed and exported — wastage calculation is handled externally by the contractor
