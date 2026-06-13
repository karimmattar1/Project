# Project Vision — Working Name: "Moneyball" (TBD)

> A real-money **Last Man Standing** football platform built around one idea:
> **make the pot snowball, and pull more people in.** Buy-backs grow the prize as
> rounds progress and as players lose lives; the pot becomes the marketing; the
> growth feeds itself.

_Status: living brainstorm doc. Nothing here is final. Decisions still open are
listed at the bottom._

---

## 1. The one-liner

Last Man Standing pools, done properly: pick one team to win each week, lose a
life when you're wrong, buy back in to stay alive, last survivor(s) take a pot
that grows every single round. Public mega-pools anyone can join, plus private
pools for friend groups — the way we already play it, but at scale and for real
money.

## 2. The core game (Last Man Standing)

- Each round (gameweek or knockout tie), every active player picks **one team to win**.
- **Win** → you survive to the next round.
- **Not a win** (loss **or draw**) → you lose a **life**. Win-or-bust: a draw
  costs you, which drives faster attrition and bigger pots.
- **0 lives** → eliminated… unless you **buy back in**.
- **Classic rule:** you can't pick the same team twice in a run. This is what
  creates strategy — you have to ration your strong teams across the season.
- **Last player(s) standing** win the pot. If everyone's eliminated in the same
  round, tie-break / split rules apply.
- Picks **lock at kickoff** of the first eligible fixture. No late edits.

## 3. The heart: pot-maximisation & buy-back economics

This is the actual product. Everything else serves it.

**Lives + buy-backs as the growth engine:**
- Lives are **purchasable, capped at 2 held at any time** — you can pay to top
  back up to 2, but never stockpile more. The cap is the anti-"buy-the-win"
  guardrail.
- **Any non-win (loss or draw) costs a life.** Win-or-bust.
- Losing a life is the trigger to spend money to stay in.
- **Buy-backs feed the pot AND generate rake** — an eliminated player is normally
  lost revenue and lost engagement; a buy-back re-monetises them, re-engages
  them, and grows the headline pot that attracts everyone else.

**Escalation curves (the levers that make the pot snowball):**
- **Round-escalating price** — buy-back gets more expensive the later the round
  (e.g. R1 = £5, mid-season = £25, late = £50+). Fits knockouts perfectly:
  group stage → R16 → QF → SF → final, each tier pricier.
- **Life-escalating price** — each successive buy-back costs the same player more
  (£5 → £10 → £20…). Adds drama and stops bottomless re-entry.
- Likely **both**, combined.

**Guardrails so it stays a game, not pay-to-win:**
- **Buy-back window** — re-entry only allowed up to round X (e.g. before halfway),
  so the endgame is "pure" survivors.
- **Buy-back cap** — max re-entries per player, so a whale can't simply buy the win.

**Payout:** pot splits among the **top 10** finishers (weighted, e.g. 1st gets the
biggest slice down to 10th). More winners keeps people chasing deep into the season.

**Worked example (illustrative):**
- 10,000 entrants × £10 entry = £100,000 gross. Rake 15% → **£15,000 to operator**,
  **£85,000 seed pot**.
- Over the season, say 6,000 buy-backs at an average £20 = £120,000 gross →
  another **£18,000 rake**, **£102,000 added to pot**.
- Headline pot ≈ **£187,000**, operator revenue ≈ **£33,000**, split across the
  top 10 — and the big pot is itself the ad that recruits the next cohort.

## 4. The growth / outreach flywheel

Bigger pot → more attractive → more entrants → bigger pot. Accelerators:
- **Referrals** — invite a friend; both get a discounted entry or a bonus life.
- **Public mega-pools** (anyone joins) alongside **private pools** (friend groups —
  our original use case).
- **Shareable hooks** — live "X players left, £Y pot" ticker; "you survived
  round N" cards built for the group chat.
- **Seasonal spikes** — World Cup / Euros pools for mass, casual influx.

## 5. Monetisation

