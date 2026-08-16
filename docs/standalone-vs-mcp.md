# Claude + MCP vs a standalone platform

**Suggestion:** do **not** build the AWS stack to get stock analysis or a Kite/Groww portfolio VIEW. That already works as **Claude (or Cursor) + official Kite MCP + Groww MCP** plus the analyser prompt in this repo. A separate platform is justified only for jobs that **must run when you are not in a chat**.

---

## What you already get without a platform

| Need | Claude + MCP |
| --- | --- |
| Indian fundamental report (ticker + horizon, 8-tab VIEW) | Paste/load [`indian-stock-fundamental-analyser.md`](./indian-stock-fundamental-analyser.md); Claude searches and fills the HTML artefact |
| “What’s in my Kite book?” | `https://mcp.kite.trade/mcp` — `get_holdings`, `get_positions`, `get_mf_holdings`, `get_margins`, `get_gtts` |
| Groww holdings / F&O | `https://mcp.groww.in/mcp` |
| Combined overlap / concentration | Ask once both MCPs are connected; Claude can merge by ISIN in the thread |
| No buy/sell from hosted Kite MCP | Hosted Kite **excludes** `place_order` and similar |

You or Claude in Cursor can do this today. No Lambda, no Cockroach, no CloudFront.

**Limits of this mode (not bugs — product gaps):**

- Nothing runs **after market close** unless you open a chat.
- No **score history** (was INFY’s long-term score 78 last month?).
- No **push** alert (Telegram/SES/SNS) if a stop breaks at 9:16 while you are offline.
- **US** names, Polygon/Alpha Vantage, NSE filings, and catalyst decay are not in Kite/Groww MCP.
- RSI/MACD/ATR will **vary** if an LLM recomputes them in prose; they are not a checked-in function.
- Context window: a full book + 8-tab report + peers does not stay cheap or complete every day.
- Session login: MCP OAuth expires; a chat is not a system of record.
- Holdings sit in **chat logs** (Claude/Cursor), which may be a worse privacy story than a box you own.
- Groww MCP: stocks/F&O first; MFs/IPOs may be `DATA UNAVAILABLE`.
- Two brokers in two MCP servers: merge quality depends on the model that day unless you persist ISIN rows.

If that list does not hurt yet, **stay on Claude + MCP**. Treat this repo as prompts + rules, not infra.

---

## What a standalone platform is *for*

Build AWS (or any always-on host) only when you need **at least one** of these. Each is something a chat cannot be.

1. **Scheduler** — EventBridge (or cron): NSE close + US close, holiday calendar, retries. Digest without you prompting.
2. **System of record** — tables: `price_history`, `scores`, `fundamental_reports`, `portfolio_snapshots`, `alerts_log`. Time travel and “what changed vs yesterday.”
3. **Push alerts** — SNS/Telegram/SES on stop-loss or penny gap. Independent of Claude being open.
4. **Deterministic engines** — one Python/TS function for RSI/MACD/ATR and scorecard thresholds. Same input → same number. LLM writes the VIEW from those numbers; it does not invent RSI.
5. **Non-broker data** — US OHLCV, fundamentals APIs, news/filings, FX. Kite/Groww will not be the universe.
6. **Unattended risk** — cap and missing-stop checks on the snapshot at 15:45 IST even if you never logged in.
7. **Cost and variance control** — one batch LLM call for the digest, not a long agent thread every evening.
8. **Your boundary** — dedicated AWS account, Secrets Manager, logs you can delete; not “every portfolio dump in Claude history.”

If you only want (1)–(3), a **single cheap box + cron + SQLite + a Telegram bot** is enough. Full Lambda/SQS/CloudFront is for when you want tear-down IaC and a private web UI.

---

## What is *not* necessary on day one (even if standalone)

Do not treat the original eight-phase AWS sketch as mandatory:

