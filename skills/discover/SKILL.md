---
name: discover
description: >
  Find investment opportunities. Combines AI Arena trades (what 3 AI models are doing with real money),
  today's movers, screener picks, and curated collections into one actionable summary.
  Triggers: "find me something to buy", "what should I invest in", "what's looking good",
  "any good opportunities", "what are the AIs buying", "what's trending", "show me ideas",
  "discover stocks", "what's worth looking at", "give me investment ideas"
---

# discover — Find Opportunities

> Every number comes from Bloom's financial data. Not a language model's training data.

**Trigger:** `/discover` — when a user wants investment ideas or to see what's moving.

No input required. Just run it.

---

## Read `bloom-data` Skill First

Before running anything, check that `bloom` is installed and authenticated. See the `bloom-data` skill for prerequisites and output rules.

```bash
mkdir -p /tmp/bloom
```

---

## Steps (Run in Order)

### 1. AI Arena trades
```bash
bloom ai-trades --limit 10 -o /tmp/bloom/discover-ai-trades.json
```

### 2. AI Arena current portfolios
```bash
bloom ai-portfolio -o /tmp/bloom/discover-ai-portfolio.json
```

### 3. Today's biggest movers
```bash
bloom market --type top_movers --limit 10 -o /tmp/bloom/discover-movers.json
```

### 4. Value collection
```bash
bloom collections --section VALUE -o /tmp/bloom/discover-value.json
```

### 5. Growth collection
```bash
bloom collections --section GROWTH -o /tmp/bloom/discover-growth.json
```

### 6. Screener — value + growth filter
```bash
bloom screen "market_cap > 10B" "pe_ratio < 20" "revenue_growth > 10" -o /tmp/bloom/discover-screen.json
```

### 7. Market sentiment
```bash
bloom sentiment -o /tmp/bloom/discover-sentiment.json
```

### 8. Extract key fields
```bash
# AI trades
cat /tmp/bloom/discover-ai-trades.json | jq '.trades[:5] | .[] | {model, ticker, direction, return_pct}'

# AI portfolios (who's winning)
cat /tmp/bloom/discover-ai-portfolio.json | jq '.portfolios | to_entries[] | {model: .key, total_return: .value.total_return, top_holding: .value.holdings[0]}'

# Top movers
cat /tmp/bloom/discover-movers.json | jq '.[:5][] | {ticker, change_pct, price, reason}'

# Screener picks
cat /tmp/bloom/discover-screen.json | jq '.[:5][] | {ticker, pe_ratio, revenue_growth, market_cap}'

# Sentiment
cat /tmp/bloom/discover-sentiment.json | jq '{fear_greed_index, fear_greed_label, aaii_bull, aaii_bear, vix}'
```

---

## Sentiment Context

Use this to frame the "Market Mood" section:

| Fear & Greed | AAII Bull% | What It Means |
|-------------|------------|---------------|
| 0-25 (Extreme Fear) | <30% | Market is panicking — historically good time to buy quality names |
| 25-45 (Fear) | 30-40% | Cautious market — look for beaten-down quality |
| 45-55 (Neutral) | ~40% | Normal conditions — look for value |
| 55-75 (Greed) | >50% | Market is optimistic — be selective |
| 75-100 (Extreme Greed) | >60% | Euphoria — proceed carefully, valuations stretched |

---

## Output Format

```
## What's Worth Looking At

### 🤖 AI Arena — What 3 AIs Are Doing With Real Money
*These are real portfolios with real money, run by Claude, GPT, and Gemini.*

| Model | Position | Direction | Ticker | Return |
|-------|----------|-----------|--------|--------|
| Claude | [Last trade] | 🟢 Long / 🔴 Exit | [TICKER] | [+X%] |
| GPT | [Last trade] | 🟢 Long / 🔴 Exit | [TICKER] | [+X%] |
| Gemini | [Last trade] | 🟢 Long / 🔴 Exit | [TICKER] | [+X%] |

**Who's winning:** [Model] is up [X]% overall.
**What they agree on:** [If 2+ models hold the same stock, call it out]

---

### 📈 Today's Big Movers
| Ticker | Change | Why |
|--------|--------|-----|
| [TICKER] | +X% | [One-line reason if available] |
| [TICKER] | -X% | [One-line reason if available] |
*[Top 5 movers, both up and down]*

---

### 🔍 Screener Picks
*Stocks with market cap >$10B, P/E under 20, and revenue growing >10%. The "boring but solid" list.*

| Ticker | P/E | Revenue Growth | Market Cap |
|--------|-----|----------------|------------|
| [TICKER] | [X] | [+X%] | [$XB] |
*[Up to 5 results. If none pass the filter, say so and relax one criterion.]*

---

### 🎯 Curated Collections
**Value picks:** [Top 2-3 from bloom collections VALUE]
**Growth picks:** [Top 2-3 from bloom collections GROWTH]

---

### 🌡️ Market Mood
**Fear & Greed:** [X] ([Label — e.g. "Fear"])
**Investors:** [X]% bullish / [X]% bearish (AAII survey)
**VIX:** [X]

[One sentence: what does this mean for the ideas above?
Example: "Market is in Fear territory — historically a decent time to add to quality positions."]

---
*Data from Bloom CLI. Source commands: bloom ai-trades, bloom ai-portfolio, bloom market, bloom collections, bloom screen, bloom sentiment.*
```

---

## Notes

- The AI Arena is the most unique section — lead with it. Real AI models, real money, real returns.
- If a stock appears in both AI Arena trades AND the screener, call that out explicitly as a double signal.
- Don't invent reasons for movers. If `bloom market` doesn't return a reason, just show the price change.
- The screener is intentionally conservative (large cap, cheap, growing). Mention this so users understand what they're looking at.

---

## Related Skills

- `bloom-data` — Prerequisites and CLI reference (read this first)
- `research` — Deep dive on any name that catches your eye here
- `check` — Before acting on any idea, run /check on the trade thesis
- `briefing` — For broader market context on what's driving today's movers
