# PotShot — Full Build Specification

> **PotShot** is a licensed real-money **Last Man Standing** football platform.
> Pick one team to win each round; any non-win costs a life; buy lives/buy back in
> to stay alive; the **pot snowballs** via escalating buy-backs; the **top 10**
> finishers split it. Public mega-pools + private friend pools.
>
> This document is the single source of truth for the build. Start from "Part J —
> Getting Started" to scaffold.

---

## Part A — Glossary

| Term | Meaning |
|---|---|
| **Competition** | A real-world tournament season, e.g. "Premier League 2025/26" or "UCL 2025/26". |
| **Round** | One pickable matchday within a competition (a league gameweek or a knockout tie). |
| **Fixture** | A single match within a round (home vs away, kickoff time, result). |
| **Pool** | A contest instance players join. Has an entry fee, ruleset, pot, and a competition. |
| **Entry** | One user's participation in one pool. Holds lives, status, and the pick history. |
| **Pick** | An entry's chosen team to win in a given round. Locks at first kickoff. |
| **Life** | A "strike" buffer. Lose one on any non-win. Capped at 2 held at once. |
| **Buy-back** | Purchasing a life — either topping up (insurance) or re-entering after elimination. |
| **Pot** | The prize fund for a pool = sum of (entry + buy-back) money after rake. |
| **Rake** | The operator's cut (20%) taken from every entry and buy-back. |
| **Window** | The period during which buy-backs are allowed (closes part-way through). |

---

## Part B — Core Game Rules

### B1. The pick
- Each round, every **active** entry selects exactly **one team** that is playing
  in that round, predicting it to **win**.
- A pick **locks** at the kickoff of the **first fixture of that round**. After lock:
  no edits, no new picks.
- **No-reuse rule:** within a single pool run, an entry may **not** pick the same
  team more than once. Already-picked teams are greyed out.

### B2. Resolution (what happens to a pick)
- **WIN** (picked team wins in normal time / per competition rules) → pick `WON`,
  entry survives, no life lost.
- **NON-WIN** (draw **or** loss) → pick `LOST`, entry loses **one life**.
- **VOID** (match postponed/abandoned before a valid result) → pick `VOID`, no life
  lost, team becomes re-pickable. (Configurable: void vs. carry-over.)

### B3. Lives
- An entry holds **0, 1, or 2** lives. **Hard cap = 2.**
- Lives are **purchasable** any time before the round locks, provided the result
  would not exceed the cap of 2.
- `lives` decrements by 1 on each non-win.
- After a round settles:
  - `lives > 0` → entry stays `ACTIVE`.
  - `lives == 0` **and** buy-back window open → entry is `ELIMINATED` (can buy back).
  - `lives == 0` **and** window closed → entry is `OUT` (permanent).

### B4. Default / missed pick (integrity rule)
- If an active entry fails to pick before lock, the system **auto-picks** for them
  to keep it fair. **Default rule (configurable):** auto-assign the entry's
  **highest win-probability unused team** in that round. (Alternative configs:
  random unused team, or treat as a non-win.)

### B5. Winning
- A pool ends when the round structure is exhausted (season/tournament over) **or**
  when ≤ a configured threshold of entries remain.
- The **top 10** surviving entries split the pot by the weighting in Part D.
- Ranking among survivors uses (in order): rounds survived → lives remaining →
  fewest buy-backs used → earliest entry timestamp (tie-break).

---

## Part C — Buy-back & Lives Economics (the engine)

### C1. Pricing formula
The price to buy one life escalates by **round** and by **prior purchases**:

```
price(round, prior_purchases) =
    ENTRY_FEE
    × (1 + ROUND_STEP × (round_index - 1))     // gets pricier as rounds progress
    × (USE_BASE ^ prior_purchases)              // gets pricier the more you've bought
```

**Default tunables (per pool):**
- `ENTRY_FEE` = pool's entry fee (e.g. £10)
- `ROUND_STEP` = 0.10 (league) — +10% of entry per round elapsed
- `USE_BASE` = 2.0 — each successive buy-back doubles the multiplier
- For **knockouts**, replace the round term with tier multipliers:
  `GROUP=1.0, R16=1.5, QF=2.0, SF=3.0, FINAL=4.0`.

### C2. Example schedule (ENTRY_FEE = £10, league)

