# Portfolio analysis page (Kite + Groww MCP)

A **separate dashboard route** from the Indian fundamental analyser. That page is “one ticker + horizon → VIEW.” This page is “**your actual books** at Zerodha Kite and Groww → combined portfolio VIEW.”

**Do not implement until a later task kicks this off.** Planning only. **Read-only.** This page must not place, modify, or cancel orders even if a self-hosted MCP exposes those tools.

---

## Product surface

| Route | Purpose | Inputs |
| --- | --- | --- |
| `/analyse` (existing) | Long-term fundamental widget | NSE/BSE ticker + years |
| **`/portfolio` (this spec)** | Broker-true holdings, P&L, risk, overlap | Connect Kite and/or Groww |

Two pages in the same SPA (or two Amplify routes). Do not merge them into one scroll.

---

## MCP sources (official hosted, preferred)

| Broker | Hosted MCP | Notes |
| --- | --- | --- |
| **Zerodha Kite** | `https://mcp.kite.trade/mcp` ([kite-mcp-server](https://github.com/zerodha/kite-mcp-server)) | Hosted instance **excludes** destructive trading tools. Login via MCP `login`. |
| **Groww** | `https://mcp.groww.in/mcp` ([Groww MCP](https://groww.in/updates/groww-mcp)) | Official hosted connector for Claude/Cursor. Stocks and F&O first; MFs/IPOs later per Groww. |

Cursor / Claude (local agent) config examples:

```json
{
  "mcpServers": {
    "kite": {
      "command": "npx",
      "args": ["mcp-remote", "https://mcp.kite.trade/mcp"]
    },
    "groww": {
      "command": "npx",
      "args": ["mcp-remote", "https://mcp.groww.in/mcp"]
    }
  }
}
```

The **web page** cannot call those URLs from the browser. A personal backend (API Gateway + Lambda) either:

1. Uses the same **read** APIs the MCPs wrap (Kite Connect + Groww APIs) with tokens in Secrets Manager, or  
2. Acts as an MCP client to the hosted servers after OAuth, server-side only.

Document which path you pick when implementation starts. The **tool contract** below is the product requirement either way.

---

## Read tools this page is allowed to use

**Kite (portfolio & account only)**  
`get_profile` · `get_margins` · `get_holdings` · `get_positions` · `get_mf_holdings` · `get_gtts` (to see whether stops exist) · quotes/LTP as needed for mark-to-market.

**Kite — do not wire on `/portfolio`**  
`place_order`, `modify_order`, `cancel_order`, `place_gtt_order`, `modify_gtt_order`, `delete_gtt_order`.

**Groww**  
Holdings, positions, margins/balance, quotes/LTP for MTM. Same rule: **no order placement** from this page even if community Groww MCPs expose `place_order`.

If a broker is disconnected, show that column as `NOT CONNECTED` and still analyse the other book.

---

## What the page shows

1. **Connection status** — Kite logged in / Groww logged in / last sync IST.  
2. **Combined holdings** — one row per ISIN: qty and avg at each broker, last price, unrealized P&L native, converted INR+USD, weight %.  
3. **Broker split** — % of equity at Kite vs Groww (and MF sleeve on Kite `get_mf_holdings` if present).  
4. **Overlap** — same ISIN at both brokers (double-count risk).  
5. **Allocation** — sector, market cap bucket, single-name concentration, top 10 weights.  
6. **Strategy buckets** — map each holding to long-term / swing / penny (manual tag or rule); P&L and weight **per bucket**.  
7. **Risk flags** (same rules as the ledger) — penny/growth over cap, name over cap, **no GTT/stop** where the bucket requires one, F&O notional if Groww/Kite positions include derivatives.  
8. **Reconcile** — vs the copilot `positions` table: in-broker-not-in-ledger, in-ledger-not-in-broker.  
9. **VIEW** — short LLM summary grounded **only** in MCP/API payloads + your caps. No buy/sell/target. Cite broker as source (`Kite holdings`, `Groww positions`). Missing field → `DATA UNAVAILABLE`.

Refresh: on-demand button + optional daily pull after NSE close (same EventBridge window as ingest). Persist a snapshot to S3/`portfolio_snapshots` so the digest can mention “books vs yesterday.”

---

## Operating rules

- Personal use only; tokens never in git or the SPA.  
- Do not treat MCP chat in Cursor as a substitute for this page; the page is the durable UI.  
- Groww MCP currently emphasizes stocks and F&O; if MFs are unavailable, say so — do not invent NAVs.  
- Combined totals must **not** double-count an ISIN held at both brokers.  
- This page does not replace `/analyse`. A holding can deep-link to `/analyse?ticker=INFY&horizon=5`.

---

## First slice

Kite **or** Groww connected, combined table if both, P&L + weights, overlap, two risk flags (cap, missing stop), VIEW paragraph. F&O detail and MF sleeve can follow.
