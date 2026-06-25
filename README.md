# iv-data

Daily **BS-implied implied volatility** and **IV-rank** for a basket of liquid US
optionable names (the universe is based on Alpaca options-data availability).

- **`iv_data.json`** — the dataset: one row per ticker, `{ticker, date, iv, iv_rank, n}`.
- **[`METHODOLOGY.md`](METHODOLOGY.md)** — how the numbers are computed.

## What the numbers are

- `iv` is **BS-implied** (Black-Scholes inversion of the ATM put's daily close;
  r=0, European, no dividends) — a **consistent proxy, not vendor "true" IV**.
- `iv_rank` is where current `iv` sits within its trailing 252-session min/max range
  (0–1); `n` is the coverage (sessions in the window).

See [`METHODOLOGY.md`](METHODOLOGY.md) for the full computation and schema.

## Disclaimer

Educational / informational only. **Not investment advice.** May contain gaps or errors.
