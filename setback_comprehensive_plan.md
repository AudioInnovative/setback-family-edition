# Setback – Comprehensive Implementation Plan
*(Generated 2025-04-23)*

---

## 1  Vision & Scope
Build a feature-complete **Setback (Pitch) card game** playable:
1. **Offline** – local single-player vs. bots (no internet required).
2. **Online** – real-time multiplayer via Firebase (anonymous auth + Firestore).
3. **PWA** – installable, offline-first, with service-worker cache.

The entire client runs inside a **single `index.html` file** using CDN scripts (no build step). All game rules are enforced by a **pure, unit-tested rule engine** that runs identically on client and server/cloud functions.

---

## 2  Gameplay Rules Summary  *(condensed from `Setback_Scoring_Rules_Updated.md`)*
| Point | Description |
|-------|-------------|
| **High** | Highest trump captured |
| **Low** | Lowest trump captured |
| **Jack** | Right Jack (Jack of trump suit) |
| **Off-Jack** | Left Jack (same-colour, opposite-suit Jack) |
| **Joker** | Joker captured |
| **Game** | Highest card-value total (10 = 10, A = 4, K = 3, Q = 2, J = 1) |

• First team to **21 points** wins.  
• **Bidding** every hand (1-6 or *6-and-Out*). Bid team must meet/exceed bid or lose that many points.  
• **Negative scores** allowed.

---

## 3  Technical Architecture  *(merged from all MD specs)*
### 3.1  Front-End Stack
| Purpose | Library | Delivery |
|---------|---------|----------|
| UI Utility | **Tailwind Play CDN** | `<script src="https://cdn.tailwindcss.com"></script>` |
| Drag & Drop | **SortableJS** v1.15 | `https://cdn.jsdelivr.net/npm/sortablejs@1.15.0/Sortable.min.js` |
| Animations | **Web Animations API** | *Built-in: `element.animate()` (no external dependency)* |
| Audio | **Howler.js** v2.2 | `https://cdn.jsdelivr.net/npm/howler@2.2.3/dist/howler.min.js` |
| Realtime DB | **Firebase v10 Compat** | app, auth, firestore compat bundles |

**Note:**
- Framer Motion was removed due to persistent CDN/network loading issues. All animations now use the built-in Web Animations API for reliability and offline support.

---

### 3.2  Pure Rule Engine (`/engine.ts`)
Defined in `setback_rule_engine_spec.md`:
- Immutable `GameState`, `Player`, `Suit`, `Phase` types.
- Pure reducers: `deal()`, `validateBid()`, `applyBid()`, `validateDiscard()`, `applyDiscard()`, `validatePlay()`, `applyPlay()`, `scoreRound()`, `checkWin()`.
- Helper: `trumpRank(code: CardCode, trump: Suit) -> number` for trump ordering.
- No DOM / side-effects → fully unit-testable with **Vitest**.

### 3.3  Folder / File Map
```
Setback4th/
│  index.html              # Single-file runnable game
│  deck-utils.ts           # generateDeck, shuffle, dealHands
│  engine.ts               # pure rule engine
│  bot-engine.ts           # heuristic AI, wraps engine
│  service-worker.js       # PWA cache, offline fall-back
│  tests/engine.spec.ts    # Vitest unit tests
│  cypress/…               # End-to-end specs
│  README.md               # project usage & dev guide
│  setback_*.md            # planning docs (incl. THIS file)
```

### 3.4  Firestore Schema
Define the Firestore document under `games/{roomId}`:
```jsonc
// games/{roomId}
{
  "seats": ["uid0", "uid1", null, "BOT"],
  "phase": "deal" | "bidding" | "play" | "score" | "complete",
  "state": { /* serialized GameState */ },
  "settings": { "botsAllowed": true, "created": 1713888000 }
}
```
Clients subscribe via `onSnapshot` and write diffs using a `playMove` callable function or direct writes with security rules enforcing valid moves.

---

## 4  Feature Roadmap & Milestones
| Phase | Goal | Key Deliverables |
|-------|------|------------------|
| **1 – Core Offline** | Play full round vs. 3 bots locally | Deck utils, engine.ts, bot-engine.ts, basic table UI, drag/tap play, score panel |
| **2 – Deal Animation** | Polish feel | Web Animations API fan-out, SFX shuffle, card-flip |
| **3 – Multiplayer Lobby** | Anonymous auth & seat selection | Lobby UI, Firestore `games/{roomId}` schema, onSnapshot hydration, playMove diff writes |
| **4 – PWA / Offline** | Installable & works offline | service-worker.js, local IndexedDB save, offline bot fallback, reconnect sync |
| **5 – Testing & QA** | Confidence | Vitest ≥ 95% coverage, Cypress happy-path game, Lighthouse PWA ≥ 90 |
| **6 – Polish & Deploy** | Production ready | Animations, accessibility pass, Vercel & Firebase Hosting CI/CD |

