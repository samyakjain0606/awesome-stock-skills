---
name: stock-research-publish
description: Publish a generated equity research report as either a professional PDF or a magazine-quality HTML page, then optionally deploy the HTML to Netlify and return a live URL. Use after the stock-research-pipeline has produced NLM query answers (Business Model, Industry, Management, Financials, Growth Triggers / VP, Risks / Bull-Base-Bear) for a SYMBOL, OR when the user says "publish report for [SYMBOL]", "deploy report to netlify", "html version of [SYMBOL] report", "ship the [SYMBOL] research", "make the [SYMBOL] report shareable", or "/stock-research-publish [SYMBOL]". Asks the user upfront whether they want PDF, HTML, or both.
---

# Stock Research Publish

Take the analysis outputs produced by `stock-research-pipeline` and turn them into a deliverable
the user can read or share. Two output modes — PDF (print-style) or HTML (web-style with optional
Netlify deploy). The user picks.

## Prerequisites

- An NLM analysis already exists for the SYMBOL — i.e. the 6 query answers from
  `stock-research-pipeline` are available in this session OR can be re-derived from
  `data/companies/{SYMBOL}/`. If neither is true, run `/stock-research-pipeline {SYMBOL}` first.
- For PDF: Python venv at `.venv/` with `reportlab` installed.
- For HTML deploy: `netlify` CLI on PATH **OR** a Netlify access token in `$NETLIFY_AUTH_TOKEN`.
  If the CLI is missing, the skill installs it (with user confirmation) — see Step 4 pre-flight.
  Requires `npm` on PATH (Node ≥ 18).

## Step 1: Ask the user what they want

Use `AskUserQuestion` with one question, three options:

- **PDF only** — print-style A4 report (existing reportlab generator).
- **HTML only** — single-file, magazine-quality web page with optional Netlify deploy.
- **Both** — produce PDF first, then HTML, then ask about deploy.

Also ask whether to deploy the HTML to Netlify if HTML is selected (yes / no). Do not branch
on whether `netlify` is installed at this stage — the Step 4 pre-flight handles install + auth +
site name + deploy scope interactively.

Do not proceed until you have explicit answers. No defaults.

## Step 2: Locate the analysis inputs

Resolve the SYMBOL from the user's request and gather the 6 NLM answers. Sources, in order:

1. The current conversation (if `/stock-research-pipeline` just ran in this session).
2. `data/companies/{SYMBOL}/analysis/` — look for any of:
   `business_model.md`, `industry.md`, `management.md`, `financials.md`, `growth_triggers.md`,
   `risks_scenarios.md`, or a single `nlm_answers.json` if the pipeline saved one.
3. As a last resort, re-run only the missing NLM queries from the pipeline (do NOT re-download
   PDFs or recreate the notebook). Use the existing notebook ID from
   `data/companies/{SYMBOL}/analysis/notebook.txt` if present.

If you can't gather all 6 answers, stop and ask the user to run `/stock-research-pipeline` first.

Also load `data/companies/{SYMBOL}/screener_data.md` for headline metrics (PE, ROE, ROCE, market
cap, latest revenue/PAT) — these populate the KPI cards.

## Step 3a: PDF path

Use the existing reportlab generator at `scripts/generators/equity_report_pdf.py` (or, if absent,
the working reference at `data/companies/GRAVITA/analysis/generate_report.py`). Render to:

```
data/companies/{SYMBOL}/analysis/{SYMBOL}_Equity_Analysis_{YYYY-MM-DD}.pdf
```

Then `open` the file and tell the user the path. Done if PDF-only was chosen.

## Step 3b: HTML path — design rules

Generate a single self-contained `.html` file (no external assets except Google Fonts and one
optional Mermaid CDN). Apply the visual-explainer playbook — these are not suggestions.

**Aesthetic — pick ONE per generation. Vary across runs.**
- **Editorial** (default for equity reports): Instrument Serif or Crimson Pro headlines,
  IBM Plex Sans body, JetBrains Mono for monospace. Generous whitespace, deep navy + muted gold,
  serif drop-caps on the executive summary.
- **Blueprint**: subtle grid background, deep slate/blue, monospace labels, precise borders.
  Good for technical/cyclical companies.
- **Paper/ink**: warm cream `#faf7f5` background, terracotta + sage accents. Good for
  consumer / FMCG names where warmth fits.

**Forbidden — do not produce these.**
- Inter / Roboto / system-ui as `--font-body`. Pick a real pairing.
- Indigo/violet accents (`#8b5cf6`, `#7c3aed`, `#a78bfa`), cyan-magenta-pink neon, gradient text
  on headings.