| Round | 1st buy-back | 2nd buy-back | 3rd buy-back |
|---|---|---|---|
| 1 | £10.00 | £20.00 | £40.00 |
| 5 | £14.00 | £28.00 | £56.00 |
| 10 | £19.00 | £38.00 | £76.00 |
| 15 | £24.00 | £48.00 | £96.00 |

### C3. Buy-back window
- Buy-backs allowed up to **round W** (default: ~60% through the competition; for
  knockouts, **closes after the quarter-finals**). After W, `lives == 0` = `OUT`.

### C4. Rake & pot accounting
- On every money-in event (entry or buy-back): `rake = amount × 0.20`,
  `pot_contribution = amount × 0.80`.
- `pool.pot` = Σ pot_contributions. `operator_revenue` = Σ rake.
- All movements recorded in a **double-entry ledger** (Part G).

### C5. Payout (top-10 weighting)
Default weights (sum = 100%):

| Rank | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 |
|---|---|---|---|---|---|---|---|---|---|---|
| % of pot | 30 | 18 | 12 | 9 | 7 | 6 | 5 | 5 | 4 | 4 |

- If fewer than 10 survivors, undistributed weight rolls up proportionally to the
  survivors (configurable: roll-up vs. carry to next pool).

### C6. Illustrative pool economics (10k entrants, £10 entry, 20% rake)
- Entries: 10,000 × £10 = £100,000 → **£20,000 rake**, **£80,000 to pot**.
- Buy-backs: ~6,000 × ~£20 avg = £120,000 → **£24,000 rake**, **£96,000 to pot**.
- **Pot ≈ £176,000**, **operator revenue ≈ £44,000**, split across top 10.

---

## Part D — Pools

### D1. Types
- **Public mega-pool:** operator-created, anyone can join, one big shared pot.
  Drives the "pot as marketing" flywheel.
- **Private pool:** user-created, invite-only (share code/link), friend groups.
  Same engine, smaller pots.

### D2. Lifecycle (state machine)
```
DRAFT → OPEN (registration) → LOCKED (round 1 kickoff) → RUNNING
      → SETTLING (final round settled) → COMPLETE (payouts done)
                                       ↘ CANCELLED (refund path)
```
- Registration may stay open for a few rounds (late entry) or close at round 1 —
  configurable per pool.

### D3. Pool config object (per pool)
```
{
  competitionId, name, visibility: PUBLIC|PRIVATE,
  entryFee, currency,
  rake: 0.20,
  startingLives: 1, lifeCap: 2,
  drawCountsAsLoss: true, noReuse: true,
  buyback: { roundStep: 0.10, useBase: 2.0, windowRound: <W>, tierMultipliers: {...} },
  payout: { topN: 10, weights: [30,18,12,9,7,6,5,5,4,4], shortfall: ROLLUP },
  defaultPick: HIGHEST_PROB | RANDOM | TREAT_AS_LOSS,
  lateEntryUntilRound: <n|null>
}
```

---

## Part E — Competition & Round Modelling

- **Leagues** (EPL): rounds = gameweeks; one pick per gameweek.
- **Knockouts** (UCL/UEL/FA/Carabao): rounds = ties; tier multipliers apply.
- **Tournaments** (World Cup/Euros): group rounds + knockout rounds; plus a parallel
  **outright "win the tournament"** contest type (Phase 4).
- Each round has: `lockAt` (first fixture kickoff), `status`, list of fixtures.
- **Result ingestion** from the data feed updates fixtures; a **settlement job**
  resolves picks once all of a round's relevant fixtures are final.

---

## Part F — Data Model (Postgres / Prisma)

Core entities and key fields (not exhaustive):