- **Rake** on every entry and every buy-back (tunable per contest, ~10–20%).
- Buy-backs are the compounding revenue line — the more rounds, the more they fire.
- Later: premium analytics subscription, sponsorships on big public pools.

## 6. Integrity (the other half of "profit max WITH integrity")

Real money means trust is the product. Non-negotiables:
- Picks lock at kickoff; no edits after.
- Can't reuse a team in a run.
- **Auto-pick / default rule** if a player forgets (rule TBD — auto-assign vs.
  eliminate).
- **Results from an official data feed**, never set by a human. Immutable resolution.
- Pot accounting fully transparent; rake and prize split disclosed up front.
- Published tie-break rules.

## 7. Real-money licensing reality (the long pole)

Decision taken: **licensed real-money from day one.** This is legitimate and
buildable, but it is the slowest, most expensive track, so we run it in parallel
with the product build. What it requires:
- **Gaming licence** in a chosen jurisdiction (e.g. UK Gambling Commission, Malta,
  Isle of Man).
- **KYC / AML** identity verification and anti-money-laundering checks.
- **Age verification** (18+).
- **Geo-fencing** to block disallowed regions.
- **Responsible-gambling tooling** — deposit limits, self-exclusion, reality checks.
- **Segregated player funds** (player money held separately from operating funds).
- **Gambling-friendly payment processor** (most mainstream PSPs won't touch it).
- **Tax / regulatory reporting** in the licensed jurisdiction.

> The product and tech can be built and tested (in play-money mode) while the
> licence is in progress. Real-money switches on once licensed in a given market.

## 8. Competitions & contest-type roadmap

- **Weekly engine:** Premier League (a pick every week — the core habit).
- **Cup runs:** Carabao, FA Cup, UCL, UEL — sharp knockout survivor pools.
- **Tournaments:** World Cup / Euros — big seasonal pools + "bet to win the
  tournament" (outright) contests.
- **Later modes (same wallet/account):** classic weekly prediction, fantasy,
  draft. Earned once the LMS core is humming.

## 9. The "Moneyball brain" (Phase 2 — differentiator, not core)

A pick co-pilot to beat every spreadsheet/WhatsApp pool out there:
- **Safest pick this week** from your *unused* teams (form, fixtures, xG, home/away,
  rotation/injury risk).
- **Pick popularity / game theory** — "73% of survivors picked Liverpool" →
  differential strategy.
- **Forward planner** — plan picks across upcoming weeks so you don't burn strong
  teams early. Pure Moneyball: managing a scarce resource.

## 10. Phased roadmap (draft)

1. **Phase 0 — Spec & rules lock-in** (this doc → finalised rules + economics).
2. **Phase 1 — Play-money MVP:** LMS engine, pools (public + private), pick lock,
   results ingestion, lives + buy-backs, pot accounting, leaderboards. Premier
   League only.
3. **Phase 2 — Compliance + real money:** licence, KYC, payments, responsible
   gambling, geo-fencing → flip on real money.
4. **Phase 3 — Reach:** cups + tournaments, referrals, shareable cards, the
   Moneyball brain.
5. **Phase 4 — New modes:** prediction / fantasy / draft / outrights.

## 11. Locked decisions

- **Lives:** purchasable, **capped at 2** held at any time.
- **Any non-win (loss or draw) costs a life** — win-or-bust.
- **Buy-back curve:** escalates by **both round and number of buy-backs used**.
- **Payout:** **top 10** split (weighted).
- **Hero product:** Last Man Standing; other modes later.
- **Money:** licensed real-money from day one.

## 12. Open decisions (need your steer)

- **Buy-back window:** how late into a season/tournament can you still re-enter?
- **Rake %:** target operator cut (~10–20%?).
- **Top-10 weighting:** how steep is the split from 1st to 10th?
- **Launch jurisdiction:** which market do we license for first? (Gates everything
  in Phase 2.)
- **Pools at launch:** public mega-pools, private friend pools, or both?
- **Same-team-twice rule:** on (classic) or relaxed?
- **Name & brand:** "Moneyball" is taken/risky as a trademark — need a real name.