- Emoji icons in section headers. Use styled monospace labels with colored dot indicators or
  numbered badges.
- "Neon dashboard" or gradient-mesh backgrounds. Use subtle radial glows or faint grid only.
- Animated glowing box-shadows or pulsing effects on static content.
- Three-dot window chrome on code/quote blocks.

**Required structure** (mirror the PDF section list, but use web idioms):

1. **Hero** — Company name (serif, 48–64px), SYMBOL in monospace, sector tag, report date.
   One-line investment thesis as a lead paragraph (larger, dimmer body color).
2. **KPI strip** — 4–6 cards: CMP, MCap, P/E, ROE, ROCE, latest QoQ growth. Hero number large;
   label small and dim. Vary one card as the "primary" with accent-tinted background.
3. **Executive summary** — 2–3 short paragraphs synthesizing the thesis. Optional pull quote
   for the single most important management line.
4. **Business model & value chain** (Query 1) — CSS Grid cards, not prose dump. Extract the
   revenue streams, unit economics, key cost drivers. If there's a flow (input → processing →
   output → customer), use a small Mermaid `graph LR` (≤6 nodes) with the full
   `diagram-shell` zoom container — never bare `<pre class="mermaid">`.
5. **Industry & competitive positioning** (Query 2) — 2-column layout: structure cards on left,
   peer comparison `<table>` on right (sticky header, alternating rows, right-aligned numbers
   with `tabular-nums`).
6. **Financial deep dive** (Query 4) — A real `<table>` for the quarterly P&L. Add a Chart.js
   line chart for revenue + margin trajectory if 4+ quarters available. Margin trend gets its
   own KPI callout.
7. **Management quality & execution** (Query 3) — Guidance-vs-delivery as a striped table.
   Italic gray management quotes inset with a subtle left-accent bar (no quote-mark emoji).
8. **Growth triggers** (Query 5) — Card grid, each card = one trigger with: name, evidence,
   sizing, kill-switch. Color the left border by conviction (high / medium / watch).
9. **Variant perception scorecard** (Query 5) — This is a flagship section, not a quick table.
   It must include:
   (a) A 1–2 sentence methodology note explaining what "consensus" means here (sell-side target,
   peer trading multiple, common buy-side view) and what the edge is being measured against.
   (b) A wide `<table>` with **at least these columns**: factor | consensus view (with a
   sourceable anchor — sell-side target, peer multiple, prior management commentary) |
   variant view (the differentiated take) | quantitative evidence (specific numbers, dates,
   data points that support variant) | edge size (in EPS / multiple / target terms — i.e. how
   much this factor moves the price if right) | time horizon (when this should resolve) |
   falsification (what would prove the variant wrong).
   (c) A minimum of **5 factors**, not 3. Cover at least: valuation multiple, the most-feared
   risk (commodity / cyclical / regulatory), execution / capex, demand durability, and one
   sector-specific factor.
   (d) Below the table, a short "consensus build vs variant build" comparison — two side-by-side
   mini-tables showing the EPS/revenue/multiple stack each view implies for the next 2 years.
10. **Bull / Base / Bear scenarios** (Query 6) — Two complementary views, both required:
    (a) **Three side-by-side scenario cards** with FY+1/+2/+3 headline numbers, probability
    weight, return %, and one-line trigger. Bear uses rose/cranberry, base neutral, bull
    sage/emerald — border color only, **never** red/green emoji or arrows.
    (b) **A detailed scenario `<table>`** below the cards, with scenarios as columns and rows
    covering at minimum: Revenue FY+1/+2/+3, Revenue CAGR, EBITDA FY+3, EBITDA margin %,
    PAT FY+3, EPS FY+3, EPS CAGR, exit P/E multiple, target price, implied return %,
    and probability. Numbers are right-aligned with `tabular-nums`. Header row tinted to match
    the scenario card colors so the table is visually anchored to the cards above.
    (c) **A scenario assumptions sub-table** listing the 4–6 key driver assumptions per
    scenario (e.g. for a recycler: spread per kg, capacity utilization %, organized share %,
    non-core revenue mix, tax rate). This makes the scenarios reproducible — the reader can
    challenge any single assumption.
    (d) A probability-weighted target line below the table, with the math shown explicitly
    (e.g. `0.20 × 1,500 + 0.55 × 2,400 + 0.25 × 3,500 = 2,495`). Show the implied upside vs CMP.
    (e) A short "what moves us between scenarios" note — 2–3 bullets describing the specific
    observable that would shift the base case toward bull or bear.
