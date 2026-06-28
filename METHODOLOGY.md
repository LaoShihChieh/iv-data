# IV Data — Methodology

**Educational / informational only. NOT investment advice. May contain gaps or errors.**

`iv_data.json` publishes a daily strategy-relevant fact set for a basket of liquid US
optionable names. **Two data sources** (stated honestly — the data is not single-source):

- **Facts 1-7 and 9 — Alpaca.** BS-implied IV, split-only-adjusted equity history, and
  option quotes/bars.
- **Fact 8 (market cap) — Finnhub** (`/stock/profile2` company market cap). **ETFs show
  N/A** because a fund has no company market cap — a category distinction, not missing data.

## The 8 facts (per ticker)

1. **typical_iv** — median of the **BS-implied** ATM-put IV over the trailing **252**
   sessions. BS inversion uses **r=0, European, no dividends** — a *consistent proxy*, not
   vendor "true" IV. Contract: put nearest spot at the ~35-DTE monthly expiry (third
   Friday, snapped to the prior session on holidays), inverted from the daily close.
   Coverage in `iv_coverage_n`.
2. **realized_vol** — annualized stdev of **21-trading-day-horizon** daily log returns
   (split-adjusted), median over the trailing **252** — **matched to the IV horizon** so
   the IV-vs-realized comparison is apples-to-apples.
3. **iv_minus_rv** — `typical_iv - realized_vol` (variance-risk-premium proxy).
4. **iv_rank** — where current IV sits within its trailing-252 min/max range (0-1); null
   when coverage < 2.
5. **drawdown** — max peak-to-trough on **split-adjusted** closes, over **1/3/5yr**
   (pre-computed), with `recovery_days` (back to the prior peak) or `underwater_pct`
   (still below). Split-only (not dividend-adjusted) — the price drawdown a holder
   experiences.
6. **largest_down_day** — worst single-day **close-to-close DOWN** move (split-adjusted),
   over **1/3/5yr**. Downside, close-to-close (a put-seller's tail; matches how the
   strategy marks).
7. **spread_pct** — current ATM-put bid/ask spread from the **INDICATIVE feed** — a
   *modeled, time-sensitive snapshot*, **NOT a reliable point-in-time liquidity read**. Lead
   with `option_volume`; a wide spread on a name with healthy volume is an indicative artifact.
8. **market_cap** — Finnhub company market cap (USD) + **band** (Mega ≥ $200B / Large
   $10-200B / Mid $2-10B / Small $300M-$2B / Micro < $300M). ETFs: N/A.
9. **option_volume** — trailing ~10-session **average daily volume** of the shared ~35-DTE
   ATM-put contract (the **genuine** liquidity-depth signal). `option_volume_days` is how many
   sessions the average covers; a low count (e.g. 1) is a single noisy day, not a stable read.

**Other honest caveats.** Open interest is **not available** from the data source (not
published — never estimated). A few names are **spin-off-adjusted** (total-return basis, so a
structural share-distribution isn't mis-read as a market crash). One name (**PSKY**, the
post-2025-merger Paramount Skydance) has **< 1 year of history** — its windows reflect that
short, single-entity track record, not a multi-year drawdown.

## Windows & coverage

IV facts are capped at ~252 sessions (data-limited ~1yr). Equity facts (drawdown,
largest move) use deep history and are pre-computed for **1/3/5yr** so the scanner can
toggle client-side without recompute. Every windowed fact carries its `coverage_days`;
deeper windows can legitimately exceed the IV window — labeled, never silently compared.

## Schema (`iv_data.json`)

```json
{
  "meta": { "...": "field descriptions + two-vendor provenance + disclaimer" },
  "as_of": "YYYY-MM-DD",
  "data": [ {
    "ticker": "NVDA", "typical_iv": 0.38, "iv_rank": 0.25, "iv_coverage_n": 252,
    "realized_vol": 0.33, "iv_minus_rv": 0.05,
    "drawdown":        { "1y": {"max_dd": -0.20, "recovery_days": 25, "underwater_pct": null, "coverage_days": 251}, "3y": {...}, "5y": {...} },
    "largest_down_day":{ "1y": {"pct": -0.06, "date": "YYYY-MM-DD", "coverage_days": 251}, "3y": {...}, "5y": {...} },
    "spread_pct": 0.02,
    "market_cap": {"usd": 4815000000000, "band": "Mega", "source": "finnhub"}
  } ]
}
```
