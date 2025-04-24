
# Setback – Family Edition  
### Gameplay & Rule Engine Implementation Spec  
*(Generated 2025-04-24)*

This spec targets a coding‑assistant AI inside **Cursor/Windsurf**.  
It defines the pure rule engine, data models, and validation contracts necessary to power the client‑side (and optional Cloud Function) logic.

---

## 1. Data Models

```ts
enum Suit { C='C', D='D', H='H', S='S' }
enum Phase { Deal='deal', Bidding='bidding', Discard='discard', Draw='draw', Play='play', Score='score', Complete='complete' }

type CardCode = string; // 'AH', '7S', 'X' (joker)

interface Player {
  uid: string;            // Firebase uid or 'BOT'
  seat: 0 | 1 | 2 | 3;
  bid?: number | '6-and-out';
  hand: CardCode[];
  captured: CardCode[];
}

interface GameState {
  phase: Phase;
  dealer: 0 | 1 | 2 | 3;
  turn:   0 | 1 | 2 | 3;
  trump?: Suit;
  trick: CardCode[];          // index = seat
  players: Player[];
  widow: CardCode[];
  bids: (number|'6-and-out'|null)[];
  highBidSeat?: number;
  scores: [number, number];   // team indices
  round: number;
}
```

---

## 2. Pure Functions (`engine.ts`)

| Function | Signature | Notes |
|----------|-----------|-------|
| `deal()` | `(seed?:number) -> GameState` | Shuffles, sets phase→Bidding |
| `validateBid()` | `(state, seat, bid) -> boolean` | Bid 1–6 or '6-and-out', exceeds prev high |
| `applyBid()` | `(state, seat, bid) -> GameState` | Mutates bids array, phase end logic |
| `validateDiscard()` | `(state, seat, discards[])` | Non‑trump only, ≤ 6 trump kept |
| `applyDiscard()` | `(state, seat, discards[])` | Removes cards, phase advance |
| `validatePlay()` | `(state, seat, card) -> boolean` | Suit‑follow and ownership |
| `applyPlay()` | `(state, seat, card) -> GameState` | Adds to trick, resolves when 4 |
| `scoreRound()` | `(state) -> GameState` | Calculates 6 point categories + bid penalties |
| `checkWin()` | `(state) -> 'team0'|'team1'|null` | 21 pts or 6‑and‑Out outcome |

All functions are pure and unit‑testable.

---

## 3. Trump Ranking Helper

```ts
function trumpRank(code: CardCode, trump: Suit): number {
  // 0 = highest, larger = weaker
  // order: A,K,Q,RJ,OJ,Joker,10,9,8,7,6,5,4,3,2
}
```

`Right Jack` = Jack of trump suit; `Off Jack` = Jack same colour.

---

## 4. Bot Heuristic (single difficulty)

1. **Bid**: compute `strength = trumpCount + highRanks*0.5`;  
   - pass if <3, bid 3 if 3‑4, bid 4/5 if >4.5, 5 % chance 6‑and‑Out.  
2. **Discard**: keep top 6 trump (or all if ≤6).  
3. **Play**: follow suit; if void, 70 % lowest trump, 30 % random filler.

Bot uses same `validate*` & `apply*` for parity.

---

## 5. Firestore Integration

- Store canonical `state` under `games/<built-in function id>`.  
- Clients send **diffs** via `playMove` callable OR directly update with security rules verifying:
  - `request.auth.uid == state.turnPlayerUid`
  - move valid per engine.

Offline fallback: skip Firestore and keep state local with bots.

---

## 6. Minimal Unit‑Test Suite (Vitest)

1. `deal()` → 6 cards each, correct widow length.  
2. `validateBid()` rejects duplicates/lower bids.  
3. `applyPlay()` resolves winner seat correctly.  
4. `scoreRound()` awards all 6 categories.  
5. 6‑and‑Out success/fail path ends game.

---

## 7. File Generation Prompts (Cursor)

- **Deck Utils**  
  > “Create `deck-utils.ts` with `generateDeck`, `shuffle`, and `dealHands`.”

- **Engine**  
  > “Generate `engine.ts` implementing pure functions described in spec.”

- **Tests**  
  > “Write Vitest tests in `engine.spec.ts` covering bid & trick logic.”

---

*End of file.*
