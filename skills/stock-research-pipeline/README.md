# Stock Research Pipeline

End-to-end equity research skill that automates the entire journey from raw data collection to a polished PDF report -- no manual steps in between.

## What It Does

1. **Fetches concall transcripts & investor presentations** from screener.in (BSE PDF links) for the last 4 quarters
2. **Extracts financial data** from screener.in -- quarterly results, annual P&L, balance sheet, cash flows, ratios, shareholding pattern
3. **Uploads everything to Google NotebookLM** as a dedicated research notebook
4. **Generates 6 tailored analysis queries** based on the company's actual business model and sector (not generic templates) covering:
   - Business model & value chain (unit economics, revenue drivers)
   - Industry structure & competitive positioning (moats, market share)
   - Management quality & execution (guidance vs delivery tracking)
   - Financial deep dive (quarter-by-quarter margins, return ratios, sector-specific KPIs)
   - Growth triggers & variant perception (SOIC-style, with kill-switches)
   - Risks & bull/base/bear scenarios (with specific numbers from management guidance)
5. **Queries NotebookLM** sequentially with conversation context so answers build on each other
6. **Generates a professional PDF report** with:
   - Cover page, table of contents
   - Key metrics at a glance (KPI boxes)
   - Detailed sections for each analysis area
   - Styled data tables, management quotes, color-coded risk boxes
   - Variant perception scorecard
   - Bull / Base / Bear scenario breakdown
   - Investment thesis summary

**Input:** Stock symbol (e.g. `GRAVITA`, `TCS`)
**Output:** `data/companies/{SYMBOL}/analysis/{SYMBOL}_Equity_Analysis_{date}.pdf`

## Setup

1. **Python environment**
   ```bash
   python -m venv .venv && source .venv/bin/activate
   pip install reportlab
   ```

2. **NLM CLI** -- install and authenticate
   ```bash
   pip install nlm-client
   nlm login
   nlm login --check   # must succeed before running the skill
   ```

3. **Internet access** -- required for screener.in and BSE PDF downloads.

## Usage

```
/stock-research-pipeline GRAVITA
```

Or natural language:
```
"Run the stock research pipeline on TCS"
"Deep dive on RELIANCE"
"Equity report for HDFCBANK"
```
