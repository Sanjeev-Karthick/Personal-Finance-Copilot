# Sample output (illustrative)

Figures are **not** live market data. They show the product shape after Friday **14 Aug 2026** close: inputs → scores → **stock analysis** → ledger → digest → optional urgent alert.

The analysis is written by an LLM **only from stored bars, fundamentals, scores, filings, and your positions**. It does not place orders.

For **NSE/BSE long-term**, the product also takes **ticker + investment horizon** and renders the 8-tab fundamental widget (View default). Spec: [indian-stock-fundamental-analyser.md](./indian-stock-fundamental-analyser.md). That report is a VIEW with citations — not a buy/sell. The INFY write-up below is a short copilot note; the widget is the full long-term artefact.

---

## Input

**Universe (excerpt)**

| Ticker | Market | Currency | Bucket interest |
| --- | --- | --- | --- |
| INFY | NSE | INR | long-term, swing |
| AAPL | US | USD | long-term, swing |
| XYZ.NS | NSE | INR | penny (held) |

**Price bar (INFY, 14 Aug)**  
OHLC 1,612 / 1,641 / 1,605 / 1,638, volume 8.4M (20-day avg 5.1M).

**Fundamentals (INFY, latest filing, simplified)**  
IT services; revenue +8% YoY; operating margin 21%; ROE 31%; net cash; PE ~22 vs 5-year median ~24.

**Your ledger**  
Long-term: 40 INFY @ 1,480 INR, stop 1,350.  
Penny: 2,000 XYZ.NS @ 18.40 INR, **no stop**. Cap: penny book ≤ 10% of equity.

**Penny news (XYZ.NS, 14 Aug 16:10 IST)**  
NSE announcement: “board to consider fundraising; no terms disclosed.”

**FX**  
83.2 INR/USD.

---

## Output — scores

```json
{
  "as_of": "2026-08-14",
  "scores": [
    {
      "ticker": "INFY",
      "market": "NSE",
      "bucket": "long_term",
      "score": 78,
      "label": "constructive",
      "why": [
        "ROE and FCF cover the quality bar",
        "PE 8% below 5y median",
        "balance sheet still net cash"
      ]
    },
    {
      "ticker": "INFY",
      "market": "NSE",
      "bucket": "swing",
      "score": 71,
      "label": "setup",
      "why": [
        "MACD histogram flipped positive",
        "RSI 58, not stretched",
        "volume 1.6× 20-day average",
        "ATR 2.1% → stop zone ~1,572 if you trade the swing"
      ]
    },
    {
      "ticker": "AAPL",
      "market": "US",
      "bucket": "long_term",
      "score": 61,
      "label": "hold / wait",
      "why": ["quality fine", "valuation above your cheapness band"]
    },
    {
      "ticker": "XYZ.NS",
      "market": "NSE",
      "bucket": "penny_growth",
      "score": 44,
      "label": "do not add",
      "why": [
        "volume surge yes (3.2×)",
        "catalyst weak: fundraising rumor, no terms",
        "catalyst freshness: 4 hours (ok)",
        "BLOCKERS: no stop-loss on open position; penny book 12% > 10% cap"
      ]
    }
  ]
}
```

---

## Output — Indian fundamental analyser (long-term)

**Input:** ticker `INFY` · horizon `5 years` (both required; no report until both exist).

**Output:** HTML widget from [`templates/indian-fundamental-report.html`](./templates/indian-fundamental-report.html), **View tab open first**.

| Tab | What you see |
| --- | --- |
| Snapshot | Company, sector, CMP / 52W / mcap / face value, flags (e.g. pledging) |
| Valuation | P/E, P/B, EV/EBITDA vs sector and 5Y own avg → CHEAP / FAIR / EXPENSIVE |
| Growth | 3Y/5Y CAGRs, 8-quarter EPS, ACCELERATING / STEADY / SLOWING / DECLINING |
| Health | D/E, coverage, current ratio, FCF + bear/base/bull **CAGR scenarios for 5 years** (not a price target) |
| Returns | ROE / ROCE / dividend |
| Peers | 3 competitors + 5 long-term news items |
| Ownership | Promoter / FII / DII / pledge + earnings-call notes |
| **View** | STRONG / MODERATE / WEAK, 3 strengths, 2 watches, 1 thing to track |

Every cell cites NSE, BSE, Screener.in, etc., or shows `DATA UNAVAILABLE`. Confidence bar: HIGH / MODERATE / LOW / VERY LOW. **No buy, sell, or target price.**

The INFY prose below is the copilot’s short cross-bucket note (includes swing). The widget is the full long-term artefact.

## Output — stock analysis (dashboard ticker page)

This is the extra layer: a readable note per name, not only a score. NSE long-term still uses the widget above as the source of truth.

