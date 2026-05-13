# bribes-data_2026

Daily-refreshed history of TLA (Terra Liquidity Alliance) bribes. Powers the "PD influence" analysis on the aDAO website.

Companion cron: [`defipatriot/cron-scripts/bribes-history`](https://github.com/defipatriot/cron-scripts/tree/main/bribes-history)

---

## Directory layout

```
bribes-data_2026/
├── README.md                          ← you are here
└── data/
    ├── pd-bribes-history.json         ← master file: every PD bribe ever
    ├── current-state.json             ← live snapshot of bribe-manager contract
    ├── bribers-registry.json          ← addresses → totals (mostly PD for now)
    └── by-epoch/
        ├── epoch-131.json
        ├── ...
        └── epoch-184.json             ← bribes active in each specific epoch
```

---

## What's in here

This repo is the canonical history of every `add_bribe` execute message PD has ever proposed via their DAODAO governance. Built by decoding the actual on-chain proposal messages (not parsing descriptions), so the data is machine-perfect.

### `pd-bribes-history.json`

Master file. One entry per `add_bribe` message, with full provenance:

| Field | Meaning |
|---|---|
| `proposal_id` | PD DAO proposal that contained this bribe |
| `proposal_title`, `proposal_status` | Human context + executed/passed/failed |
| `briber_address`, `briber_label` | Who placed the bribe (PD's DAO addr → `"PD"`) |
| `bribe_token`, `bribe_amount` | Token denom/contract + amount in micro units |
| `for_pool` | Target LP token (cw20 contract or native denom) |
| `gauge` | TLA gauge bucket: stable/project/bluechip/single |
| `start_epoch`, `end_epoch` | Epoch range the bribe covers |
| `distribution` | Distribution function (`linear` over the epoch range) |

### `current-state.json`

Direct passthrough of the bribe-manager contract's `{bribes:{period:null}}` query. Shows what bribes are CURRENTLY active on chain right now, regardless of who deposited them. Useful for computing "total bribe TVL right now" vs the historical PD-only sum.

### `bribers-registry.json`

For each unique briber address: total bribes count, total LUNA bribed, pools bribed, first/last activity. Currently only PD (since we walk PD's DAO proposals). Future enhancement will scan Terra tx history to capture non-DAO bribers.

### `data/by-epoch/epoch-{N}.json`

Each epoch's file lists every bribe that's active in that specific epoch, with an `amount_this_epoch` field showing the per-epoch portion (= `bribe_amount ÷ epoch_range_length`).

---

## Refresh cadence

The cron runs every 4 hours and rebuilds all files from scratch. Bribes change infrequently (a few times per epoch), so 4-hourly refresh is plenty.

If the cron ever misses runs, the next successful run repairs everything — the cron is stateless.

---

## Data source

- **Terra LCD** (`terra-rest.publicnode.com` with `terra.publicnode.com` fallback)
- **PD's proposal module contract**: `terra1660g9mle5kfsq8c0p4k4hgr9ujdyr3m48c22cawy0akr98rmwksqehqnup`
- **Bribe manager contract**: `terra1tuuwm8yrj54qeg0c8xu00aha9ryatyhtczq8qq2q8tntuw0auzas9037wh`

---

## Schema versioning

Top-level `schemaVersion` field in every JSON file. v1 is the current shape. Bumps will be announced via the `CHANGES_PENDING.md` in `website-adao-core`.
