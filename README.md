# 💖 BudChat — Setup Guide

A private 2-person PWA chat (with 1 backup account = 3 max). Made with love by Kaushal Sharma.

## Files in this folder
- `BudChat.html` — the entire app (open this)
- `manifest.json` — PWA manifest
- `sw.js` — service worker (offline shell + cache)
- `firestore.rules` — paste these into Firebase Console
- `README.md` — this file

## 1. Host on GitHub Pages (free)
1. Create a new public repo, e.g. `budchat`.
2. Upload all 4 files (`BudChat.html`, `manifest.json`, `sw.js`, `firestore.rules`) to the root.
3. Repo → **Settings → Pages → Source: Deploy from branch → main / root**.
4. Wait ~1 min. Open `https://<your-username>.github.io/budchat/BudChat.html`.
5. On phone, open the URL → "Add to Home Screen" → installs as a real app.

> **Important:** GitHub Pages serves from a subpath like `/budchat/`. The manifest already uses relative `./BudChat.html` so it works as-is. If you host at the root domain, no changes needed either.

## 2. Firebase Console setup (Spark / Free plan)
Project: `chat-e45d7` (already configured in `BudChat.html`).

### A. Enable Authentication
- Console → **Authentication → Sign-in method**
  - Enable **Google**
  - Enable **Phone** (Spark gives ~10 free SMS/day — enough for 3 people)
- **Authentication → Settings → Authorized domains** → add:
  - `<your-username>.github.io`

### B. Create Firestore Database
- Console → **Firestore Database → Create database** → start in **production mode**.
- Open **Rules** tab → paste the contents of `firestore.rules` → **Publish**.

### C. Create the meta document
You need one initial doc so the rules work. In Firestore:
- Collection: `meta`
- Document ID: `accounts`
- Field: `uids` (type: array, leave empty `[]`)
- Save.

(After this, the first 3 users that sign in will auto-add themselves and the 4th will be blocked.)

## 3. Push notifications (true closed-app, free, no server)

BudChat uses **OneSignal** (free tier, no Firebase Blaze needed):

1. Sign up at https://onesignal.com (free).
2. Create a new app → **Web** → **Typical Site**.
3. Site URL: `https://<your-username>.github.io/budchat/` (with trailing slash).
4. Default icon URL: any HTTPS image.
5. OneSignal will give you a **download** of `OneSignalSDKWorker.js` — upload it to the same folder as `BudChat.html` in your repo.
6. Copy the **App ID** OneSignal shows you.
7. Open `BudChat.html`, find this line near the top of the script:
   ```js
   const ONESIGNAL_APP_ID = "";
   ```
   Paste your App ID inside the quotes. Commit & push.

Now when either user opens BudChat, they'll be prompted to allow notifications. Even when the PWA is closed, OneSignal will deliver notifications.

> **Note on cross-user delivery:** OneSignal's free plan supports notifications to "External User IDs" (each Firebase uid). To trigger them when a message is sent, you have two options:
> - **Easiest (manual / journey):** use OneSignal's **Journeys / In-App Triggers** to fire on a custom event, OR
> - **Best:** add a tiny free Cloudflare Worker (or Vercel function) that watches Firestore `/notifications` and calls OneSignal's REST API. The notification record is already being written by `sendPush()`. Ask Kaushal/Lovable for the worker snippet when you're ready.

If `ONESIGNAL_APP_ID` is left blank, the app still works perfectly — you just won't get push when fully closed.

## 4. WebRTC Calling
Already configured. Uses Firestore for signaling and Google's free public STUN servers. Works on most networks; if a call ever fails on a strict cellular NAT, you can later add a free TURN server (e.g. Metered.ca free tier) — edit the `ICE` constant in `BudChat.html`.

## 5. Account & device limits (already enforced)
- **Max 3 accounts ever** — enforced atomically via Firestore transaction on `/meta/accounts`.
- **Max 3 devices per user** — enforced when registering a new device. Remove devices from Settings.

## 6. Personal touches included 💖
- Default **Love** theme with floating hearts in the background
- Heartbeat animation on the logo and call avatars
- Custom **Aurora** theme made just for BudChat
- "Made with 💖 by Kaushal Sharma" in the About section and on the chat list
- Messages animate in with a soft pop
- Replying, reactions (❤️😂🔥😮😢), delete-for-me, delete-for-everyone
- Voice notes with inline play & progress
- Image sharing (auto-compressed for Spark plan — no Storage required, sent as base64 in Firestore, ~700KB cap)
- Full offline reading via IndexedDB + Firestore persistence
- Outbox queue: send while offline, auto-syncs when back online

Enjoy 💖
