---
name: stock-research-pipeline
description: >
  End-to-end equity research pipeline that downloads concalls and investor presentations from screener.in,
  uploads them to Google NotebookLM, dynamically generates analysis queries tailored to the specific company
  and sector, and produces a professional PDF equity deep-dive report. Use when user asks to "research [company]",
  "deep dive on [stock]", "analyze [symbol]", "equity report for [company]", "stock research pipeline for [symbol]",
  "concall to report for [company]", or any request for comprehensive stock analysis that should combine
  concall data with NotebookLM-powered research and a final PDF output. Also trigger when user says
  "run the pipeline on [symbol]", "full analysis of [stock]", or "/stock-research-pipeline [SYMBOL]".
---

# Stock Research Pipeline

Automate end-to-end equity research: screener.in data collection, NotebookLM-powered analysis with
dynamically generated queries, and a professional PDF report with variant perception scorecard.

## Prerequisites

- `nlm` CLI installed and authenticated (`nlm login --check` should succeed)
- Python venv at `.venv/` with `reportlab` installed
- Internet access for screener.in and BSE PDF downloads

If `nlm login --check` fails, tell the user to run `nlm login` first and stop.

## Input

The user provides a stock **SYMBOL** (e.g., `GRAVITA`, `TCS`, `RELIANCE`). Optionally they can specify:
- Number of quarters (default: 4)
- Specific quarters to analyze

## Workflow

### Step 1: Verify NLM Authentication

```bash
nlm login --check
```

If this fails, stop and ask the user to authenticate first.

### Step 2: Fetch Concall & Presentation Links from Screener

Use WebFetch on the screener.in consolidated page (fall back to standalone if needed):

```
URL: https://www.screener.in/company/{SYMBOL}/consolidated/
Prompt: Extract ALL conference call transcript links AND investor presentation links.
        For each, return the date (month and year), type (Concall or Investor Presentation),
        and the full URL. Only include bseindia.com PDF links.
        Return as a markdown table: Date, Type, Link
```

If consolidated returns no results, retry with:
```
https://www.screener.in/company/{SYMBOL}/
```

Sort results by date (newest first). Select the most recent N quarters (default 4).

### Step 3: Auto-Download PDFs

Create directories and download without asking:

```bash
mkdir -p data/companies/{SYMBOL}/concalls data/companies/{SYMBOL}/presentations
```

Download concalls:
```bash
curl -sL -o "data/companies/{SYMBOL}/concalls/{SYMBOL}_concall_{YYYY-MM}.pdf" "{URL}"
```

Download investor presentations:
```bash
curl -sL -o "data/companies/{SYMBOL}/presentations/{SYMBOL}_investor_pres_{YYYY-MM}.pdf" "{URL}"
```

Verify downloads with `ls -lh`. If any file is 0 bytes or suspiciously small (<10KB), warn and retry.

### Step 4: Create NotebookLM Notebook & Upload Sources

```bash
nlm notebook create "{COMPANY_NAME} - Equity Analysis"
```

Save the notebook ID. Then upload each PDF:

```bash
nlm source add {NOTEBOOK_ID} --file "path/to/file.pdf"
```

Upload ALL downloaded files (both concalls and presentations). Wait for each to complete before
starting the next.

### Step 5: Fetch Screener Financial Data & Upload to NLM

This step extracts comprehensive financial data from screener.in, converts it to a PDF, and
uploads it as an additional NLM source so that all queries can reference actual financial numbers.

**Step 5a: Extract screener financial data**

Use WebFetch on the screener page to extract ALL financial data:
```
URL: https://www.screener.in/company/{SYMBOL}/consolidated/
Prompt: Extract ALL financial data from this page in a structured format. Include:
        1. Company overview and description
        2. All key ratios (PE, PB, ROE, ROCE, Dividend Yield, etc.)
        3. Quarterly results table (all quarters visible) - Revenue, Expenses, Operating Profit, OPM%, Net Profit, EPS
        4. Profit & Loss annual data (all years visible)
        5. Balance Sheet data (all years visible)
        6. Cash Flow data (all years visible)
        7. Key financial ratios table (all years visible)
        8. Shareholding pattern (promoter, FII, DII, public)
        9. Peer comparison table if available
        10. Pros and Cons listed on the page
        Return everything as clean markdown with tables preserved.
```