11. **Key risks** (Query 6) — Color-coded callout boxes (rose for high severity, amber for
    medium, slate for informational). Each risk has a 1-line "monitor via" line.
12. **Investment thesis summary** — Closing prose, 4–6 sentences max, with the kill-switches
    explicit.
13. **Footer** — sources (concall dates analyzed, screener URL, disclaimer "not investment
    advice").

**Page mechanics**
- Sticky sidebar TOC on desktop ≥1024px (linking to each section); horizontal scrollable bar
  on mobile. See visual-explainer's `references/responsive-nav.md` pattern.
- Light + dark via `@media (prefers-color-scheme)`. Both must look intentional.
- Staggered `fadeUp` on cards via `--i` CSS variable; respect `prefers-reduced-motion`.
- Every grid/flex child needs `min-width: 0` to prevent overflow on narrow viewports.
- Tabular numbers everywhere (`font-variant-numeric: tabular-nums`).

**Output location**
```
data/companies/{SYMBOL}/analysis/{SYMBOL}_Equity_Analysis_{YYYY-MM-DD}.html
```

After writing, `open` the file locally so the user can preview before deploy.

## Step 4: Netlify deploy (HTML path only, if user said yes)

This step has two phases: a **pre-flight** that gathers everything Netlify needs from the user
(CLI, auth, site name, deploy scope), then the **deploy** itself. Do not skip the pre-flight —
it produces the variables the deploy command consumes. Each `AskUserQuestion` below is
mandatory; do not invent defaults the user did not pick.

### 4.1 Pre-flight: Netlify CLI

```bash
command -v netlify >/dev/null 2>&1 && netlify --version
```

If the command is **missing**, ask the user via `AskUserQuestion`:

> **Netlify CLI is not installed. Install it now?**
> - **Install globally** — runs `npm install -g netlify-cli` (one-time, ~30 MB). Requires npm.
> - **Install locally to project** — runs `npm install --save-dev netlify-cli` and uses `npx netlify` thereafter. No global pollution.
> - **Skip deploy** — keep HTML local only, print manual install + deploy commands.

If the user picks an install option:
- First verify npm exists: `command -v npm` — if missing, abort with the Node install link
  (`https://nodejs.org/`) and fall back to "skip deploy".
- Run the install. If it fails with `EACCES`, do NOT use `sudo` automatically — instead tell the
  user to either fix npm prefix (`npm config set prefix ~/.npm-global`) or pick local install
  instead, then re-ask.
- After successful install, re-check `netlify --version` to confirm.

### 4.2 Pre-flight: Authentication

```bash
netlify status --json 2>/dev/null
```

If the JSON shows `"NetlifyUser"` with an email, the user is authed — continue.
If not authed, ask via `AskUserQuestion`:

> **Netlify needs to authenticate. Which method?**
> - **Interactive login** — runs `netlify login`, opens a browser, paste-back token flow. Best for personal use.
> - **Personal access token** — paste a token from `https://app.netlify.com/user/applications#personal-access-tokens`. Best for CI / scripted use. The skill will export `NETLIFY_AUTH_TOKEN` for the session and remind the user to add it to their shell profile if they want it persistent.
> - **Skip deploy** — same as above, keep local only.

If interactive login: run `netlify login` (it opens the browser; the user pastes the auth code
back into the terminal). Wait for it to complete. Re-check `netlify status`.

If token: prompt the user to paste the token (do NOT log it; treat it like a password). Set it
in the current shell only:
```bash
export NETLIFY_AUTH_TOKEN="<token>"
```
Tell the user this is session-scoped — to make it persistent, they should add the export to
`~/.zshrc` or use a tool like `direnv` themselves. Do not modify their shell profile without
explicit permission.

### 4.3 Pre-flight: Site name

Ask via `AskUserQuestion`:

> **What site name should this report deploy to?** (becomes the public URL: `https://{name}.netlify.app`)
> - **Default — `{symbol-lower}-equity-{yyyymmdd}`** — date-stamped, new URL per run. Good for ad-hoc reports.
> - **Stable per-symbol — `{symbol-lower}-equity`** — reused on every re-publish for this stock. Good if you want a consistent URL to share.
> - **Custom** — user supplies the name (must be globally unique on netlify.app).
> - **Reuse existing site** — only offer this option if `data/companies/{SYMBOL}/analysis/netlify_site.txt` exists with a previously-deployed site name; auto-fill it.

After the user picks, save the chosen site name to
`data/companies/{SYMBOL}/analysis/netlify_site.txt` so future runs can offer "reuse existing".

### 4.4 Pre-flight: Deploy scope

Ask via `AskUserQuestion`:

> **Deploy as production or preview?**
> - **Production** (`--prod`) — replaces the live URL with this build. Use for the canonical version.
> - **Preview** — creates a one-off `https://<hash>--{site}.netlify.app` URL that won't replace prod. Good for sharing a draft for feedback before publishing.

### 4.5 Deploy

Stage the HTML, ensure the site exists, deploy.

```bash
DEPLOY_DIR=$(mktemp -d)
trap 'rm -rf "$DEPLOY_DIR"' EXIT
cp "data/companies/{SYMBOL}/analysis/{SYMBOL}_Equity_Analysis_{YYYY-MM-DD}.html" \
   "$DEPLOY_DIR/index.html"

SITE_NAME="<from 4.3>"

# Create site if it doesn't exist (idempotent — ignore "already exists")
netlify sites:create --name "$SITE_NAME" 2>/dev/null || true

# Deploy
PROD_FLAG="<--prod or empty per 4.4>"
netlify deploy --dir="$DEPLOY_DIR" --site="$SITE_NAME" $PROD_FLAG \
  --message "Equity report {SYMBOL} {YYYY-MM-DD}"
```

Parse the output for `Website URL` (production) or `Website Draft URL` (preview). Print:

```
✓ Published to Netlify
Live URL:  https://{site-name}.netlify.app   # or the draft URL
Local:     data/companies/{SYMBOL}/analysis/{SYMBOL}_Equity_Analysis_{YYYY-MM-DD}.html
Site name: {site-name}  (saved to netlify_site.txt for reuse)
```

### 4.6 Sensitive content warning

**Always show this before running 4.5**, even if the user already opted into deploy:

> Netlify URLs are **public and crawlable**. This report contains analysis of {SYMBOL} based on
> public concalls and screener data — no proprietary data — but if you've added private notes or
> internal commentary to the HTML, abort the deploy now.

If the user has not seen this warning in this session, ask one final yes/no confirmation before
calling `netlify deploy`.

## Step 5: Confirm and summarize

Report back, in 3–4 lines max:
- What was generated (PDF path / HTML path / Netlify URL).
- Number of sections rendered.
- Any sections that were thin because NLM didn't return useful content (so the user knows where
  to push deeper next time).

