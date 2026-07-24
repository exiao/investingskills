---
name: briefing
description: >
  Daily market update. Covers sentiment, major indexes, biggest movers, and optionally
  a watchlist. 2-minute read. Run it in the morning to know what's happening.
  Triggers: "what happened today", "market update", "morning briefing", "how's the market",
  "what's moving today", "market summary", "catch me up on markets", "daily briefing",
  "what's the market doing", "give me a market update", "briefing"
---

# briefing — Daily Market Update

> Every number comes from Bloom's financial data. Not a language model's training data.

**Trigger:** `/briefing [WATCHLIST]` — market update, optionally with your tickers.

Examples:
- `/briefing` — full market summary, no specific tickers
- `/briefing AAPL MSFT NVDA` — add watchlist section for those tickers
- `/briefing TSLA AMZN GOOGL META` — track your portfolio tickers

---

## Read `bloom-data` Skill First

Before running anything, check that `bloom` is installed and authenticated. See the `bloom-data` skill for prerequisites and output rules.

```bash
mkdir -p /tmp/bloom
```

---

## Steps (Run in Order)

### 1. Market sentiment
```bash
bloom sentiment -o /tmp/bloom/briefing-sentiment.json
```

### 2. Major indexes
```bash
bloom market --type major_indexes -o /tmp/bloom/briefing-indexes.json
```

### 3. Today's biggest movers
```bash
bloom market --type top_movers --limit 5 -o /tmp/bloom/briefing-movers.json
```

### 4. Watchlist info (only if tickers provided)
```bash
bloom info <TICKER1> <TICKER2> ... -o /tmp/bloom/briefing-watchlist.json
```

### 5. Watchlist news (only if tickers provided)
```bash
bloom news <TICKER1> <TICKER2> --limit 3 -o /tmp/bloom/briefing-news.json
```

### 6. Extract key fields
```bash
# Sentiment
cat /tmp/bloom/briefing-sentiment.json | jq '{fear_greed_index, fear_greed_label, aaii_bull, aaii_bear, vix}'

# Indexes
cat /tmp/bloom/briefing-indexes.json | jq '.[] | {name, price, change_pct, ytd_change}'

# Movers
cat /tmp/bloom/briefing-movers.json | jq '.[] | {ticker, change_pct, price, reason}'

# Watchlist (if applicable)
cat /tmp/bloom/briefing-watchlist.json | jq '.[] | {symbol, price, change_pct, analyst_consensus}'

# News (if applicable)
cat /tmp/bloom/briefing-news.json | jq '.[].headline'
```

---

## Index Trend Icons

| Change | Icon |
|--------|------|
| +1% or more | 🟢 |
| -0.5% to +1% | 🟡 |
| -1% to -0.5% | 🟡 |
| -1% or worse | 🔴 |

---

## Sentiment Context

| Fear & Greed | What It Tends to Mean |
|-------------|----------------------|
| 0-25 Extreme Fear | Oversold market — quality names get cheap |
| 25-45 Fear | Cautious — defensive positioning |
| 45-55 Neutral | Normal conditions |
| 55-75 Greed | Optimism priced in — be selective |
| 75-100 Extreme Greed | Euphoria — expect choppier conditions |

---

## Output Format

```
## Market Briefing — [Today's Date]

### 🌡️ Market Mood
**Fear & Greed:** [X] — [Label, e.g. "Fear"]
**Investors:** [X]% bullish / [X]% bearish (AAII weekly survey)
**VIX:** [X]

[One plain-English sentence: what does this combination mean?
Example: "Investors are fearful and hedging — VIX above 20 means options are expensive."]

---

### 📊 Major Indexes
| Index | Price | Today | YTD |
|-------|-------|-------|-----|
| S&P 500 | $[X] | [+/-X%] [🟢/🟡/🔴] | [+/-X%] |
| Nasdaq | $[X] | [+/-X%] [🟢/🟡/🔴] | [+/-X%] |
| Dow | $[X] | [+/-X%] [🟢/🟡/🔴] | [+/-X%] |
| Russell 2000 | $[X] | [+/-X%] [🟢/🟡/🔴] | [+/-X%] |

[One sentence: what's the dominant theme? Tech selling off? Rotation happening?]

---

### 🔥 Biggest Movers
| Ticker | Change | Why |
|--------|--------|-----|
| [TICKER] | [+X%] | [Reason if available, otherwise leave blank] |
| [TICKER] | [-X%] | [Reason if available] |
*[Top 5 movers — mix of up and down]*

---

[ONLY include this section if a watchlist was provided]
### 👀 Your Watchlist
| Ticker | Price | Today | Analyst |
|--------|-------|-------|---------|
| [TICKER] | $[X] | [+/-X%] | [Buy/Hold/Sell] |

### 📰 Watchlist Headlines
- [TICKER]: [Headline]
- [TICKER]: [Headline]
- [TICKER]: [Headline]
[END WATCHLIST SECTION]

---

### ⚡ One Thing To Watch
[Single most important thing for today or the near term.
Could be: an earnings report coming out, a Fed speaker, a sector rotation underway,
a technical level being tested. Be specific. Not "markets are volatile."]

---
*Data from Bloom CLI. Source commands: bloom sentiment, bloom market, bloom info, bloom news.*
```

---

## Notes

- Keep it fast. The whole briefing should be readable in 2 minutes. Resist the urge to add more.
- "One Thing To Watch" is the most valuable section — don't make it generic. Find the actual thing.
- If VIX is above 25, always mention it — that's elevated fear territory.
- If the watchlist section is empty (no tickers provided), omit it entirely. Don't show an empty table.
- Don't describe what happened in detail for every mover. One line max per ticker.

---

## Related Skills

- `bloom-data` — Prerequisites and CLI reference (read this first)
- `research` — Deep dive on anything that caught your attention in the briefing
- `check` — Evaluate a trade idea sparked by today's movers
- `discover` — Find opportunities in the current market conditions