```
User            id, email, displayName, authProviderId, kycStatus, ageVerified,
                country, selfExclusion, createdAt
Wallet          id, userId, currency, balanceCached
LedgerEntry     id, walletId, type(DEPOSIT|WITHDRAWAL|ENTRY|BUYBACK|PAYOUT|RAKE|REFUND),
                amount, balanceAfter, refType, refId, createdAt   // double-entry
Competition     id, provider, providerId, name, season, format(LEAGUE|KNOCKOUT|TOURNAMENT),
                status
Round           id, competitionId, index, label, lockAt, status(SCHEDULED|LOCKED|IN_PLAY|SETTLED)
Fixture         id, roundId, homeTeamId, awayTeamId, kickoffAt, status, homeScore,
                awayScore, result(HOME|DRAW|AWAY|VOID)
Team            id, competitionId, providerId, name, shortName, crestUrl
Pool            id, competitionId, ownerId(null=operator), visibility, config(jsonb),
                status, potCached, joinCode
Entry           id, poolId, userId, lives, status(ACTIVE|ELIMINATED|OUT|WINNER),
                buybacksUsed, joinedAt, finalRank
Pick            id, entryId, roundId, teamId, status(PENDING|LOCKED|WON|LOST|VOID),
                isAutoPick, lockedAt
LifePurchase    id, entryId, roundId, kind(TOPUP|BUYBACK), price, ledgerEntryId, createdAt
Payout          id, poolId, entryId, rank, amount, ledgerEntryId, paidAt
```

**Key invariants**
- `Entry.lives` ∈ [0, 2]; never exceeds `config.lifeCap`.
- One `Pick` per `(entryId, roundId)`; team unique per entry when `noReuse`.
- Wallet balance must equal Σ ledger entries (reconciliation job).
- `pool.potCached` must equal Σ ledger `pot_contribution` (entry+buyback minus rake).

---

## Part G — Wallet, Payments & Integrity

- **Double-entry ledger** is the source of truth; `Wallet.balanceCached` is a cache.
- Money-in (entry/buyback): write `RAKE` + pot allocation atomically in a DB tx.
- **Idempotency keys** on all money operations; never double-charge on retries.
- **Result resolution is machine-only** — sourced from the data feed, never set by
  a human. Settlement is deterministic and re-runnable.
- **Pick lock** enforced server-side against `round.lockAt` (UTC); client clock
  never trusted.
- **Audit log** for every state transition (entry status, pick lock, settlement).

---

## Part H — Real-Money Compliance (Phase 2 gate)

Built in parallel; real money flips on per licensed market.
- **Gaming licence** (jurisdiction TBD — UK GC / Malta MGA / IoM).
- **KYC/AML** via provider (Onfido / Veriff / Sumsub): identity + sanctions checks.
- **Age verification** (18+) gating any real-money action.
- **Geo-fencing** — block disallowed regions (IP + KYC address).
- **Responsible gambling** — deposit limits, cool-off, self-exclusion, reality checks.
- **Segregated player funds** — player balances held separately from operating funds.
- **Gambling-friendly PSP** for deposits/withdrawals.
- **Reporting/tax** per licence.

---

## Part I — Architecture & Tech Stack

### I1. Stack (TypeScript end-to-end)
- **Web app:** Next.js (App Router) + React + Tailwind. Deploy on Vercel.
- **API:** Next.js route handlers for the app; a separate **worker service** for
  scheduled jobs (round locking, ingestion, settlement, reconciliation).
- **DB:** PostgreSQL (Neon or Supabase) via **Prisma** ORM.
- **Cache/locks:** Redis (Upstash) — distributed locks for settlement, pick-lock,
  idempotency.
- **Auth:** Auth.js (NextAuth); later wired to KYC provider.
- **Data feed:** football API (API-Football / Sportmonks to start; Opta/Stats
  Perform for production-grade). Abstract behind a `FootballDataProvider` interface.
- **Payments (Phase 2):** PSP + ledger; **play-money wallet** for MVP.
- **Observability:** structured logging, Sentry, a settlement dashboard.

### I2. Background jobs
- `lockRounds` — at each `round.lockAt`, lock picks, auto-pick missing.
- `ingestResults` — poll/stream fixture results from the provider.
- `settleRound` — when a round's fixtures are final: resolve picks, decrement lives,
  transition entries, recompute pot, advance pool.
- `settlePool` — on completion: rank survivors, compute & write payouts.
- `reconcile` — verify wallet balances vs. ledger; alert on drift.

### I3. Key API surface (MVP)
```
POST  /api/auth/*                         // signup/login
GET   /api/competitions                   // list active competitions
GET   /api/pools?visibility=PUBLIC        // browse pools
POST  /api/pools                          // create private pool
POST  /api/pools/:id/join                 // join (debits wallet, creates Entry)
GET   /api/pools/:id                      // pool detail: pot, players left, my entry
GET   /api/pools/:id/rounds/:roundId      // fixtures + my pick + pickable teams
POST  /api/entries/:id/pick               // make/change pick (pre-lock only)
POST  /api/entries/:id/buy-life           // buy life / buy back (price from formula)
GET   /api/pools/:id/leaderboard          // standings, survivors, eliminations
GET   /api/wallet                         // balance + ledger history
```

