# DönerDate – PWA MVP

A ultra-lean Progressive Web App to **create → share → join** spontaneous kebab meetups (“DönerDates”).  
Focus: *local, low friction, link-first virality.*

## ✨ Core MVP

- Map with nearby venues (Leaflet + OpenStreetMap).
- Create a DönerDate: pick venue on map, set time.
- Share an invitation link (`/e/:id`).
- RSVP: **Going / Maybe / No**.
- Live participant list.
- (Optional) basic history and simple moderation.

## 🧱 Tech Stack

- **React + Vite + TypeScript**
- **Firebase** – Firestore (DB), Hosting, optional Functions/Analytics
- **React Router**, **TanStack Query**
- **Leaflet / react-leaflet** with **OSM tiles**
- **PWA** using `vite-plugin-pwa` (optional at first)

## 🚀 Quick Start

```bash
# Node 18+
npm create vite@latest doenerdate -- --template react-ts
cd doenerdate

npm i firebase react-router-dom @tanstack/react-query leaflet react-leaflet
npm i -D vite-plugin-pwa workbox-window

# dev
npm run dev
# prod build
npm run build
npm run preview

Create .env.local:

VITE_FIREBASE_API_KEY=...
VITE_FIREBASE_AUTH_DOMAIN=...
VITE_FIREBASE_PROJECT_ID=...
VITE_FIREBASE_APP_ID=...

Minimal Firebase bootstrap lives in src/lib/firebase.ts.

🗂️ Project Structure

src/
├─ app/
│  ├─ routes.tsx
│  └─ queryClient.ts
├─ components/
│  └─ Map.tsx
├─ features/
│  └─ events/
│     ├─ CreateEvent.tsx
│     ├─ EventDetail.tsx
│     └─ EventList.tsx
├─ lib/
│  ├─ firebase.ts
│  └─ time.ts
├─ types/
│  └─ models.ts
├─ App.tsx
└─ main.tsx
public/
├─ icons/
└─ manifest.webmanifest

🗃️ Data Model (MVP)

// src/types/models.ts
export type RSVPStatus = 'going' | 'maybe' | 'no';

export interface EventDoc {
  title: string; // "Döner @ {venueName}"
  venue: { name: string; lat: number; lng: number; address?: string };
  startAt: number;             // epoch ms
  hostId: string;              // pseudonym or uid
  visibility: 'public' | 'link';
  status: 'open' | 'closed' | 'cancelled';
  counts?: { going: number; maybe: number };
  createdAt: number;
}

export interface ParticipantDoc {
  id: string;                  // uid or anonymous id
  displayName: string;
  status: RSVPStatus;
  joinedAt: number;
}

Suggested Collections

/events/{eventId}
/events/{eventId}/participants/{participantId}
/users/{uid}                 // optional for pseudonyms
/reports/{id}                // optional minimal moderation

🔐 Firestore Security (development)

For local hacking you can allow open access, but do not ship this:

// firebase.rules — DEV ONLY
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} { allow read, write: if true; }
  }
}

Harden before public tests:
	•	Only host can update/cancel an event.
	•	A participant can write only their own RSVP doc.
	•	Consider rate limits and basic validation (timestamps in the future, status enum).
	•	Optionally add a Cloud Function that denormalizes counts.going/maybe on RSVP writes.

🗺️ Maps & Geocoding
	•	Tiles: https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png
	•	Keep the OSM attribution in the map UI.
	•	Nominatim (if you add geocoding): respect the usage policy (≤ 1 req/s, proper User-Agent/Referer, no heavy bulk use). For the MVP, prefer manual venue name input.

📱 PWA Setup

public/manifest.webmanifest:

{
  "name": "DönerDate",
  "short_name": "DönerDate",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#111111",
  "icons": [
    { "src": "/icons/icon-192.png", "sizes": "192x192", "type": "image/png" },
    { "src": "/icons/icon-512.png", "sizes": "512x512", "type": "image/png" }
  ]
}

Optional service worker via vite-plugin-pwa:

// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { VitePWA } from 'vite-plugin-pwa';

export default defineConfig({
  plugins: [
    react(),
    VitePWA({
      registerType: 'autoUpdate',
      workbox: { navigateFallbackDenylist: [/^\/e\//] } // example
    })
  ]
});

iOS supports Home-Screen web apps and Web Push (iOS 16.4+). Test install & notifications on real devices.

🔔 Notifications (later)

Start without push. If needed:
	•	Web Push via Firebase Cloud Messaging (FCM).
	•	For store presence, wrap with Capacitor (Push, native share, app badges).
Migration to Expo React Native is also viable once mobile-first UX becomes core.

📊 Metrics
	•	North Star: Confirmed DönerDates per active week (CDPW).
	•	Core: D1/D7 retention, Invite→Join conversion, Show-up rate, Time-to-first-date.
	•	Telemetry: event_created, rsvp_going, invite_opened.

🚢 Deployment

Easiest: Firebase Hosting.

npm i -D firebase-tools
firebase login
firebase init hosting
npm run build
firebase deploy

Ensure correct rewrites for SPA routing:

{
  "hosting": {
    "public": "dist",
    "ignore": ["firebase.json", "**/.*", "**/node_modules/**"],
    "rewrites": [{ "source": "**", "destination": "/index.html" }]
  }
}

🧭 Roadmap (example)
	•	Week 0–1: Create → Share → Join flow, live in one district.
	•	Week 2: Pseudonyms, basic reports, simple analytics.
	•	Week 3: Match-Ping (now / nearby), badges v1.
	•	Week 4: Sponsored spots, pilot partners, push notifications.

🛡️ Trust & Safety
	•	Clear community guidelines.
	•	Easy block/report.
	•	Minimal data collection (privacy by default).
	•	Track no-show rates softly; surface reputation later.

📝 License

MIT — see LICENSE.