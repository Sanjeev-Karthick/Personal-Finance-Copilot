# Personal Finance Copilot

A personal AI co-pilot for **long-term investing**, **swing trading**, and **penny/high-growth** plays across **India** and **US** markets.

The stack is an AWS pipeline (Lambda, CockroachDB or Aurora, SQS/SNS) feeding three scoring engines (fundamentals, technicals, catalyst detection), an **Indian stock fundamental analyser** (ticker + investment horizon), a **separate portfolio page** fed by **Kite and Groww MCP** (read-only holdings), a positions ledger with P&L per strategy, and a daily digest with urgent alerts.

**Build status:** planning only. Sequenced work: [TODO.md](./TODO.md). Samples: [docs/sample-output.md](./docs/sample-output.md). Long-term widget: [docs/indian-stock-fundamental-analyser.md](./docs/indian-stock-fundamental-analyser.md). Portfolio page: [docs/portfolio-analysis.md](./docs/portfolio-analysis.md). Implementation has not started.

## Buckets

| Bucket | Idea | Cadence |
| --- | --- | --- |
| Long-term | Indian analyser: ticker + horizon → 8-tab VIEW | Weekly + on demand |
| Swing | RSI / MACD / volume / ATR | Daily |
| Penny / high-growth | Volume surge + LLM catalyst detection | Daily, last to ship |

Penny/high-growth waits until position caps, stop-loss enforcement, and catalyst-decay checks exist.

## Markets

NSE/BSE and US, one normalized schema tagged by market and currency, with native plus converted P&L.

## What you see

Scores per bucket, an **Indian long-term fundamental report** on `/analyse` (ticker + years → **View**), a **separate `/portfolio` page** that pulls **Kite + Groww** books via MCP and combines them (P&L, overlap, risk flags), a daily digest, and urgent alerts when a stop or held penny blows up.

No buy/sell/target prices on either page. The analyser and portfolio VIEW are not order tickets. The portfolio page never places trades through MCP.
