---
name: research
description: >
  Full stock analysis for any publicly traded company. Covers financials, valuation,
  growth, analyst consensus, catalysts, risks, technicals, and recent news.
  Triggers: "tell me about AAPL", "analyze NVDA", "research Tesla", "what do you think of MSFT",
  "is GOOGL a good buy", "break down META", "deep dive on AMD", "what's the deal with TSLA"
---

# research — Full Stock Analysis

> Every number comes from Bloom's financial data. Not a language model's training data.

**Trigger:** `/research <TICKER>` — or any time a user asks about a specific stock.

---

## Read `bloom-data` Skill First

Before running anything, check that `bloom` is installed and authenticated. See the `bloom-data` skill for prerequisites and output rules.

```bash
mkdir -p /tmp/bloom
```

---

## Steps (Run in Order)

### 1. Company overview
```bash
bloom info <TICKER> -o /tmp/bloom/research-info.json
```

### 2. Revenue history
```bash
bloom financials <TICKER> --type revenue -o /tmp/bloom/research-revenue.json
```

### 3. Operating margins
```bash
bloom financials <TICKER> --type operating_margin -o /tmp/bloom/research-margins.json
```

### 4. Free cash flow
```bash
bloom financials <TICKER> --type free_cash_flow -o /tmp/bloom/research-fcf.json
```

### 5. Earnings history + next date
```bash
bloom earnings <TICKER> -o /tmp/bloom/research-earnings.json
```

### 6. Upcoming catalysts
```bash
bloom catalysts <TICKER> -o /tmp/bloom/research-catalysts.json
```

### 7. Technical picture
```bash
bloom technicals <TICKER> -o /tmp/bloom/research-technicals.json
```

### 8. Recent news
```bash
bloom news <TICKER> --limit 5 -o /tmp/bloom/research-news.json
```

### 9. Extract key fields
```bash
# Price and valuation
cat /tmp/bloom/research-info.json | jq '{price, market_cap, pe_ratio, sector, analyst_consensus, price_target, bottom_line}'

# Revenue trend (last 4 periods)
cat /tmp/bloom/research-revenue.json | jq '.[-4:]'

# Margins trend
cat /tmp/bloom/research-margins.json | jq '.[-4:]'

# FCF
cat /tmp/bloom/research-fcf.json | jq '.[-4:]'

# Earnings
cat /tmp/bloom/research-earnings.json | jq '{next_earnings_date, recent_results}'

# Top catalysts
cat /tmp/bloom/research-catalysts.json | jq '.catalysts[:3]'

# Technicals
cat /tmp/bloom/research-technicals.json | jq '{trend, momentum, support, resistance, rsi}'

# News headlines
cat /tmp/bloom/research-news.json | jq '.[].headline'
```

---

## Verdict Rubric

Use this to assign 🟢/🟡/🔴 to each metric:

| Metric | 🟢 Good | 🟡 OK | 🔴 Concern |
|--------|---------|-------|------------|
| Revenue Growth (YoY) | >15% | 5-15% | <5% or negative |
| Operating Margin | >20% | 10-20% | <10% |
| Free Cash Flow | Positive + growing | Positive, flat | Negative |
| P/E vs sector | Below avg | At avg | >50% above avg |
| Analyst consensus | Majority Buy | Mixed | Majority Sell |

---

## Output Format

Use exactly this structure:

```
## [Company Name] ($TICKER) — $[Price]
**Sector:** [X] | **Market Cap:** $[X] | **P/E:** [X]

### What It Does
[1-2 plain English sentences. What does this company actually sell or do?]

### By The Numbers
| Metric | Value | Signal |
|--------|-------|--------|
| Revenue Growth (YoY) | X% | 🟢/🟡/🔴 |
| Operating Margin | X% | 🟢/🟡/🔴 |
| Free Cash Flow | $X | 🟢/🟡/🔴 |
| P/E Ratio | X | 🟢/🟡/🔴 |
| EPS Trend | $X → $X | 🟢/🟡/🔴 |

### What Analysts Say
[X] Buy / [X] Hold / [X] Sell — Avg target: $[X] ([X]% upside/downside from current price)
[One sentence: what does the consensus suggest?]

### Catalysts (Potential Upside)
1. [Top catalyst from bloom catalysts output]
2. [Second catalyst]
3. [Third catalyst]

### Risks (What Could Go Wrong)
1. [Biggest risk — from data, not guessing]
2. [Second risk]
3. [Third risk]

### Technical Picture
**Trend:** [Bullish/Bearish/Neutral]
**Momentum:** [RSI: X — overbought/oversold/neutral]
**Key levels:** Support $[X] | Resistance $[X]
[One sentence on what the technicals suggest about timing]

### Recent News
- [Headline 1]
- [Headline 2]
- [Headline 3]

### Bottom Line
[2-3 sentences. Is this worth owning? At what kind of price? What's the biggest thing to watch?
Be direct. Don't hedge everything. Take a stance based on the data.]

---
*Data from Bloom CLI. Source commands: bloom info, bloom financials, bloom earnings, bloom catalysts, bloom technicals, bloom news.*
```

---

## Notes

- If a ticker has no analyst coverage, say so rather than leaving the section blank.
- If FCF is negative, always flag it — even for growth companies.
- "Bottom Line" is the most-read section. Make it worth reading.
- Don't use the word "robust" or "pivotal." Say what's actually happening.

---

## Related Skills

- `bloom-data` — Prerequisites and CLI reference (read this first)
- `check` — Evaluate a specific trade thesis on this stock
- `briefing` — Broader market context for the day
- `discover` — Find similar opportunities if this one doesn't pan out