### I4. Core state machines
```
Pool:  DRAFT → OPEN → LOCKED → RUNNING → SETTLING → COMPLETE | CANCELLED
Round: SCHEDULED → LOCKED → IN_PLAY → SETTLED
Entry: ACTIVE → (ELIMINATED ↔ ACTIVE via buy-back) → OUT | WINNER
Pick:  PENDING → LOCKED → WON | LOST | VOID
```

---

## Part J — User Journeys (screens)

1. **Onboarding:** sign up → (Phase 2: KYC + age) → wallet funded (play-money in MVP).
2. **Lobby:** browse public mega-pools (pot size, players left, entry fee, round) +
   "Create private pool" + "Join with code".
3. **Join:** confirm entry fee → wallet debit → Entry created (starting lives).
4. **Make pick:** see this round's fixtures, win-prob hints, greyed-out used teams,
   countdown to lock → submit pick.
5. **Buy life / buy back:** see live price (round + uses), confirm → wallet debit →
   lives updated / re-entered.
6. **Round results:** see your pick outcome, lives remaining, who got knocked out,
   updated pot.
7. **Leaderboard:** survivors, eliminations, pot ticker, "you're in the top N".
8. **Payout:** final standings → top-10 split credited to wallets.

---

## Part K — Phased Roadmap

- **Phase 0 — Spec lock** (this doc).
- **Phase 1 — Play-money MVP (web, EPL only):** auth, play-money wallet+ledger,
  public + private pools, fixtures/results ingestion, pick + lock + auto-pick,
  lives + buy-backs (full pricing engine), settlement, leaderboard, top-10 payout.
- **Phase 2 — Compliance + real money:** licence, KYC, geo, responsible gambling,
  PSP, segregated funds → flip real money on.
- **Phase 3 — Reach:** cups + tournaments, referrals, shareable result cards,
  the Moneyball brain (safest-pick, pick-popularity, forward planner).
- **Phase 4 — New modes:** classic prediction, fantasy, draft, outright tournament bets.

### MVP "definition of done"
A user can: sign up → join a play-money EPL pool → make a weekly pick that locks at
kickoff → lose a life on a non-win → buy a life / buy back at the formula price →
see the pot grow → see the leaderboard → and on pool completion, the top 10 are paid
from the pot. All money flows recorded in a reconciled ledger.

---

## Part L — Getting Started (scaffold)

```bash
# 1. Scaffold the web app
npx create-next-app@latest potshot --typescript --tailwind --app --eslint
cd potshot

# 2. Add core deps
npm install @prisma/client zod
npm install -D prisma
npm install next-auth
npm install ioredis            # Redis client (Upstash in prod)

# 3. Init Prisma (Postgres)
npx prisma init --datasource-provider postgresql
#   → put the Part F schema into prisma/schema.prisma, then:
npx prisma migrate dev --name init

# 4. Suggested structure
#   /app            → routes + pages (lobby, pool, pick, wallet)
#   /app/api        → route handlers (Part I.3)
#   /lib/game       → rules engine (pricing, lives, settlement, payout)
#   /lib/data       → FootballDataProvider interface + adapter (API-Football)
#   /lib/db         → prisma client
#   /lib/wallet     → ledger + idempotency
#   /worker         → cron jobs (lockRounds, ingestResults, settleRound, settlePool)

# 5. First slices to build, in order
#   a) Prisma schema + migrations (Part F)
#   b) Pure rules engine in /lib/game with unit tests:
#        - buybackPrice(round, priorPurchases, config)
#        - applyResult(entry, pickResult) → new lives/status
#        - rankSurvivors(entries) and computePayouts(pot, survivors, weights)
#   c) FootballDataProvider stub returning fixtures/results (swap real feed later)
#   d) Join → pick → lock → settle happy path end-to-end (play money)
#   e) UI: lobby, pick screen, leaderboard
```

**Build the rules engine as pure, well-tested functions first** — pricing, life
resolution, ranking, payouts. Everything else (UI, feeds, payments) plugs into it.
This is the heart of PotShot and must be provably correct before real money touches it.
