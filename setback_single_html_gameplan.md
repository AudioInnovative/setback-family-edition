# Setback – Family Edition  
### **Single–File Web Build Game‑Plan**  
*(Generated: 2025-04-23)*

---

## Table of Contents
1. [Project Overview](#project-overview)  
2. [Tech Stack & CDN Dependencies](#tech-stack--cdn-dependencies)  
3. [`index.html` Skeleton](#indexhtml-skeleton)  
4. [Firestore Schema](#firestore-schema)  
5. [Rule Engine & Bot AI](#rule-engine--bot-ai)  
6. [Implementation Steps](#implementation-steps)  
7. [Testing & QA](#testing--qa)  
8. [Deployment](#deployment)  
9. [Outstanding Questions](#outstanding-questions)

---

## Project Overview
Build **Setback – Family Edition** as a *self‑contained* `index.html` that:

- Plays offline (local bots) **and** online (Firebase multiplayer).  
- Provides drag‑and‑drop (desktop) & tap‑to‑play (mobile).  
- Uses SVG card art, smooth Framer‑Motion animations, subtle ambient music & SFX.  
- Requires **no build step** – everything loads from CDN.

---

## Tech Stack & CDN Dependencies
| Purpose | Library | CDN URL (ESM‑ready) |
|---------|---------|---------------------|
| UI animations | `framer-motion` web UMD | `https://cdn.jsdelivr.net/npm/framer-motion@11/dist/framer-motion.umd.js` |
| Drag & Drop | `sortablejs` | `https://cdn.jsdelivr.net/npm/sortablejs@1.15.0/+esm` |
| CSS Utility | Tailwind v3 | `https://cdn.jsdelivr.net/npm/tailwindcss@3/dist/tailwind.min.css` |
| Realtime DB | **Firebase v10** (app, auth, firestore) | `https://www.gstatic.com/firebasejs/10.9.0/firebase-*.js` |
| Audio | Howler.js | `https://cdn.jsdelivr.net/npm/howler@2.2.3/+esm` |

> **Local dev:** only `live‑server` or VS Code Go‑Live is required.

---

## `index.html` Skeleton
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Setback – Family Edition</title>
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <!-- Tailwind -->
  <link rel="stylesheet"
        href="https://cdn.jsdelivr.net/npm/tailwindcss@3/dist/tailwind.min.css"/>
  <style>
    /* Card sizing & flip */
    .card{width:80px;height:120px;perspective:1000px}
    .card>.inner{transition:transform .4s;transform-style:preserve-3d}
    .flipped{transform:rotateY(180deg)}
    .face,.back{position:absolute;backface-visibility:hidden}
    .back{transform:rotateY(180deg)}
  </style>
</head>
<body class="bg-gradient-to-br from-slate-900 to-slate-800 text-gray-100
             flex flex-col items-center min-h-screen select-none">

  <!-- Lobby / Game containers -->
  <div id="lobby" class="my-4"></div>
  <div id="table" class="flex-1 w-full"></div>
  <audio id="bg-loop" src="audio/ambience.mp3" loop></audio>

  <!-- CDN scripts -->
  <script type="module" src="https://www.gstatic.com/firebasejs/10.9.0/firebase-app-compat.js"></script>
  <script type="module" src="https://www.gstatic.com/firebasejs/10.9.0/firebase-firestore-compat.js"></script>
  <script type="module" src="https://www.gstatic.com/firebasejs/10.9.0/firebase-auth-compat.js"></script>
  <script type="module" src="https://cdn.jsdelivr.net/npm/sortablejs@1.15.0/+esm"></script>
  <script type="module" src="https://cdn.jsdelivr.net/npm/framer-motion@11/dist/framer-motion.umd.js"></script>
  <script type="module" src="https://cdn.jsdelivr.net/npm/howler@2.2.3/+esm"></script>

  <!-- === APP CODE === -->
  <script type="module">
  /* ========= CONFIG ========= */
  const firebaseConfig = {{ /* TODO: insert keys */ }};
  firebase.initializeApp(firebaseConfig);
  const db = firebase.firestore();
  const auth = firebase.auth();

  /* ========= UTILITIES ========= */
  // AI-GEN-BEGIN deck-utils
  // generateDeck(), shuffle(), dealHands() …
  // AI-GEN-END

  /* ========= STATE & STORE ========= */
  const state = {{
    view: 'lobby', playerId: null, roomId: null,
    seat: 0, game: null, offline: false
  }};

  /* ========= RENDERERS ========= */
  // AI-GEN-BEGIN renderLobby
  // AI-GEN-END

  // AI-GEN-BEGIN renderTable
  // AI-GEN-END

  /* ========= DRAG / TAP HANDLERS ========= */
  // AI-GEN-BEGIN attachDrag
  // AI-GEN-END

  /* ========= FIREBASE SYNC ========= */
  // subscribeToRoom(), playMove()

  /* ========= BOT ENGINE ========= */
  // AI-GEN-BEGIN botEngine
  // AI-GEN-END

  /* ========= OFFLINE FALLBACK ========= */
  window.addEventListener('offline', () => {{
    state.offline = true;
    spawnLocalBots();
  }});
  /* … */

  /* ========= AUDIO ========= */
  const sfx = new Howl({{src:['audio/shuffle.mp3']}});
  document.getElementById('bg-loop').volume = 0.25;

  /* ========= BOOTSTRAP ========= */
  auth.signInAnonymously().then(() => renderLobby());
  </script>
</body>
</html>
```

### AI‑Marker Conventions
Blocks are wrapped by `// AI-GEN-BEGIN <name>` / `// AI-GEN-END` for a coding‑assistant to target.

---

## Firestore Schema
```jsonc
// games/{roomId}
{
  "seats": ["uidA", "uidB", null, "BOT"],
  "phase": "deal" | "bidding" | "play" | "score",
  "dealer": 0,
  "state": { /* serialised game snapshot */ },
  "settings": { "botsAllowed": true, "created": 1713888000 }
}
```

---

## Rule Engine & Bot AI
| Module | Responsibility |
|--------|----------------|
| `deck-utils` | Card encoding, shuffle, trump ranking |
| `engine.playMove()` | Pure reducer producing next state or error |
| `ai.evaluateHand()` | 70% lowest‑legal / 30% highest‑legal heuristic |

---

## Implementation Steps
1. [x] **Deck & Rule Logic** – write, unit‑test with Vitest-in-browser (or import).  
2. [ ] **Lobby UI** – anon Auth, room create/join, seat buttons.  
3. [x] **Table Layout** – four seats, widow, trick pile; responsive grid.  
4. [ ] **Deal Animation** – Framer keyframes fan from deck center.  
5. [x] **Drag Play** – Sortable list for hand; dropzone executes `playMove`.  
6. [ ] **Firestore Sync** – `onSnapshot` to hydrate UI; `playMove` writes diff.  
7. [x] **Bot Engine** – activate for `"BOT"` seats or offline fallback.  
8. [x] **Scoring Panel** – slide‑in, point tally SFX.  
9. [ ] **Offline PWA** – Service Worker cache, local bots.  
10. [ ] **QA** – manual + headless (Cypress with two tabs).

---

## Testing & QA
- **In‑browser Vitest**: bidding, trick winner, scoring.  
- **Cypress**: full bot game, offline fallback.  
- **Lighthouse**: PWA score ≥ 90.

---

## Deployment
```bash
firebase init hosting
cp index.html cards/ audio/ public/
firebase deploy
```
or  
```bash
vercel --prod
```

---

## Outstanding Questions
1. **Card Art** – custom or generated minimalist SVG set?  
2. **Music Loop** – original track or CC‑0 ambience?  
3. **Analytics** – integrate Firebase Analytics?

---

*End of document.*
