---
name: growth-trigger-analysis
description: >
  Buy-side equity analyst (SOIC style) that extracts variant perception factors and concrete
  investible triggers from company concall transcripts. Use when user asks to "analyze growth
  triggers for [company]", "create VP scorecard for [company]", "extract triggers from concalls",
  "variant perception analysis", or "growth trigger map for [SYMBOL]". Reads transcripts from
  data/companies/{SYMBOL}/concalls/ and produces evidence-backed VP scorecard with ranked
  triggers, top 3 deep dives, and forward-looking growth triggers list.
---

# Growth Trigger & Variant Perception Analysis

Produce an SOIC-style variant perception scorecard and growth triggers list from company concall transcripts.

## When to use

Invoke when user says:
- "growth triggers for SAILIFE"
- "VP analysis for HDFCBANK"
- "analyze concalls for TATAELXSI"
- "variant perception scorecard for [company]"
- any request for forward-looking trigger extraction from earnings calls

## Prerequisites

Concall transcripts should ideally exist at:
```
data/companies/{SYMBOL}/concalls/
```

If transcripts are missing, the skill fetches them automatically (see Step 1).

## Workflow

### Step 1: Identify symbol and verify data

```
1. Extract SYMBOL from user request
2. Glob for data/companies/{SYMBOL}/concalls/*_Transcript.md (or *.pdf)
3. If at least 2 transcripts exist (4 preferred), sort by date and pick 4 most recent
4. If none found → auto-fetch using the fallback strategy below
```

#### Auto-fetch concall transcripts

If local transcripts are missing:

1. **Screener.in**: WebFetch `https://www.screener.in/company/{SYMBOL}/consolidated/` — extract all concall PDF links (BSE links). If consolidated fails, try `https://www.screener.in/company/{SYMBOL}/`
2. **Web Search (official)**: If screener yields no links, WebSearch `"{COMPANY_NAME}" conference call transcript site:bseindia.com`
3. **Web Search (third-party)**: If still nothing, WebSearch `"{COMPANY_NAME}" earnings call transcript`
4. Download the latest 4 concall PDFs using `curl -sL` into `data/companies/{SYMBOL}/concalls/`
5. Read each downloaded PDF using the Read tool to extract transcript content

### Step 2: Read all transcripts

Read the 4 most recent `*_Transcript.md` files FULLY — management remarks AND Q&A. Do not skip or summarize.

### Step 3: Build company context

From the transcripts, establish:
- Sector and sub-sector
- Business segments and revenue mix
- Key products / services / customers
- Capacity base (current + planned)
- Cycle position (early/mid/late in growth cycle)
- Current consensus / market narrative (inferred from management guidance + analyst questions)

### Step 4: Identify VP factors

Scan for factors in **two buckets**:

**Company-specific growth drivers** (tag each if present with evidence):
- New product introduction
- Geographical expansion
- Growth in end-user industry
- Client mining / wallet share expansion
- Taking away market share
- Industry growth
- Expanding distribution
- Acquisitions
- Capital expenditure / capacity addition
- Margin expansion (mix / operating leverage / efficiency)
- Premiumisation-led growth
- Economies of scale
- Cost reduction / backward integration
- Operating leverage
- Regulatory / approval-led expansion

**Industry/sector VP buckets** (tag each if present):
- Demand-supply mismatch
- Regulatory changes / new norms
- New industry creation
- Consolidation

For each factor, capture:
- **Evidence snippet:** verbatim quote (max ~25 words) + source quarter
- **Why it matters:** mechanism to revenue / margin / cashflow
- **Time-to-impact:** 0-2Q / 3-4Q / 5-8Q / >8Q
- **Structural vs one-off:** state which and why

### Step 5: Create VP Scorecard

Rank by (Probability x Magnitude of Impact). Table columns:

| # | VP Factor | Type | Current Market Belief | Alternate Reality (VP) | Trigger(s) | Evidence (quote) | Timing | EPS Impact Path | Rerating Path | Probability | Kill-Switch |
|---|---|---|---|---|---|---|---|---|---|---|---|

Include at least 5 ranked factors.

### Step 6: Top 3 VP deep dives

For each of top 3 VPs, write 5-7 bullets:
- Why this is truly variant (not already priced in)
- The cleanest trigger to watch
- 2-3 variables to track each quarter

### Step 7: Forward-looking growth triggers

Extract ALL forward-looking triggers into a consolidated bullet list:
- Each trigger = one bullet point
- Start with action/outcome, not "management said"
- Include specific numbers and timelines
- Present/future tense only — NO past metrics
- Deduplicate across quarters (keep most recent version)

### Step 8: Write output

Write the COMPLETE analysis to:
```
data/companies/{SYMBOL}/growth_trigger_analysis.md
```

Use this structure:
```markdown
# {SYMBOL} — Variant Perception & Growth Triggers Analysis

## Company Context
[sector, segments, customers, capacity, cycle position, consensus narrative]

## Growth Drivers Checklist
[table of present factors with evidence]

## VP Scorecard
[ranked table]

## Top 3 VPs — Deep Dive
[5-7 bullets each]

## Growth Triggers
Key Triggers for {COMPANY_NAME}:
- [trigger 1]
- [trigger 2]
...
```

## Evidence rules

- Every VP MUST have verbatim evidence. If missing → "Unproven in provided docs"
- Don't mix one-offs vs structural — state which and why
- Be explicit on time horizons and falsifiers
- Cross-reference across quarters — use latest version if updated
- No past performance metrics in triggers — only forward-looking
- No generic statements — be specific with numbers and timelines

## Output location

```
data/companies/{SYMBOL}/growth_trigger_analysis.md
```