## Quality checks before delivery (HTML)

Before claiming done, mentally run the visual-explainer checks:
- **Squint test**: hierarchy still readable when blurred?
- **Swap test**: would replacing fonts + colors with a generic dark theme make this
  indistinguishable? If yes, push the aesthetic further.
- **Both themes**: does it look intentional in light AND dark?
- **No overflow**: nothing escapes its container at 375px width.
- **No slop signals**: no Inter, no purple gradients, no emoji headers, no glowing shadows.

If two or more slop signals are present, regenerate with a different aesthetic.

## Error handling

- **Missing analysis inputs:** stop, tell user to run `/stock-research-pipeline` first.
- **Reportlab missing:** `pip install reportlab` in the venv, then retry.
- **Netlify CLI missing:** Step 4.1 handles this — ask the user via `AskUserQuestion` whether to
  install globally (`npm install -g netlify-cli`), install locally to project, or skip deploy.
  Never auto-install without asking.
- **npm missing:** if the user picks an install option but `npm` is not on PATH, abort the
  install and fall back to "skip deploy" with a pointer to `https://nodejs.org/`.
- **npm EACCES on global install:** do NOT escalate with `sudo`. Show the user the npm-prefix
  fix (`npm config set prefix ~/.npm-global` + add to PATH) or offer the local install path,
  then re-ask.
- **Netlify auth missing:** Step 4.2 asks the user to pick interactive `netlify login` or paste
  a personal access token.
- **Token paste:** treat the token like a password — never log, never echo, never write to disk.
  Set as `NETLIFY_AUTH_TOKEN` for the session only. Tell the user how to make it persistent
  themselves; never edit `~/.zshrc` automatically.
- **Netlify site name collision:** if the user chose "default" or "custom" and that name is
  taken, ask whether to append a suffix (`-v2`, `-v3`) or pick a new name.

## Example usage

```
User: "publish the GRAVITA report"
User: "deploy the GRAVITA report to netlify"
User: "I want an html version of the TCS report"
User: "/stock-research-publish RELIANCE"
```