### INFY (NSE) — as of 14 Aug 2026

**Snapshot:** Last 1,638 INR. Held: 40 shares long-term @ 1,480, stop 1,350. Unrealized +6,320 INR (~$76).

**Business / quality**  
Large-cap IT services. Scorecard sees high ROE (31%), 21% operating margin, and a net-cash balance sheet. Growth is mid-single to high-single digit (+8% revenue YoY in the stored print) — acceptable for long-term, not a hyper-growth story.

**Valuation**  
PE ~22 vs your 5-year median ~24 (~8% cheaper). Long-term engine: **78 / constructive**. Not a deep-value name; quality at a slight discount to its own history.

**Technicals (swing)**  
Close near the day’s high on 1.6× average volume. MACD histogram just flipped positive; RSI 58. ATR 2.1%. Swing engine: **71 / setup**. If you treat this as a swing (separate from the long-term lot), invalidation near **1,572** (about 1× ATR under the close). That is *not* the 1,350 long-term stop.

**Catalyst**  
None required for these two buckets. No penny-style filing in the ingest for INFY today.

**Risks**  
IT spending cycle; INR/USD translation; a failed swing if volume was a one-day burst. Long-term stop 1,350 is far under last — you are giving the core holding room.

**Implied action (not an order)**  
| Bucket | Bias | Note |
| --- | --- | --- |
| Long-term | See analyser VIEW | 5-year horizon widget; not a buy/sell |
| Swing | Setup, optional | Only with a ~1,572 invalidation and size that does not collide with the long-term lot |
| Penny | n/a | |

---

### AAPL (US) — as of 14 Aug 2026

**Snapshot:** Watchlist only (no position in the sample ledger).

**Business / quality**  
Quality clears the long-term bar (franchise, cash generation). Stored fundamentals are not the constraint.

**Valuation**  
Above your cheapness band. Long-term: **61 / hold / wait**. Fine company, not “good” as a buy under *your* value rule.

**Technicals**  
No US swing setup cleared the daily bar in this run.

**Catalyst**  
None in the sample ingest.

**Risks**  
Paying up for quality; multiple compression if rates or earnings disappoint.

**Implied action**  
Long-term: wait. Swing: none. Do not force a buy because the brand is strong.

---

### XYZ.NS (NSE) — as of 14 Aug 2026

**Snapshot:** Last 21.10 INR. Held: 2,000 penny @ 18.40, **no stop**. Unrealized +5,400 INR (~$65). Penny book **12% vs 10% cap**.

**Business / quality**  
Penny bucket does **not** treat this as a long-term quality compounder. No full fundamental pass in this sample.

**Valuation**  
Not the driver today. Price is up on volume, not on a completed fundraise.

**Technicals**  
Volume **3.2×** 20-day average (surge flag = yes).

**Catalyst**  
Board “to consider fundraising; no terms.” LLM tag: **weak / incomplete**. Freshness ~4 hours, so not decayed yet — but there is no size, price, or dilution math to underwrite.

**Risks**  
Dilution, gap risk, and **process breaks**: missing stop, over cap. A high-volume day without terms is not a thesis.

**Implied action**  
Penny: **do not add**. First jobs: set a stop or cut to the 10% cap. Score **44** is a warning, not a breakout buy.

---

## Output — positions + P&L

| Bucket | Name | Qty | Avg | Last | P&L native | P&L USD | Stop | Flags |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| long-term | INFY | 40 | 1,480 | 1,638 | +6,320 INR | +$76 | 1,350 | — |
| penny | XYZ.NS | 2,000 | 18.40 | 21.10 | +5,400 INR | +$65 | — | **no stop**, **over cap** |

---

## Output — daily digest (email / Telegram)

> **Copilot brief — 14 Aug 2026 (NSE + US close)**  
> **Long-term:** INFY 78 constructive — quality + slight cheapness vs its own PE history; hold the 40-share lot (stop 1,350). AAPL 61 wait — quality fine, valuation not in band.  
> **Swing:** INFY 71 setup (MACD + volume); optional ~1,572 invalidation, separate from the long-term stop. No US swing cleared.  
> **Penny:** XYZ.NS 44 — volume without a real catalyst (fundraising, no terms). **Do not add.** Open XYZ has **no stop** and penny exposure is **over cap**.  
> Full write-ups: INFY, AAPL, XYZ.NS on the dashboard. Not advice — your rules, your size.

---

## Output — urgent alert (SNS)

No stop-price breach in this sample, so SNS is quiet. Missing stop and cap breach still appear in **analysis**, **ledger flags**, and the **digest**.

If XYZ later traded 16.20 with a stop at 17.00:

```text
URGENT penny | XYZ.NS last 16.20 < stop 17.00 | P&L -4,400 INR | 2026-08-17 09:16 IST
```