If consolidated returns 404, fall back to standalone URL.

**Step 5b: Save as markdown**

Save the extracted data to `data/companies/{SYMBOL}/screener_data.md` for reference.

**Step 5c: Convert to PDF and upload to NLM**

NLM only accepts PDF sources, so convert the screener data to PDF using reportlab:
- Create a clean PDF with all financial tables (quarterly results, annual P&L, balance sheet,
  cash flow, ratios, shareholding pattern)
- Include key metrics, pros/cons, and growth rates
- Save to `data/companies/{SYMBOL}/{SYMBOL}_screener_data.pdf`
- Upload to the NLM notebook:
```bash
nlm source add {NOTEBOOK_ID} --file "data/companies/{SYMBOL}/{SYMBOL}_screener_data.pdf"
```

This ensures NLM queries can cross-reference concall commentary with actual reported numbers,
enabling much richer analysis (e.g., validating management guidance against actual delivery,
tracking margin trends, balance sheet changes, shareholding shifts).

### Step 6: Build Company Context (Critical Step)

This step is what makes queries dynamic and tailored. Gather context from TWO sources:

**Source A: Screener page context** (already extracted in Step 5a — reuse it, don't re-fetch)

**Source B: NLM notebook describe**

```bash
nlm notebook describe {NOTEBOOK_ID}
```

This returns an AI-generated summary of the uploaded sources, including suggested topics.

### Step 7: Generate Tailored Analysis Queries

Using the company context from Step 6, generate 6 analysis queries. These are NOT hardcoded
templates - they must be tailored to the specific company's business model, sector, and what's
actually discussed in the sources.

The 6 query categories are:

1. **Business Model & Value Chain** - How does THIS specific company make money? What are the
   unit economics? Adapt to the actual business (e.g., processing spread for a recycler,
   subscription metrics for SaaS, same-store growth for retail, NIM for a bank).

2. **Industry Structure & Competitive Positioning** - What's the industry structure for THIS
   sector? Who are the real competitors? What moats exist? Adapt terminology to the sector
   (e.g., "organized vs unorganized" for Indian manufacturing, "TAM penetration" for tech,
   "branch network" for banking).

3. **Management Quality & Execution** - Track guidance vs delivery across the quarters available.
   How has tone changed? What strategic bets are being made? Are they delivering on promises?

4. **Financial Deep Dive** - Quarter-by-quarter P&L, margins, balance sheet, return ratios.
   Focus on the metrics that matter most for THIS type of business (e.g., EBITDA/ton for
   commodity processors, ARPU for telecom, book value for banks, SSG for retail).

5. **Growth Triggers & Variant Perception** - SOIC-style analysis. Scan for all standard growth
   drivers (new products, geographic expansion, market share gains, capex, regulatory tailwinds,
   etc.). Identify where market consensus may be wrong. Build a VP scorecard with kill-switches.

6. **Risks & Bull/Base/Bear Scenarios** - What could go wrong? Build three FY+2/3 scenarios
   using management's own guidance as the base case, then stress-test up and down with specific
   numbers.

**Query generation guidelines:**
- Each query should be 150-300 words, structured with numbered sub-questions
- Always ask for specific data points, numbers, and direct management quotes
- Reference sector-specific metrics and terminology
- Ask for quarter-by-quarter tracking where relevant
- Reference screener financial data explicitly (e.g., "Using the screener financial data, trace
  the revenue journey from X to Y" or "Cross-reference the quarterly P&L from screener with
  management commentary") — this ensures NLM uses the uploaded screener source
- End each query requesting structured data presentation

### Step 8: Query NotebookLM

Run each query sequentially using `nlm notebook query`:

```bash
nlm notebook query "{NOTEBOOK_ID}" "{QUERY}" -t 180
```

For the first query, capture the `conversation_id` from the JSON response. Use it for subsequent
queries with `-c {CONVERSATION_ID}` to maintain context continuity.

**Important:** Parse the JSON output from each query and extract the `answer` field. Store all 6
answers for the report generation step.

If a query fails or times out, retry once. If it fails again, note the gap and continue with
remaining queries.

### Step 9: Generate PDF Report

Use the reportlab-based PDF generator. The report generation script should be written dynamically
based on the actual NLM query responses, but follow this structure:

**Report Sections:**
1. Cover Page (company name, symbol, date, data sources)
2. Table of Contents
3. Key Metrics at a Glance (KPI boxes from financial data)
4. Business Model & Value Chain (from Query 1)
5. Industry Structure & Competitive Positioning (from Query 2)
6. Financial Deep Dive with tables (from Query 4)
7. Management Quality & Execution Tracking (from Query 3)
8. Growth Drivers Checklist (from Query 5)
9. Variant Perception Scorecard (from Query 5)
10. Forward-Looking Growth Triggers (from Query 5)
11. Bull / Base / Bear Scenarios (from Query 6)
12. Key Risks (from Query 6)
13. Investment Thesis Summary (synthesized from all queries)

**PDF Design Guidelines:**
- Use A4 page size with professional styling
- Color palette: Navy (#1B2A4A) primary, Blue (#3B82F6) accent, clean whites and grays
- Custom flowables: SectionHeader (navy bar), SubSectionHeader (light blue bar), InfoBox (callout
  boxes with left accent), AccentBar
- Professional header/footer on every page (company name left, report type + date right)
- KPI metric boxes for key numbers
- Styled data tables with alternating row colors
- Management quotes in italic gray
- Color-coded risk boxes (red for high severity, amber for medium, blue for informational)
- Proper page breaks between major sections

Reference the PDF generation script at `scripts/generators/equity_report_pdf.py` if it exists.
Otherwise, write a new one following the reportlab patterns in the reference below and save it to
`data/companies/{SYMBOL}/analysis/generate_report.py`.

**Output location:**
```
data/companies/{SYMBOL}/analysis/{SYMBOL}_Equity_Analysis_{YYYY-MM-DD}.pdf
```

### Step 10: Open and Confirm

```bash
open "data/companies/{SYMBOL}/analysis/{SYMBOL}_Equity_Analysis_{YYYY-MM-DD}.pdf"
```

Tell the user the report is ready and summarize what's included (number of sections, key findings).

## Reference: PDF Generation Pattern

Use `reportlab.platypus` with `SimpleDocTemplate`. Key imports and patterns:

```python
from reportlab.lib.pagesizes import A4
from reportlab.lib.units import inch
from reportlab.lib.colors import HexColor
from reportlab.lib.styles import ParagraphStyle, getSampleStyleSheet
from reportlab.platypus import (
    SimpleDocTemplate, Paragraph, Spacer, Table, TableStyle,
    PageBreak, HRFlowable
)
from reportlab.platypus.flowables import Flowable

# Custom flowables for section headers, info boxes, etc.
# Use onFirstPage for cover, onLaterPages for header/footer
# Build story list and call doc.build(story, onFirstPage=..., onLaterPages=...)
```

See `data/companies/GRAVITA/analysis/generate_report.py` for a complete working example of the
PDF generation pattern including custom flowables, color palette, KPI boxes, styled tables,
quote formatting, and scenario boxes.

## Reference: NLM CLI Commands

```bash
nlm login --check                              # Verify auth
nlm notebook create "Name"                     # Create notebook, returns ID
nlm notebook describe {ID}                     # AI summary of sources
nlm source add {ID} --file "path.pdf"          # Upload PDF source
nlm notebook query {ID} "question" -t 180      # Query (180s timeout)
nlm notebook query {ID} "q" -c {CONV_ID} -t 180  # Follow-up query
```

## Error Handling

- **NLM auth expired:** Tell user to run `nlm login` and retry
- **Screener page not found:** Try both consolidated and standalone URLs. If both fail, ask user
  to verify the symbol
- **PDF download fails:** Retry once. If still fails, skip that file and note it
- **NLM query timeout:** Retry with `-t 240`. If still fails, skip and note the gap in the report
- **No concall links found:** Fall back to web search for BSE/NSE filing links

## Example Usage

```
User: "Run the stock research pipeline on GRAVITA"
User: "Deep dive on TCS"
User: "Equity report for RELIANCE"
User: "/stock-research-pipeline HDFCBANK"
```
