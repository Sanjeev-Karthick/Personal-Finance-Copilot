# Indian stock fundamental analyser

On-demand **long-term** report for **NSE/BSE** names. The user supplies a **ticker** and an **investment horizon**. The product returns a VIEW (not a buy/sell), in the same 8-tab widget as the Claude artefact: Snapshot, Valuation, Growth, Health, Returns, Peers, Ownership, **View** (default tab on load).

**Run this in Claude first.** It is a prompt + HTML artefact, not an AWS feature. A hosted `/analyse` page is only needed if you want the same widget without opening Claude. See [standalone-vs-mcp.md](./standalone-vs-mcp.md).

This is the long-term analysis UX. Swing and penny stay on other engines (and those *do* need code/schedule if you want them unattended). A US analogue is later work.

**Spec files**
- Operating rules and steps: this document
- Widget markup: [`templates/indian-fundamental-report.html`](./templates/indian-fundamental-report.html)

**Do not implement until a later task kicks off this phase.** Planning only.

---

## Product input (Step 1)

Do not run research until both fields are present.

| Field | Allowed values |
| --- | --- |
| Stock | Company name or NSE/BSE ticker (e.g. TCS, RELIANCE, HDFCBANK, INFY) |
| Horizon | 3 years · 5 years · 10 years · or a custom number of years |

In Claude this is a chat prompt. In the copilot it is the dashboard/API: ticker + horizon, then the same report.

---

## Operating rules (must hold in the product)

1. Both inputs required before any analysis.
2. No language that implies future performance as a recommendation (“this stock should…”, “expected to…”). Horizon projections (Step 7) are labeled as **historical-CAGR scenarios only**, not guarantees.
3. Every metric **cites a source**. If not found → `DATA UNAVAILABLE`. Never estimate or fill gaps.
4. Never fabricate financials. Prefer live fetch (NSE → BSE → Screener.in → Tickertape → Moneycontrol → annual reports → earnings transcripts → Tijori). If live fetch fails: state clearly that figures are missing or stale and must be verified independently.
5. **No buy / sell / target price.** Ever. Output a VIEW. The user decides.
6. Steps 3–11 run in order. Do not skip or merge.
7. Render the **entire** result as the self-contained HTML widget (no wrapping `<html>` / `<head>` / `<body>`). Start with `<style>` then `<div class="wrap">`. **View (tab 7) is active on load.**

Minimum **two sources** per data point when live research runs. Do not expose the raw research checklist to the user.

---

## Research checklist (silent; persist citations)

- Live CMP, 52W high, 52W low, market cap, face value — NSE/BSE
- P/E, P/B, EV/EBITDA — current + sector average + stock 5-year average
- Revenue / net profit / EPS CAGR: 3-year and 5-year
- EBITDA margin and net profit margin: 5-year trend
- EPS: last 8 quarters with YoY change
- Free cash flow: last 3–5 years
- Debt-to-equity: 5-year trend
- Interest coverage, current ratio
- ROE and ROCE: current + 3-year avg + 5-year avg
- Dividend history and payout ratio
- Promoter holding: last 8–12 quarters; pledging flag if **above 10%**
- FII and DII holding: last 8 quarters
- Moat (pricing power, brand, switching costs, market share)
- Sector tailwinds / headwinds (described as context, not a forecast pitch)
- Regulatory risks
- Management: guidance vs delivery, governance flags
- Latest quarterly earnings call: key commentary
- 3 closest peers: P/E, P/B, ROE, revenue growth, D/E
- Top 5 recent news items relevant to long-term holders

---

## Assessment steps (map to widget tabs)

**3 — Valuation**  
Compare current P/E, P/B, EV/EBITDA vs sector avg and own 5-year avg. Per metric: CHEAP / FAIR / EXPENSIVE. Overall: UNDERVALUED / FAIRLY VALUED / OVERVALUED / MIXED.

**4 — Growth**  
Revenue, net profit, EPS, margins. Classify: ACCELERATING / STEADY / SLOWING / DECLINING.

**5 — Health**  
- D/E: below 1 SAFE · 1–2 MODERATE · above 2 LEVERAGED  
- Interest coverage: above 3x HEALTHY · 1.5–3x WATCH · below 1.5 RISK  
- Current ratio: above 1.5 COMFORTABLE · 1–1.5 WATCH · below 1 RISK  
- FCF: positive and growing STRONG · positive flat STABLE · negative CONCERN  

**6 — Returns**  
ROE/ROCE: above 15% GOOD · 10–15% AVERAGE · below 10% WEAK. Dividend consistency and payout sustainability.

**7 — Horizon scenarios**  
Bear / base / bull from **historical CAGR only** for the stated horizon. Not a price target.

**8 — Peers**  
Three closest competitors. Standing: LEADING / MID-PACK / LAGGING.

**9 — Ownership**  
Promoter BUYING / STABLE / SELLING; FII/DII INCREASING / STABLE / DECREASING; pledging FLAG if above 10%.

**10 — View (default tab)**  
One-sentence summary, 3 strengths, 2 watch points, 1 thing to track, overall STRONG / MODERATE / WEAK. Plus opportunities vs risks cards. Not a recommendation.

**11 — Data confidence**  
Live named-source metrics vs DATA UNAVAILABLE. 9–10 sections live = HIGH · 6–8 MODERATE · below 6 LOW (warn) · 0 VERY LOW.

**12 — Render**  
Fill [`templates/indian-fundamental-report.html`](./templates/indian-fundamental-report.html). Missing metric inline: `🚩 DATA UNAVAILABLE — verify at [source URL]`.

---

## How this fits the copilot

| Piece | Role |
| --- | --- |
| **Claude (default)** | This entire spec as a prompt; artefact HTML; web search for metrics |
| API / dashboard (later) | Same widget when you want a URL |
| Ingestion / DB (later) | Citations + weekly refresh so you are not searching from scratch |
| Long-term score (later) | Code consumes CHEAP/FAIR/… classifications |
| Digest (later) | View one-liner without opening Claude |
| Swing / penny | Do **not** use this widget as their primary output |

Claude.ai artefact rules (raw HTML in chat) apply in Claude. A future SPA can host the **same HTML**, View tab default.
