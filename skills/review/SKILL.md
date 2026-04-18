---
name: review
description: >
  Portfolio review and grading. Takes a user's holdings (tickers + weights or dollar amounts),
  runs analysis on the whole portfolio, and returns a letter grade A-F with specific problems
  and actionable suggestions.
  Triggers: "review my portfolio", "how's my portfolio doing", "grade my holdings",
  "what do you think of my stocks", "check my portfolio", "is my portfolio good",
  "portfolio review", "I own AAPL MSFT TSLA what do you think"
---

# review — Portfolio Review

> Every number comes from Bloom's financial data. Not a language model's training data.

**Trigger:** `/review <HOLDINGS>` — when a user shares their portfolio holdings.

Holdings can be:
- Tickers + percentages: `AAPL 40%, MSFT 30%, GOOGL 30%`
- Tickers + dollar amounts: `AAPL $10000, MSFT $7500, TSLA $2500`
- Just tickers (assume equal weight): `AAPL MSFT GOOGL NVDA`

---

## Read `bloom-data` Skill First

Before running anything, check that `bloom` is installed and authenticated. See the `bloom-data` skill for prerequisites and output rules.

```bash
mkdir -p /tmp/bloom
```

---

## Steps (Run in Order)

### 1. Parse holdings into JSON

Convert whatever format the user gave into a JSON object with ticker:weight pairs summing to 1.0.

Examples:
- `AAPL 40%, MSFT 30%, GOOGL 30%` → `{"AAPL": 0.4, "MSFT": 0.3, "GOOGL": 0.3}`
- `AAPL $10k, MSFT $5k, TSLA $5k` → `{"AAPL": 0.5, "MSFT": 0.25, "TSLA": 0.25}`
- `AAPL MSFT GOOGL NVDA` → `{"AAPL": 0.25, "MSFT": 0.25, "GOOGL": 0.25, "NVDA": 0.25}`

### 2. Portfolio-level analysis
```bash
bloom portfolio '{"AAPL": 0.4, "MSFT": 0.3, "GOOGL": 0.3}' -o /tmp/bloom/review-portfolio.json
```

### 3. Stock info (batch — up to 10 tickers at once)
```bash
bloom info AAPL MSFT GOOGL -o /tmp/bloom/review-info.json
```

### 4. Revenue financials per ticker
```bash
bloom financials AAPL --type revenue -o /tmp/bloom/review-fin-AAPL.json
bloom financials MSFT --type revenue -o /tmp/bloom/review-fin-MSFT.json
# repeat for each holding
```

### 5. Extract key fields
```bash
# Portfolio summary
cat /tmp/bloom/review-portfolio.json | jq '{grade, concentration_risk, sector_breakdown, correlation, suggestions}'

# Info for each ticker
cat /tmp/bloom/review-info.json | jq '.[] | {ticker: .symbol, pe: .pe_ratio, sector: .sector, analyst: .analyst_consensus, market_cap: .market_cap}'
```

---

## Grading Rubric

Grade the portfolio A through F across 5 categories. Average them for the overall grade.

### Diversification
| Grade | Criteria |
|-------|----------|
| A | 8+ holdings, 3+ sectors, some international exposure |
| B | 5-7 holdings, 2-3 sectors |
| C | 4-6 holdings, 1-2 sectors |
| D | 3-4 holdings, mostly 1 sector |
| F | 1-2 holdings or 90%+ in one sector |

### Valuation
| Grade | Criteria |
|-------|----------|
| A | Weighted avg P/E below market (under 20) |
| B | P/E near market average (20-25) |
| C | Slightly elevated (25-35) |
| D | Expensive (35-50) |
| F | Extreme premium (50+ avg P/E) |

### Growth
| Grade | Criteria |
|-------|----------|
| A | Avg revenue growth >20% YoY across holdings |
| B | 12-20% avg growth |
| C | 5-12% avg growth |
| D | Under 5% avg growth |
| F | Declining revenue across majority of holdings |

### Income
| Grade | Criteria |
|-------|----------|
| A | >3% weighted avg dividend yield with sustainable payout |
| B | 1.5-3% yield |
| C | <1.5% yield with dividend payers |
| D | No dividend income |
| F | No income + high payout ratios among non-payers |

### Risk
| Grade | Criteria |
|-------|----------|
| A | Low correlation between holdings, defensives present |
| B | Some correlation, mixed sectors |
| C | Moderate correlation, similar sectors |
| D | High correlation, same sector/style |
| F | Extremely concentrated, highly correlated positions |

---

## Output Format

```
## Portfolio Review — Overall Grade: [A/B/C/D/F]

### Top 3 Problems
1. 🔴 [Biggest issue — be specific. "65% in tech" not "concentration risk"]
2. 🟡 [Second issue]
3. 🟡 [Third issue — can be 🟢 if portfolio is solid]

### Holdings Breakdown
| Ticker | Weight | P/E | Revenue Growth | Analyst | Flag |
|--------|--------|-----|----------------|---------|------|
| [TICKER] | [X]% | [X] | [+X%] | [Buy/Hold/Sell] | [⚠️ if issue] |

### Sector Exposure
| Sector | Weight |
|--------|--------|
| Technology | X% |
| Healthcare | X% |
| [etc.] | |

[One sentence: is sector concentration a problem here?]

### Risk Assessment
**Concentration:** [Single stock or sector risk]
**Correlation:** [How much do these move together?]
**Drawdown risk:** [What happens if tech sells off 30%? If rates rise?]

### Suggestions
1. [Specific and actionable — "Add VTI or an international ETF to reduce single-stock risk"]
2. [Second suggestion]
3. [Third suggestion]

*Keep suggestions realistic. Don't tell someone to add 15 new positions.*

### Grade Breakdown
| Category | Grade | Why |
|----------|-------|-----|
| Diversification | [X] | [One-line reason] |
| Valuation | [X] | [One-line reason] |
| Growth | [X] | [One-line reason] |
| Income | [X] | [One-line reason] |
| Risk | [X] | [One-line reason] |

---
*Data from Bloom CLI. Source commands: bloom portfolio, bloom info, bloom financials.*
```

---

## Notes

- Be honest about problems. A portfolio of 3 high-P/E tech stocks is a C at best — say so.
- Suggestions should be specific, not "consider diversifying." Say what to buy or trim.
- If a user has fewer than 5 holdings, the diversification grade is almost always C or below.
- Don't penalize concentrated portfolios held by people who clearly know what they're doing (e.g., single-stock employee RSUs situation) — acknowledge the context if they share it.

---

## Related Skills

- `bloom-data` — Prerequisites and CLI reference (read this first)
- `research` — Deep dive on any holding that raised concerns
- `check` — Evaluate a potential new position to add
- `discover` — Find diversification candidates
