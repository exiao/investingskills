---
name: check
description: >
  Evaluate a specific trade idea. Takes a natural language thesis ("I want to buy NVDA before earnings"),
  runs data against it, and returns a verdict with bull/bear case, key numbers, and position sizing.
  Triggers: "should I buy AAPL", "is this trade dumb", "check this trade", "I'm thinking of buying TSLA",
  "evaluate this trade", "is NVDA a good buy right now", "I want to short META", "analyze my trade idea",
  "what do you think of buying AMD", "is it too late to buy MSFT"
---

# check — Evaluate a Trade

> Every number comes from Bloom's financial data. Not a language model's training data.

**Trigger:** `/check <TRADE_THESIS>` — when a user describes a trade they're considering.

The thesis can be:
- A simple buy: "buy AAPL"
- A directional idea: "short TSLA because valuation is stretched"
- A thesis with context: "buy NVDA before next earnings because AI spending is accelerating"
- A question: "should I buy META right now?"

---

## Read `bloom-data` Skill First

Before running anything, check that `bloom` is installed and authenticated. See the `bloom-data` skill for prerequisites and output rules.

```bash
mkdir -p /tmp/bloom
```

---

## Step 0: Parse the Trade

Before running any commands, extract:
1. **Ticker** — the stock symbol (e.g., NVDA)
2. **Direction** — Buy (long) or Sell/Short?
3. **Thesis** — the reason or context (if any)
4. **Time horizon** — short-term trade vs. longer hold? (infer if not stated)

If no ticker is clear, ask the user to clarify before proceeding.

---

## Steps (Run in Order)

### 1. Company overview
```bash
bloom info <TICKER> -o /tmp/bloom/check-info.json
```

### 2. AI thesis evaluation
```bash
bloom trades "<full thesis from user>" -o /tmp/bloom/check-eval.json
```

Pass the user's actual words. The more specific, the better.

### 3. Technical picture
```bash
bloom technicals <TICKER> -o /tmp/bloom/check-technicals.json
```

### 4. Revenue trend
```bash
bloom financials <TICKER> --type revenue -o /tmp/bloom/check-revenue.json
```

### 5. Earnings history + next date
```bash
bloom earnings <TICKER> -o /tmp/bloom/check-earnings.json
```

### 6. Upcoming catalysts
```bash
bloom catalysts <TICKER> -o /tmp/bloom/check-catalysts.json
```

### 7. Position sizing
```bash
bloom position-size --bull <upside_est> --bear <downside_est> --conviction <score> -o /tmp/bloom/check-sizing.json
```

Estimate `--bull` and `--bear` from the data:
- Bull: analyst price target upside % (from bloom info)
- Bear: estimated downside if thesis is wrong (use recent pullback or support level distance)
- Conviction: 1-10 based on quality of data supporting the thesis (not gut feel — data quality)

### 8. Extract key fields
```bash
# Core info
cat /tmp/bloom/check-info.json | jq '{price, pe_ratio, market_cap, analyst_consensus, price_target, sector}'

# Thesis evaluation
cat /tmp/bloom/check-eval.json | jq '{verdict, bull_case, bear_case, risk_reward, supporting_data}'

# Technicals
cat /tmp/bloom/check-technicals.json | jq '{trend, momentum, rsi, support, resistance}'

# Revenue trend
cat /tmp/bloom/check-revenue.json | jq '.[-4:]'

# Next earnings
cat /tmp/bloom/check-earnings.json | jq '{next_earnings_date, last_result}'

# Top catalysts
cat /tmp/bloom/check-catalysts.json | jq '.catalysts[:3]'

# Sizing
cat /tmp/bloom/check-sizing.json | jq '{position_pct, risk_reward_ratio, max_loss_pct}'
```

---

## Verdict Framework

| Verdict | When to use |
|---------|-------------|
| ✅ Looks Good | Data supports the thesis, risk is manageable, technicals aren't screaming "don't" |
| ⚠️ Proceed With Caution | Thesis has merit but there are real concerns — maybe size down, wait for a trigger |
| 🛑 Rethink This | Data contradicts the thesis, or the risk/reward is poor |

Be honest. Most trade ideas are somewhere in the middle. ⚠️ is the most common answer.

---

## Output Format

```
## Trade Check: [Buy/Short] $[TICKER] at $[Price]

### Verdict: [✅ Looks Good / ⚠️ Proceed With Caution / 🛑 Rethink This]

### Your Thesis
"[User's original thesis, quoted exactly]"

### What The Data Says

**For it:**
1. [Strongest data point supporting the trade]
2. [Second supporting point]
3. [Third supporting point]

**Against it:**
1. [Strongest counter-argument from data]
2. [Second concern]
3. [Third concern]

### Key Numbers
| Metric | Value | Signal |
|--------|-------|--------|
| Current Price | $[X] | |
| Analyst Target | $[X] ([X]% upside) | 🟢/🟡/🔴 |
| P/E vs Sector Avg | [X] vs [X] | 🟢/🟡/🔴 |
| Revenue Trend | [X]% YoY | 🟢/🟡/🔴 |
| Analyst Consensus | [X Buy / X Hold / X Sell] | 🟢/🟡/🔴 |
| Technical Trend | [Bullish/Bearish/Neutral] | 🟢/🟡/🔴 |

### Timing
**Next earnings:** [Date — or "no date set yet"]
[One sentence: does timing help or hurt this trade? Buying before earnings is a bet on the report.]

**Key levels:**
- Support: $[X] — [what happens if it breaks?]
- Resistance: $[X] — [what's the next target if it clears?]

**Upcoming catalysts:** [Top 1-2 from bloom catalysts]

### If You Do It
**Suggested size:** [X]% of portfolio
**Risk/reward:** [X]:1
**Where to stop out:** $[X] ([X]% below current price)
**What would change the thesis:** [One sentence — what data would make you exit?]

---
*Data from Bloom CLI. Source commands: bloom info, bloom trades, bloom technicals, bloom financials, bloom earnings, bloom catalysts, bloom position-size.*
```

---

## Notes

- Don't let a user talk you into a 🟢 when the data says 🔴. The point is honest evaluation.
- "Proceed With Caution" isn't a cop-out — be specific about what the caution is.
- If the user's thesis is vague ("AAPL seems cheap"), run the data and help them form a real thesis based on what you find.
- Position sizing is the most ignored part of any trade. Always include it.
- If earnings are within 2 weeks, always flag that. Earnings = binary event = higher risk.

---

## Related Skills

- `bloom-data` — Prerequisites and CLI reference (read this first)
- `research` — Full deep dive if you want more context before deciding
- `briefing` — Check market conditions before entering a trade
- `discover` — If this one doesn't check out, find something that does
