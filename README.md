# Personal Balance Sheet — Finance Dashboard

A single-page, static personal finance dashboard for tracking cash, savings,
and a BTC/XAUT/USDT DCA strategy on OKX — net worth, asset allocation,
DCA zone signals, sell/realized P&L, and more, all rendered client-side
from JSON data pulled from a Google Sheet.

No backend, no build step — it's one `index.html` file plus a `data/`
folder of JSON snapshots kept fresh by a scheduled GitHub Action.

## How it works

```
Google Sheet  ──(GitHub Action, every 8h)──▶  data/*.json  ──(fetch, client-side)──▶  index.html
```

1. **`.github/workflows/fetch-sheet.yml`** runs on a schedule (and can be
   triggered manually), pulls four tabs from a published Google Sheet via
   its `gviz` JSON endpoint, and commits the results into `data/`:
   - `Savings Dashboard` → `data/savings-dashboard.json`
   - `DCA Log` → `data/dca-log.json`
   - `DCA Rules` → `data/dca-rules.json`
   - `Monthly Balance` → `data/monthly-balance.json`
2. **`index.html`** fetches those JSON files directly (no server needed —
   works from GitHub Pages or any static host) and renders everything in
   the browser.
3. Risk-score history for BTC and XAUT is pulled live, read-only, from two
   companion repos ([`btc-risk-score`](https://github.com/Yamand/btc-risk-score),
   [`gold-risk-score`](https://github.com/Yamand/gold-risk-score)) and drives
   the DCA zone temperature gauges.

## Features

**Overview**
- Total net worth hero figure with a cash-vs-OKX split bar
- DCA Zone Temperature gauges for BTC and XAUT (cold = accumulate, hot = trim)
- Financial-health panels: debt payoff, milestone ladder, Coast FI, emergency fund
- **Asset Allocation** — two breakdowns, each toggleable between a treemap
  and an interactive donut chart with click-to-drill-down into underlying
  accounts/holdings:
  - *Current Allocation* — Cash / BTC / Gold / Stablecoin
  - *By Purpose* — Cash / Savings / Emergency Fund / Investment
- Sell Summary — realized P&L and sell activity per asset

**Cash & Bank** — every bank/cash account, balances in native currency and USD

**Holdings — DCA Positions** — live BTC/XAUT/USDT positions and cost basis

**Performance — Equity Curve** — net worth trend over time

**Trade Log — Weekly Trades** — full DCA buy/sell history

**Risk Scores** — score history, tier streaks, and score-component breakdowns
that determine each week's DCA zone

## Setup

1. Publish your Google Sheet to the web (or share it so the `gviz` endpoint
   is publicly readable) and add its ID as a repository secret:
   - **Settings → Secrets and variables → Actions → New repository secret**
   - Name: `SHEET_ID`
2. Make sure your sheet has tabs named exactly `Savings Dashboard`,
   `DCA Log`, `DCA Rules`, and `Monthly Balance` — the workflow fetches
   these by name.
3. Run the **Fetch Google Sheet Data** workflow manually once (Actions tab
   → *Run workflow*) to populate `data/` for the first time.
4. Serve `index.html` — either enable **GitHub Pages** (Settings → Pages →
   deploy from the `main` branch) or open/serve it locally.

To point the risk-score gauges at your own data, edit `RISK_SOURCES` near
the top of the `<script>` in `index.html`.

## Local development

No build step — just serve the folder so `fetch()` can load the local
JSON files (opening `index.html` directly via `file://` won't work in most
browsers):

```bash
python3 -m http.server 8000
# then open http://localhost:8000
```

## Tech stack

- Vanilla HTML/CSS/JS — no framework, no bundler
- [Chart.js](https://www.chartjs.org/) + [chartjs-plugin-datalabels](https://chartjs-plugin-datalabels.netlify.app/)
  for the interactive allocation donuts
- Hand-rolled SVG squarified treemap for the allocation treemaps
- Google Sheets (via the `gviz` JSON endpoint) as the only "backend"
- GitHub Actions for scheduled data refresh

## Repo structure

```
.
├── index.html                    # the entire app
├── data/
│   ├── savings-dashboard.json    # cash/bank accounts + OKX balances
│   ├── dca-log.json              # trade history
│   ├── dca-rules.json            # DCA zone rules per asset
│   └── monthly-balance.json      # net-worth trend
└── .github/workflows/
    └── fetch-sheet.yml           # scheduled Google Sheet → JSON sync
```

## Disclaimer

This is a personal tracking tool, not financial advice. DCA zones, target
allocations, and risk scores reflect one person's strategy and assumptions
— adjust `TARGET_ALLOCATION`, `SELL_TIER_MIN_HOLDING`, and the DCA rules
sheet to match your own.
