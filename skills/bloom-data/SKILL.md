---
name: bloom-data
description: >
  Foundation skill for all Bloom investing skills. Load this first before running any
  other Bloom skill. Handles CLI setup, authentication checks, output conventions,
  and documents every available bloom command with its flags.
  Triggers: "set up bloom", "bloom CLI help", "check bloom install", "bloom commands"
---

# bloom-data — Foundation

> Every number comes from Bloom's financial data. Not a language model's training data.

This skill is the shared foundation. Every other Bloom skill reads this first. It defines the rules, checks prerequisites, and lists every available CLI command.

---

## Step 0: Prerequisites Check

Before running any bloom command, verify:

### 1. CLI is installed
```bash
bloom --help
# or, if not globally installed:
npx @bloomai/cli --help
```

If that fails:
```bash
npm install -g @bloomai/cli
```

### 2. Authentication is set up

Option A — API key (preferred for automation):
```bash
export BLOOM_API_KEY=your_key_here
```

Option B — Interactive login:
```bash
bloom auth login
```

### 3. Test it works
```bash
bloom info AAPL
```

You should see Apple's stock data. If you see an auth error, run `bloom auth login`.

---

## Output Rules

These rules apply to every skill that builds on bloom-data:

1. **Always save to file.** Use `-o /tmp/bloom/<descriptive-name>.json` for every command. This keeps the raw JSON out of your context window.

   ```bash
   mkdir -p /tmp/bloom
   bloom info AAPL -o /tmp/bloom/research-info.json
   ```

2. **Use `jq` to extract only what you need.** Don't load entire JSON files into context. Pull the specific fields:

   ```bash
   cat /tmp/bloom/research-info.json | jq '.price, .pe_ratio, .market_cap'
   ```

3. **Never estimate or hallucinate financial data.** If the CLI returns no data, say so. Don't fill in numbers from training data.

4. **Cite source commands.** When presenting a key number, note where it came from:
   - "P/E 33.2 (via `bloom info AAPL`)"
   - "Revenue grew 12% YoY (via `bloom financials AAPL --type revenue`)"

5. **Create the output directory first:**
   ```bash
   mkdir -p /tmp/bloom
   ```

---

## Available Commands

Full reference for every `bloom` CLI command.

### `bloom info <TICKER> [TICKER2 ...]`
Stock overview: price, market cap, P/E, analyst ratings, sector, summary.

```bash
bloom info AAPL
bloom info AAPL MSFT GOOGL          # batch up to 10 tickers
bloom info NVDA -o /tmp/bloom/nvda-info.json
```

Key output fields: `price`, `market_cap`, `pe_ratio`, `sector`, `analyst_consensus`, `price_target`, `bottom_line`

---

### `bloom financials <TICKER> --type <TYPE>`
Historical financial metrics. Always specify `--type`.

```bash
bloom financials AAPL --type revenue
bloom financials AAPL --type operating_margin
bloom financials AAPL --type free_cash_flow
bloom financials AAPL --type net_income
bloom financials AAPL --type eps
bloom financials AAPL --type revenue -o /tmp/bloom/aapl-revenue.json
```

Available types: `revenue`, `operating_margin`, `free_cash_flow`, `net_income`, `eps`

---

### `bloom earnings <TICKER>`
Recent earnings history + next earnings date.

```bash
bloom earnings AAPL
bloom earnings AAPL -o /tmp/bloom/aapl-earnings.json
```

Key output fields: `next_earnings_date`, `recent_results`, `beat_miss_history`

---

### `bloom catalysts <TICKER>`
Upcoming events and catalysts that could move the stock.

```bash
bloom catalysts AAPL
bloom catalysts NVDA -o /tmp/bloom/nvda-catalysts.json
```

Key output fields: `catalysts` (list with `title`, `description`, `impact`)

---

### `bloom technicals <TICKER>`
Technical analysis: trend, momentum, support/resistance levels.

```bash
bloom technicals AAPL
bloom technicals TSLA -o /tmp/bloom/tsla-technicals.json
```

Key output fields: `trend`, `momentum`, `support`, `resistance`, `moving_averages`, `rsi`

---

### `bloom news <TICKER> [--limit N]`
Recent news headlines for a ticker.

```bash
bloom news AAPL
bloom news AAPL --limit 5
bloom news AAPL MSFT --limit 3 -o /tmp/bloom/news.json
```

Default limit: 10. Recommended: `--limit 5` for research, `--limit 3` for briefings.

---