- SQS between fetch and write — add when APIs time out, not before.
- Cockroach/Aurora — SQLite or one Postgres instance is enough for personal use until you have a reason.
- S3 + CloudFront SPA — Claude artefacts + Telegram cover UI until you hate chat.
- Three Lambdas — one Fargate task or one cron script with `--mode=swing|long_term|digest`.
- Amplify, Step Functions, dual `dev`/`prod` — later.
- Re-implementing Kite/Groww as “our MCP inside AWS” — **call their hosted MCP or official APIs**. Do not write a fake MCP for the same `get_holdings`.

**Necessary for a real standalone (minimum):** scheduler, durable store, secrets, one push channel, code-defined scores, read-only broker fetch, holiday-aware jobs, “do not place orders,” citations or `DATA UNAVAILABLE`.

---

## Recommended sequence

**Stage A — Claude-native (now)**  
Kite MCP + Groww MCP + analyser prompt + portfolio questions in chat. Save good reports as files in this repo if you want an archive (manual).

**Stage B — Promote the painful parts**  
When you notice “I forgot to run the brief” or “I need last week’s score,” add **cron + DB + Telegram** only. Keep Claude for on-demand `/analyse`-style reports.

**Stage C — Full dashboard**  
`/analyse` and `/portfolio` as web pages when you want IP-allowlisted UI, snapshots, and deep-links without opening Claude. Broker data still from **the same** Kite/Groww APIs/MCP, not a second source of truth.

Do not start Stage C until Stage A is a habit. A platform you do not open is worse than a prompt you use.

---

## How the two modes should share rules

Same product rules in chat and on the platform:

- No buy / sell / target price; VIEW only.
- Cite source or `DATA UNAVAILABLE`; never fabricate.
- Hosted Kite: no order tools. Groww page/chat: do not call place-order tools.
- Merge brokers by **ISIN**; never double-count.
- Penny last; caps and stops before that bucket.

Platform-only extra: **numbers from code**, LLM for prose. Chat-only extra: live web search for the 8-tab report until ingest exists.

---

## Auth, privacy, failure (missing details)

| Topic | Claude + MCP | Standalone |
| --- | --- | --- |
| Broker login | MCP `login` / OAuth in the client | Kite Connect + Groww tokens in Secrets Manager; refresh; never in the SPA |
| Session death | Re-login in Claude | Job retries + alarm if refresh fails |
| Portfolio in LLM | Full holdings in the prompt | Prefer: compute stats in code, send **summaries** to the LLM for the VIEW |
| Audit | Chat transcript | `alerts_log` + snapshot rows + CloudWatch |
| Who can see UI | Your Claude/Cursor account | IP allowlist / signed URL; still not a multi-user SaaS |
| Legal | Same: not SEBI research, not advice | Same disclaimer on digest and pages |

**Failure modes to design for (platform):** market holiday, MCP/API 429, partial Groww (no MF), one broker disconnected, IST vs US DST, split ISIN, GTT vs “stop in our DB” mismatch.

**Cost sketch (personal):** Claude Pro + MCP ≈ subscription you already pay. Standalone ≈ LLM batch (Bedrock/Anthropic) + tiny compute + DB free tier + SES. The expensive mistake is **both** a daily agent thread **and** a daily Lambda calling the same model.

---

## Mapping old AWS phases → this decision

| Original phase | Claude+MCP | Build standalone only if… |
| --- | --- | --- |
| 1 AWS foundation | Skip | You need secrets, logs, tear-down |
| 2 ingest | Skip (Kite quotes ≠ full US/history) | You want bars every day without asking |
| 3 database | Skip | You want history and snapshots |
| 4 scoring + analyser | Analyser in Claude; swing math unreliable | You want stable RSI and weekly scorecard |
| 5 ledger / risk | Ask Claude to flag caps | You want flags without a chat |
| 6 digest / SNS | You remember to ask | You want email/Telegram unattended |
| 7 dashboard | Artefact + chat | You want `/analyse` and `/portfolio` URLs |
| Kite+Groww page | **This is MCP** | You want a persistent combined table |

Default: **Stage A.** Promote features by pain, not by architecture completeness.
