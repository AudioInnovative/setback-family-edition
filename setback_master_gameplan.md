# Setback – Family Edition: Master Gameplan

## 1. Project Vision
Build a feature-complete, self-contained Setback card game playable both offline (with bots) and online (with Firebase multiplayer), all in a single `index.html` file. Prioritize smooth UI/UX, accurate rules, and robust multiplayer support.

---

## 2. Core Rules & Scoring (Reference)
- **Goal:** First team to 21 points, points earned only via captured cards at end of round.
- **Points per Round:** High, Low, Jack, Off-Jack, Joker, Game (max 6 per round).
- **Game Point:** Highest card-value total (10=10, A=4, K=3, Q=2, J=1, others=0).
- **Bidding:** Bidding team must meet/exceed bid or lose that amount; non-bidding team keeps what they earn. 6-and-Out is instant win/loss if all 6 points are made/missed.
- **Negative scores allowed.**

---

## 3. Technical Architecture
- **Single-file HTML app** (no build step, all CDN dependencies)
- **Pure rule engine** (TypeScript-style, pure reducers for state)
- **Bot AI** (basic heuristics, parity with player moves)
- **Firebase Firestore** for real-time sync
- **Offline/PWA** fallback with local bots
- **Vitest (in-browser)** for core logic/unit tests
- **Cypress for E2E**

---

## 4. Implementation Phases & Milestones

### Phase 1: Core Game Logic & UI
- [x] Deck utilities: encoding, shuffling, trump ranking
- [x] Rule engine: deal, bid, discard, play, score, win-check (pure functions)
- [x] Table layout: four seats, widow, trick pile, responsive grid
- [x] Drag-and-drop/tap play
- [x] Bot engine (basic AI, offline fallback)
- [x] Scoring panel (slide-in, SFX)
- [ ] Deal animation (Framer/CSS keyframes)

### Phase 2: Multiplayer & Lobby
- [ ] Lobby UI: anonymous auth, room create/join, seat assignment
- [ ] Firestore sync: onSnapshot for UI hydration, playMove for diffs

### Phase 3: Offline & PWA
- [ ] Service Worker for offline cache
- [ ] Local bot fallback
- [ ] Ensure all game logic works offline/online

### Phase 4: Testing & QA
- [x] Unit: bid, play, trick, scoring, win
- [ ] Multiplayer: room join, sync, play
- [ ] Offline: bot fallback, state persistence
- [ ] UI: drag/tap, animation, sound
- [ ] Accessibility: keyboard, ARIA, color contrast
- [ ] Lighthouse: PWA, performance

### Phase 5: Polish & Deploy
- [ ] UI/UX polish: animations, responsive, SFX
- [ ] Final deployment (Firebase Hosting or Vercel)
- [ ] Documentation & outstanding questions

---

## 5. Outstanding Questions & Decisions
- Card art: custom SVG or minimalist generated?
- Music: original or CC0?
- Analytics: integrate Firebase Analytics?

---

## 6. Immediate Next Steps (Recommended)
1. **Implement Lobby UI**: Auth, room management, seat assignment.
2. **Integrate Firestore sync**: Real-time state updates, secure playMove writes.
3. **Add deal animation**: Visual polish for card dealing.
4. **Begin Service Worker for offline/PWA.**
5. **Expand Vitest/Cypress coverage.**

---

## 7. Reference: Data Models & Engine API

```ts
// Key types (see setback_rule_engine_spec.md for full details)
enum Suit { C, D, H, S }
enum Phase { Deal, Bidding, Discard, Draw, Play, Score, Complete }
interface Player { uid: string; seat: 0|1|2|3; bid?: number|'6-and-out'; hand: CardCode[]; captured: CardCode[]; }
interface GameState { phase: Phase; dealer: 0|1|2|3; turn: 0|1|2|3; trump?: Suit; trick: CardCode[]; players: Player[]; widow: CardCode[]; bids: (number|'6-and-out'|null)[]; highBidSeat?: number; scores: [number, number]; round: number; }
```

---

## 8. File/Module Breakdown
- `index.html`: All markup, style, and scripts (using AI-GEN markers for codegen)
- `deck-utils`: Card/deck helpers
- `engine`: Pure game logic
- `ai`: Bot heuristics
- `renderLobby`, `renderTable`: UI renderers
- `attachDrag`: DnD/tap handlers
- `botEngine`: Bot move logic
- `firebase sync`: subscribeToRoom, playMove
- `service worker`: PWA/offline

---

## 9. Testing & QA Checklist
- [x] Unit: bid, play, trick, scoring, win
- [ ] Multiplayer: room join, sync, play
- [ ] Offline: bot fallback, state persistence
- [ ] UI: drag/tap, animation, sound
- [ ] Accessibility: keyboard, ARIA, color contrast
- [ ] Lighthouse: PWA, performance

---

## 10. Deployment
- Firebase Hosting: `firebase init hosting`, copy assets, `firebase deploy`
- Vercel: `vercel --prod`

---

## 11. Review & Iterate
- Regularly revisit this plan after each milestone or major feature.
- Update outstanding questions and polish as needed.