### `bloom sentiment`
Market-wide sentiment: Fear & Greed index, AAII bull/bear survey, VIX.

```bash
bloom sentiment
bloom sentiment -o /tmp/bloom/sentiment.json
```

Key output fields: `fear_greed_index`, `fear_greed_label`, `aaii_bull`, `aaii_bear`, `vix`

---

### `bloom market --type <TYPE> [--limit N]`
Market overview by category.

```bash
bloom market --type major_indexes
bloom market --type top_movers --limit 10
bloom market --type sector_performance
bloom market --type major_indexes -o /tmp/bloom/indexes.json
```

Available types: `major_indexes`, `top_movers`, `sector_performance`

---

### `bloom portfolio '<json>'`
Analyze a portfolio. Input is a JSON object with ticker:weight pairs.

```bash
bloom portfolio '{"AAPL": 0.4, "MSFT": 0.3, "GOOGL": 0.3}'
bloom portfolio '{"NVDA": 0.5, "TSLA": 0.25, "AMD": 0.25}' -o /tmp/bloom/portfolio.json
```

Weights should sum to 1.0. Key output fields: `grade`, `concentration_risk`, `sector_breakdown`, `correlation`, `suggestions`

---

### `bloom screen "<filter>" ["<filter2>" ...]`
Screen for stocks matching criteria.

```bash
bloom screen "market_cap > 10B" "pe_ratio < 20" "revenue_growth > 10"
bloom screen "dividend_yield > 3" "payout_ratio < 60"
bloom screen "market_cap > 10B" "pe_ratio < 20" -o /tmp/bloom/screener.json
```

Common filters: `market_cap > XB`, `pe_ratio < X`, `revenue_growth > X`, `dividend_yield > X`, `payout_ratio < X`

---

### `bloom collections [--section <SECTION>]`
Curated stock lists by theme.

```bash
bloom collections
bloom collections --section VALUE
bloom collections --section GROWTH
bloom collections --section DIVIDEND
bloom collections -o /tmp/bloom/collections.json
```

Available sections: `VALUE`, `GROWTH`, `DIVIDEND`, `MOMENTUM`, `DEFENSIVE`

---

### `bloom ai-trades [--limit N]`
Trades made by AI models in the AI Arena (real money, live portfolios).

```bash
bloom ai-trades
bloom ai-trades --limit 10 -o /tmp/bloom/ai-trades.json
```

Key output fields: `trades` (list with `model`, `ticker`, `direction`, `return_pct`, `date`)

---

### `bloom ai-portfolio`
Current holdings of each AI model in the Arena.

```bash
bloom ai-portfolio
bloom ai-portfolio -o /tmp/bloom/ai-portfolio.json
```

Key output fields: `portfolios` (keyed by model name, with `holdings`, `total_return`, `cash_pct`)

---

### `bloom trades "<thesis>"`
Evaluate a trade thesis. Pass a natural language description.

```bash
bloom trades "buy NVDA before next earnings because AI spending is accelerating"
bloom trades "short TSLA because valuation is stretched" -o /tmp/bloom/check-eval.json
```

Key output fields: `verdict`, `bull_case`, `bear_case`, `risk_reward`, `supporting_data`

---

### `bloom position-size --bull <est> --bear <est> --conviction <score> [--value <portfolio-value>]`
Calculate appropriate position size based on risk/reward.

```bash
bloom position-size --bull 20 --bear 10 --conviction 7
bloom position-size --bull 30 --bear 15 --conviction 8 --value 100000 -o /tmp/bloom/sizing.json
```

- `--bull`: estimated upside % if thesis is right
- `--bear`: estimated downside % if thesis is wrong
- `--conviction`: score 1-10
- `--value`: optional portfolio value in dollars

Key output fields: `position_pct`, `max_loss_pct`, `risk_reward_ratio`, `dollar_amount` (if value provided)

---

## Error Handling

| Error | Fix |
|-------|-----|
| `command not found: bloom` | Run `npm install -g @bloomai/cli` |
| `Authentication required` | Run `bloom auth login` or set `BLOOM_API_KEY` |
| `Rate limit exceeded` | Wait 60 seconds, then retry |
| `No data found for <TICKER>` | Check ticker spelling; some OTC stocks aren't covered |
| Empty JSON output | Check auth, retry once, then report no data available |

---

## Related Skills

- `research` — Full stock analysis using bloom commands
- `review` — Portfolio grading and suggestions
- `discover` — Finding new opportunities
- `check` — Evaluating a specific trade
- `briefing` — Daily market update
