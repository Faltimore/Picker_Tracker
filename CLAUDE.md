# CLAUDE.md — Picker Tracker

## Project Overview

PWA for Nashy's Farm to track daily picker harvest weights. Multiple contractors log in and record bucket weights for their assigned pickers throughout the day. An admin (Anna) monitors all sessions live.

**Live URL:** https://nashys-pickers.web.app  
**Firebase project:** `nashys-pickers`  
**Git repo:** https://github.com/Faltimore/Picker_Tracker

---

## Architecture

**Single-file app** — everything lives in `index.html`. There is no build system, no npm, no node_modules. React and all dependencies are loaded from CDN. Edits to `index.html` are deployed directly via `firebase deploy --only hosting`.

Do not introduce a build system, package.json, or separate component files unless explicitly requested.

## Key Files

| File | Purpose |
|---|---|
| `index.html` | Entire application — React components, Firebase init, all logic |
| `sw.js` | Service worker — PWA offline caching |
| `manifest.json` | PWA manifest (name, icons, theme) |
| `firebase.json` | Firebase Hosting config — serves from repo root |
| `.firebaserc` | Firebase project alias (`nashys-pickers`) |
| `seed_data.html` | Dev utility to reset Firestore to a clean state for testing |
| `404.html` | Firebase Hosting 404 fallback |

## Firebase / Firestore

**Collections:**

- `appData/pickers` — `{ list: [{ id, name, idCode }] }` — master picker registry
- `sessions/{sessionId}` — active day sessions, deleted when contractor ends day
- `history/{sessionId}` — completed sessions, permanent record

**Session document shape:**
```js
{
  date: "YYYY-MM-DD",
  activePickers: ["pickerId", ...],
  entries: { pickerId: [{ weight: 12.5, time: "09:30", bin: "BIN001" }] },
  pickerNames: { pickerId: "Name (ID)" },
  contractorEmail: "contractor@example.com",
  contractorName: "contractor",
  currentBin: "BIN002",
  bins: [{ number: "BIN001", totalWeight: 142.5, bucketCount: 11, closedAt: "11:20" }]
}
```

Stale sessions from previous days are automatically moved to history when any user loads the app.

## Auth & Roles

- All users: Firebase email/password auth
- **Admin emails** (hardcoded in `index.html`): `["anna@nashysfarm.com"]`
- Admins land on the Live dashboard; contractors land on Setup
- To add a new admin, update the `ADMIN_EMAILS` array in `index.html`

## App Screens

| Screen | Who | Description |
|---|---|---|
| `setup` | Contractor | Select pickers, enter starting bin, start the day |
| `tracking` | Contractor | Add bucket weights, manage bins, end day |
| `history` | All | Search/filter past sessions, export to Excel |
| `manage` | All | Add/remove pickers from the master list |
| `live` | Admin only | Real-time view of all active contractor sessions |

## Multi-Contractor Concurrency

- Each contractor gets a unique session ID stored in `localStorage`
- Pickers assigned to another contractor's active session today are shown as unavailable (greyed out) on the Setup screen
- `takenPickers` is filtered against the current pickers list — deleted pickers never show as "taken"

## Excel Export

Exports via SheetJS with four sheets:
1. **Daily** — per-day table with each picker's bucket-by-bucket weights, gross totals, and bin details
2. **Weekly** — aggregated picker summary grouped by week
3. **Monthly** — aggregated picker summary grouped by month
4. **Picker Summary** — lifetime totals per picker with avg gross/day and avg gross/bucket

**Gross only** — no wastage or nett figures are shown anywhere in the app or exports. The contractor calculates wastage externally.

## Deployment

```bash
firebase deploy --only hosting
```

Always commit to git before deploying. No build step needed.

## Style Conventions

- Dark theme — background `#1a1e2a`, surface `#232838`
- Accent colour: `#e8a838` (amber)
- Font: JetBrains Mono (monospace throughout)
- All colours defined in the `C` constant object at the top of the script
- Inline styles only (no CSS classes, no CSS-in-JS library)
- Button base styles in `baseBtn`, input base styles in `inputBase`

## Common Tasks

**Add a new admin:** Find `ADMIN_EMAILS` array in `index.html`, add the email.

**Change the app title/theme colour:** Edit `manifest.json` and the `<title>` / `<meta name="theme-color">` in `index.html`.

**Reset test data:** Open `seed_data.html` in a browser while logged in — it clears and re-seeds Firestore.

**Test locally:** Open `index.html` directly in a browser. Firebase Hosting emulator is not required for basic testing, but Firestore calls require internet access (no local emulator configured).
