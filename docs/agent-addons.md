# Features on top of Kite (Zerodha) MCP + agent analysis

**Kite MCP is Zerodha.** You do not add a second “Zerodha MCP.” Optional second broker: **Groww MCP**. The agent already can pull holdings, positions, margins, MFs (Kite), GTTs, and quotes, then write a VIEW.

This list is **what to add on top of that**. Still VIEW-only: no buy/sell/target, no `place_order`. Cite or `DATA UNAVAILABLE`.

Default: extra **prompts, local files, and more MCPs**. Promote to cron/DB only when the gap is “must run while I am offline” ([standalone-vs-mcp.md](./standalone-vs-mcp.md)).

---

## Tier 1 — still just the agent (highest leverage)

| Feature | What the agent does | Extra input |
| --- | --- | --- |
| **Book vs thesis** | Each holding tagged long-term / swing / penny; flag names with no thesis file | `holdings-thesis.md` in this repo |
| **GTT / stop coverage** | Kite `get_gtts` vs holdings; “held, no GTT” | Your rule: penny must have a stop |
| **Concentration** | Weights, top 10, sector, single-name vs your caps | Caps in a small `risk-rules.md` |
| **Broker overlap** | Same ISIN on Kite and Groww; do not double-count MTM | Groww MCP |
| **Cash vs invested** | `get_margins` + holdings MTM | — |
| **F&O sleeve** | Positions notional vs equity; “this is not the long-term book” | Groww/Kite positions |
| **What changed** | Diff vs last saved snapshot you pasted or committed | `snapshots/YYYY-MM-DD.md` (manual at first) |
| **Pre-mortem** | For one ticker: what would invalidate the VIEW | Analyser prompt + your horizon |
| **Tax lots (India)** | STCG vs LTCG using *holding date from broker* only; never guess | Holdings fields; else DATA UNAVAILABLE |
| **Rebalance VIEW** | “Over cap / under cash” in words, not an order ticket | Caps file |

---

## Tier 2 — more tools the agent can call (not AWS)

Kite does **not** give a full fundamental tape, US listings, or NSE announcement history. Add MCPs or APIs the agent is allowed to use:

| Add-on | Why |
| --- | --- |
| **Web / Screener / NSE / BSE** (already in the analyser spec) | 8-tab long-term report per name in the book |
| **Earnings calendar** | Next result date for held ISINs; “no VIEW change until print” |
| **Filings / corporate actions** | Bonus, split, rights, buyback — only if sourced |
| **Benchmark** | Book vs Nifty 50 / Nifty Midcap; excess is descriptive, not a forecast |
| **US data MCP** (Polygon, Alpha Vantage, etc.) | ADRs or US names Kite will not price as your India book |
| **News restricted to held ISINs** | Avoid market-wide noise; penny catalyst only on names you own |
| **Bank/FD/gold CSV** | Net worth VIEW; brokers are not the whole life |
| **Mutual funds** | Kite `get_mf_holdings`; Groww MF may be unavailable — say so |

Do not add a trading MCP that can place orders “for convenience.”

---

## Tier 3 — agent skills (prompts), not product pages

Keep as reusable prompts in `docs/` or Claude projects:

1. **Daily book brief** — holdings + GTT gaps + one news item per name, 15 lines.  
2. **Weekly fundamental** — run the 8-tab analyser only for names over X% weight or tagged long-term.  
3. **Swing tape** — RSI/MACD only if you later pin formulas in code; until then label as *indicative*.  
4. **Earnings week** — held names reporting in 7 days; no new penny adds.  
5. **Post-trade journal** — you paste a fill; agent files it under the bucket and updates thesis.  
6. **Family / second PAN** — only if you ever connect another Kite login; isolate books.

---

## Tier 4 — only if chat is not enough

Same as the standalone doc: scheduled brief, score history, Telegram if a GTT-less penny moves, deterministic indicators, `/portfolio` URL.

Skip until you miss them. They do not make the agent “smarter”; they make it **on time**.

---

## Do not add (or add last)

- Auto-orders, GTT placement from the agent, “AI will exit for you”
- Price targets and “should buy”
- A second Zerodha integration beside Kite MCP
- Scraping unofficial apps for holdings
- Social sentiment as a score
- Penny bucket before caps + stop rules exist

---

## Sensible order for you

1. `risk-rules.md` + `holdings-thesis.md` + GTT coverage prompt (Tier 1).  
2. Groww MCP if you actually hold there; overlap VIEW.  
3. Analyser on **top holdings only**, not the whole universe.  
4. Manual snapshot files so “what changed” works.  
5. Earnings/filings for held names.  
6. US/news MCP only if those names are in the book.  
7. Cron/Telegram only after you use 1–4 for a while.
