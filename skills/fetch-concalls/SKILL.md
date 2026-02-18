---
name: fetch-concalls
description: >
  Fetch and optionally download conference call (concall/earnings call) transcript PDFs for any
  Indian listed company. Use when user asks to "get concall links", "fetch concall transcripts",
  "find earnings call links", "download concall PDFs", "download concalls for [company]",
  or mentions getting conference call links for a company symbol.
  Accepts a BSE/NSE stock symbol as argument (e.g., YATHARTH, GRSE, TCS).
---

# Fetch & Download Concall Links

Fetch all available conference call transcript PDF links for an Indian listed company using a 3-tier fallback strategy, then optionally download them.

## Usage

Argument: company stock symbol (e.g., `YATHARTH`, `GRSE`, `TCS`)

## Workflow

Try each tier in order. Move to the next tier only if the previous one yields no concall links.

### Tier 1: Screener.in (best source — aggregated official BSE links)

1. WebFetch `https://www.screener.in/company/{SYMBOL}/consolidated/` with prompt:

   ```
   Extract ALL conference call / earnings call transcript links from this page.
   For each, return the date (month and year) and the full URL.
   Only include links that point to concall/earnings call transcripts (typically bseindia.com PDF links).
   Return as a markdown table with columns: Date, Link
   ```

2. If consolidated page returns no results or fails, retry with standalone:
   `https://www.screener.in/company/{SYMBOL}/`

3. If concall links found, present results and stop.

### Tier 2: Web Search for official sources only

If Tier 1 fails, use WebSearch with these queries:

1. `"{COMPANY_NAME}" conference call transcript site:bseindia.com`
2. `"{COMPANY_NAME}" concall transcript site:nseindia.com`
3. `"{COMPANY_NAME}" investor relations earnings call site:{company_website}`

Only include links from:
- `bseindia.com` (BSE filings)
- `nseindia.com` / `nsearchives.nseindia.com` (NSE filings)
- The company's own website (investor relations / compliance pages)

If official links found, present results and stop.

### Tier 3: Web Search for third-party sources (last resort)

If Tier 1 and Tier 2 both fail, use WebSearch:

1. `"{COMPANY_NAME}" earnings call transcript {CURRENT_YEAR}`
2. `"{SYMBOL}" concall transcript`

Accept links from third-party aggregators like:
- trendlyne.com
- marketscreener.com
- alphaspread.com
- gurufocus.com

Clearly label these as **third-party sources** in the output.

## Output Format

Present results as a markdown table:

| Quarter | Transcript | Source |
|---------|-----------|--------|
| Q3 FY26 (Feb 2026) | [PDF](https://www.bseindia.com/...) | BSE |
| Q2 FY26 (Nov 2025) | [PDF](https://www.bseindia.com/...) | BSE |

- Include a **Source** column only when mixing official and third-party links.
- If all links are from the same source (e.g., all BSE via screener), omit the Source column.
- If no concall links found across all 3 tiers, inform the user.

## Download Step

After presenting the links table, use AskUserQuestion to ask:

**"Do you want to download these concall PDFs?"** with options:
- **All** — download all transcripts
- **Latest only** — download only the most recent transcript
- **Select quarters** — let user specify which ones
- **No** — skip download

### Download process

1. Create target directory: `data/companies/{SYMBOL}/concalls/` (relative to project root)
2. Download each selected PDF using Bash `curl`:
   ```
   curl -sL -o "data/companies/{SYMBOL}/concalls/{SYMBOL}_concall_{YYYY-MM}.pdf" "{URL}"
   ```
3. Name files as `{SYMBOL}_concall_{YYYY-MM}.pdf` (e.g., `YATHARTH_concall_2026-02.pdf`)
4. After download completes, list the saved files with sizes to confirm.
