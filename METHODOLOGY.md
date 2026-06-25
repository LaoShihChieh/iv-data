# IV Data — Methodology

**Educational / informational only. NOT investment advice. May contain gaps or errors.**

This dataset (`iv_data.json`) publishes a daily ATM **implied-volatility** reading and an
**IV-rank** for a basket of liquid US optionable names.

## What the numbers are

- **`iv` — BS-implied IV (a consistent proxy, not vendor "true" IV).** Recovered by
  **Black-Scholes inversion** of the option's price under fixed simplifying assumptions:
  **risk-free rate r = 0, European exercise, no dividends.** Held constant for every name
  and day, so the series is internally consistent and reproducible — but it is a *proxy*,
  and its absolute level differs from a vendor's "true" IV.
- **Contract:** the **put nearest the underlying's spot**, at the **~35-DTE monthly
  expiry** (standard third-Friday expiry, **snapped to the prior trading session when that
  Friday is a market holiday**), inverted from the option's **daily close**.
- **`iv_rank`** — where current `iv` sits within its **trailing 252-session min/max range**
  (0–1). `null` when coverage is insufficient (`n < 2`).
- **`n`** — recorded sessions in the trailing window (coverage). Treat small `n` as
  low-confidence.

## Filtering

A day is recorded only when the option's daily bar has **non-zero volume and trades** and
the inversion is recoverable (price above intrinsic). Failing days are omitted, not guessed.

## Schema (`iv_data.json`)

```json
{
  "meta": { "...": "field descriptions + disclaimer" },
  "as_of": "YYYY-MM-DD",
  "data": [ { "ticker": "SPY", "date": "YYYY-MM-DD", "iv": 0.13, "iv_rank": 0.19, "n": 252 } ]
}
```

Each row is exactly `{ticker, date, iv, iv_rank, n}` — no other fields are published.

## Reference implementation (pseudocode)

```text
spot          = underlying daily close on the session
expiry        = third_friday(~35 DTE);  if holiday -> previous trading session
strike        = listed strike nearest spot
option_price  = ATM put daily close   (skip if volume==0 or trades==0)
iv            = bs_invert_put(option_price, spot, strike, T=(expiry-session)/365, r=0)
iv_rank       = (iv - min(window)) / (max(window) - min(window))   over trailing 252 sessions
```
