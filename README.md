[![make.fun · token-radar-01](https://make.fun/badge/token-radar-01.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![rank](https://make.fun/badge/token-radar-01/sun-justin/token-radar-01/rank.svg)](https://make.fun/bounty/token-radar-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![reward](https://make.fun/badge/token-radar-01/sun-justin/token-radar-01/reward.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)

# Launch Radar v1: 198 rows, 40 watchlist, value fitted on 119 token peers, held-out tested twice

A token launch radar for make.fun's `token-radar-01` bounty ($1,325, closes 14 Dec 2026 17:00 WIB).
Every pre-token protocol in the sealed snapshot, ranked by the token value its own
figures imply — with a farm plan for each one.

## The board at a glance

| Figure | Value |
| --- | --- |
| Board rows | **198** — the full 40-protocol $50M watchlist + 158 early finds below the floor |
| Snapshot | 758 protocols at TVL ≥ $5M, as of 2026-11-14 |
| Total TVL on the board | $20,707,499,265 |
| Total implied value | $1,901,280,109 |
| Median implied / TVL | 0.27× (global peer median 0.25×) |
| Rank 1 | Steakhouse Financial — $78.7M implied on $2.75B TVL |
| Farm plans | 6 concrete steps per protocol, each with a cost and a time |

## How the value was fitted — every term fitted, tested held-out twice

The model is fitted at runtime from the snapshot's own token-bearing protocols
("peers": 119 with a market cap, a TVL, and no exchange token). No protocol name
is ever special-cased.

```
implied = exp(a + b · ln TVL + category residual) × growth × chains × maturity
```

- `a = 7.7881`, `b = 0.4721` — OLS fit of log market cap on log TVL over the
  119 peers. `b < 1` means the mcap/TVL multiple shrinks as protocols grow.
- **Category residual** — each category's median log-residual among its peers,
  shrunk toward zero by `n / (n + 10)`, so a category with one or two peers
  barely moves the value and a category with many peers earns its own multiple.
- **Growth** — `1 + 0.5 × clamp(change_7d, −60%, +120%) / 100`.
- **Chains** — `1 + 0.06 × ln(number of chains)`.
- **Maturity** — `0.70` for a newborn protocol to `1.00` at 3 years
  (unknown `listed_at` scores as 365 days).

The fit is tested, not trusted — held out twice, fitted on the train peers only:

| Test | Train | Held out | Median \|log error\| | Within 2× | Baseline median \|log error\| | Baseline within 2× |
| --- | --- | --- | --- | --- | --- | --- |
| Split A — every 3rd peer held out | 80 | 39 | **1.0264** | 26% | 1.2740 | 26% |
| Split B — every odd peer held out | 60 | 59 | **1.2923** | 31% | 1.7308 | 27% |

The baseline is a single global median mcap/TVL fitted on the same train peers.
The fitted model beats it on both splits. The honest reading is the size of the
errors: a typical held-out market cap lands about e^1.03 ≈ 2.8× away from the
prediction. Implied value is an order-of-magnitude claim — and the board ranks
by it anyway, because ranking by TVL alone would crown the biggest protocol
and ignore what the market pays per TVL dollar.

## Verdicts

Each row carries a verdict from the same two signals: **TOP FARM** is a top-10
implied value, or an implied/TVL of 0.50× or more, with 7-day growth at or
above zero; **COOLING** is 7-day growth below −10%; **WATCH** is an implied/TVL
of 0.30× or more, or a top-40 rank; everything else is **EARLY**.

## What the board is — and is not

- The board ranks **pre-token** protocols: ones that record no token (no market
  cap, no CoinGecko id, no symbol). Wrapped, staked, bridged and versioned
  products are excluded, as are exchanges — a wrapped token is not a launch
  waiting to happen.
- The watchlist is the floor, never the ceiling: the board carries all 40 of
  its protocols plus every other rankable pre-token protocol in the snapshot.
- The 24 largest token-bearing protocols are shown separately as a **Live
  tokens** comparison table; their market values are the market's own, never
  mixed into the ranked board.
- Implied value does **not** predict whether or when a token launches. It says
  what a token would be worth if the protocol's record stayed as it is. A
  protocol that "records no token" may still have one its DefiLlama record
  does not carry.

## The dashboard

A static, no-network page (`dashboard/index.html` + `app.css` + `app.js`,
data from `data.js` written by `main.ts`):

- a dense dark board — big sortable table, stat strip, verdict per row;
- a detail view for every protocol: the model breakdown term by term, the
  nearest live comparables, and the 6-step farm plan with cost and time;
- filters for category, chain, minimum TVL, watchlist-only, and search;
- a live-token comparison table and the full "How the value was fitted" section.

## Run it

```sh
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input <snapshot.json|directory>
```

Node 22, standard library only, no network at runtime: it reads the snapshot,
fits the model, writes `board.json`, `data.js`, copies `dashboard/` and
`screenshots/` into the output directory, and writes `dossier.json` with
`buildDossier` from `vendor/makefun-board.ts`. Both runs write the same bytes.

## Files

| File | What it is |
| --- | --- |
| `main.ts` | Entry point: reads the snapshot, builds the board, writes the outputs |
| `lib/model.ts` | The value model, the held-out tests, the farm plans, the board builder |
| `dashboard/` | The static dashboard (`index.html`, `app.css`, `app.js`) |
| `screenshots/` | `1-board.png` (the board), `2-detail.png` (a detail view), `3-filtered.png` (filtered view) |
| `vendor/makefun-board.ts` | make.fun's dossier library, copied byte-identical from the kit |

## License

MIT — see [LICENSE](LICENSE).