---

## 5  Detailed Task Breakdown
### 5.1  Phase 1 – Core Offline (Estimated ~2 days)
- [ ] **Deck Utils** `deck-utils.ts`  
  • `generateDeck()`, `shuffle()`, `dealHands()`  
- [ ] **Rule Engine** `engine.ts`  
  • Implement spec functions  
  • Unit tests in `tests/engine.spec.ts`  
- [ ] **Bot Engine** `bot-engine.ts`  
  • Bids based on hand strength  
  • Plays follow-suit, trump logic  
- [ ] **Table UI**  
  • Four seat grid, widow pile, trick area  
  • Drag (SortableJS) / tap handler -> `playMove()`  
- [ ] **Scoring Panel**  
  • Slide-in tally, restart button  

### 5.2  Phase 2 – Deal Animation (1 day)
- Web Animations API fan-out from deck center.  
- Shuffle SFX via Howler.  

### 5.3  Phase 3 – Multiplayer Lobby (3 days)
- Anonymous auth (`firebase.auth().signInAnonymously()`).  
- Lobby page: create/join room, seat buttons.  
- Firestore sync: `onSnapshot` → render, `playMove` callable diff.  
- Conflict resolution → engine pure diff apply.

### 5.4  Phase 4 – PWA / Offline (1-2 days)
- `service-worker.js` cache static, CDN, and last state.  
- Detect `navigator.onLine`, spawn local bots when offline.  

### 5.5  Phase 5 – Testing & QA (ongoing)
- Vitest for every pure function.  
- Cypress: full game vs. bot, multiplayer room sync.  
- Accessibility audit (Lighthouse).  

### 5.6  Phase 6 – Polish & Deploy (1-2 days)
- Add responsive tweaks, animations, sound sliders.  
- Prepare `firebase.json`, `vercel.json`.  
- Deploy to Firebase Hosting & Vercel preview.  

---

## 6  Timeline (Aggressive Estimate)
| Week | Focus |
|------|-------|
| **1** | Phase 1 completion, playable offline round |
| **2** | Deal animations + Phase 3 lobby groundwork |
| **3** | Multiplayer sync stable, start PWA work |
| **4** | PWA complete, heavy testing, polish |
| **5** | Buffer, accessibility, final deploy |

*(Each “week” ≈ 20 focused hrs; adjust as needed.)*

---

## 7  Risks & Mitigations
| Risk | Impact | Mitigation |
|------|--------|-----------|
| Firestore latency & quotas | Laggy multiplayer | Throttle diff rate, compress state, store minimal snapshot + ops |
| Offline complexity | Desync on reconnect | Hash state, CRC diff, merge on reconnect, authoritative seat |
| Animation perf on mobile | Jank | Web Animations API `will-change`, reduce blur/shadow on low-end |
| Rule engine bugs | Incorrect scoring | 100% unit test coverage, golden test vectors |

---

## 8  Acceptance Criteria (Definition of Done)
1. **Single-player offline** round completes with correct scoring.  
2. **Multiplayer** two browsers sync perfectly, seat assignment locks.  
3. **Installable PWA**, offline launch shows bots.  
4. **Unit tests** pass in-browser Vitest.  
5. **Cypress** happy-path (bot & multi) green on CI.  
6. **Lighthouse** PWA score ≥ 90, no accessibility blockers.  
7. Deployed to **https://setback.family** (or similar) with CI-CD.

---

## 9  Next Immediate Actions
1. Scaffold `deck-utils.ts`, `engine.ts`, `tests/engine.spec.ts`.  
2. Integrate these into existing `index.html` with dummy buttons.  
3. Build minimal lobby and seat render with current local bot logic.

---

### 4.2  UI/UX Fixes (2025-04-24)
- **Bot hands are now hidden until the deal animation begins.**
- **Dealing animation** is implemented using the Web Animations API, one card at a time.
- **No external animation library is required.**
- **Game starts with an empty table** (no bot or player cards visible) until the user clicks "Deal."
- **All CDN/network dependency issues for animation are resolved.**

> **Let’s build Setback one phase at a time—begin with deck utils + rule engine!**
