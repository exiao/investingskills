# Investing Skills

Your AI agent lies about stock data. Fix it.

Claude, GPT, Gemini: they all hallucinate financial numbers. Ask for Apple's P/E ratio and you'll get a number from their training data, not today's market. These skills connect your AI agent to [Bloom's](https://getbloom.app) live financial data so every number is real.

5 plain-English skills. No finance jargon. No terminal experience required.

| Skill | What you say | What happens |
|-------|-------------|-------------|
| `/research` | "Tell me about NVDA" | Full analysis: financials, valuation, risks, catalysts, technicals |
| `/review` | "How's my portfolio?" | Letter grade A-F, top problems, actionable suggestions |
| `/discover` | "Find me something to buy" | AI Arena trades, screener picks, market movers, curated ideas |
| `/check` | "Should I buy TSLA here?" | Data-backed verdict: ✅ do it, ⚠️ be careful, or 🛑 rethink |
| `/briefing` | "What happened today?" | 2-minute market catch-up with your watchlist |

## Why this exists

60 million Americans manage their own investments. The average retail investor spends 6 minutes researching a trade. You're making $5,000+ decisions with less thought than you put into a restaurant choice.

These skills give your AI agent access to the same data Wall Street uses: real-time prices, analyst ratings, earnings data, technical indicators, and something nobody else has: **3 AI models trading real money** in Bloom's AI Arena.

## Install

### 1. Get the Bloom CLI (the data layer)

```bash
npm install -g @bloomai/cli
bloom auth login
```

Test it:
```bash
bloom info AAPL
```

You should see Apple's live financials. If that works, you're good.

### 2. Install the skills

**Claude Code:**
```bash
git clone https://github.com/exiao/investingskills.git ~/.claude/skills/investingskills
```

**Any other agent (Cursor, Windsurf, Codex):**
```bash
git clone https://github.com/exiao/investingskills.git
```
Then point your agent's skills directory to the `skills/` folder.

That's it. Start with `/research AAPL` and see real data.

## The skills

### `/research` — Full stock analysis

Ask about any stock. Get a structured report with real numbers.

```
/research NVDA
```

Returns: what the company does, revenue growth, margins, free cash flow, analyst consensus, upcoming catalysts, risks, technical levels, recent news, and a bottom line verdict. Every metric is color-coded 🟢/🟡/🔴 so you can scan it in 30 seconds.

### `/review` — Portfolio report card

Paste your holdings. Get graded.

```
/review AAPL 40%, GOOGL 30%, MSFT 30%
```

Returns: overall grade (A-F), your 3 biggest problems, holdings breakdown, sector exposure, risk assessment, and specific suggestions. Grades across 5 categories: Diversification, Valuation, Growth, Income, Risk.

### `/discover` — What's worth looking at

No input needed. Just run it.

```
/discover
```

Returns: what 3 AI models are buying and selling with real money (AI Arena), today's biggest movers, stocks passing value+growth screens, curated collections, and market sentiment. The AI Arena section is unique to Bloom: Claude, GPT, and Gemini each manage a real portfolio. See who's winning.

### `/check` — Sanity check a trade

Describe what you're thinking. Get a data-backed verdict.

```
/check Buy NVDA before earnings, AI spending is accelerating
```

Returns: ✅ Looks Good, ⚠️ Proceed With Caution, or 🛑 Rethink This. Plus: evidence for and against, key numbers, timing considerations, and a suggested position size. The trade equivalent of "let me sleep on it" but with actual data.

### `/briefing` — 2-minute market catch-up

Get today's market summary. Add your watchlist for personalized coverage.

```
/briefing
/briefing AAPL NVDA TSLA
```

Returns: market mood (Fear & Greed, VIX), major indexes, biggest movers, your watchlist tickers, relevant headlines, and one thing to watch. Everything you need in 2 minutes.

## How it works

Every skill runs commands from the [Bloom CLI](https://www.npmjs.com/package/@bloomai/cli), which pulls live data from Bloom's financial API. The skills are markdown files that tell your AI agent which commands to run, what to extract, and how to format the output.

```
Your question → AI agent reads skill → runs bloom CLI commands → real data → structured answer
```

No scraping. No stale training data. No hallucinated numbers.

## What makes this different

**vs. ChatGPT/Claude/Gemini alone:** They guess. We pull. Ask any LLM for a P/E ratio and compare it to `bloom info`. You'll see the difference.

**vs. InvestSkill (18 skills):** Built for analysts who know what Piotroski F-Scores are. We built for people who just want to know "should I buy this?"

**vs. tradermonty (12 skills):** Requires FMP API keys and CSV downloads. We work out of the box after `bloom auth login`.

**vs. yfinance wrappers:** yfinance scrapes Yahoo Finance, gets IP-blocked at scale, and breaks randomly. Bloom's API is maintained and reliable.

**The thing nobody else has:** 3 AI models managing real money in Bloom's AI Arena. `/discover` shows you what Claude, GPT, and Gemini are actually buying and selling. Not backtests. Not paper trading. Real money.

## Skill architecture

```
┌─────────────────────────────────┐
│          bloom-data             │
│  (CLI setup, auth, data rules)  │
└──────────────┬──────────────────┘
               │ every skill reads this first
    ┌──────────┬──────────┼──────────┬──────────┐
    ▼          ▼          ▼          ▼          ▼
/research   /review   /discover   /check    /briefing
```

`bloom-data` is the foundation. It checks prerequisites, defines output rules, and documents every CLI command. The 5 skills build on top of it.

## Contributing

Found a bug? Want to improve a skill? PRs welcome.

The skills are markdown files in `skills/`. Edit the SKILL.md, test it, submit a PR.

## License

MIT

---

Built by [Eric](https://github.com/exiao) · Powered by [Bloom](https://getbloom.app) · Data from [@bloomai/cli](https://www.npmjs.com/package/@bloomai/cli)
