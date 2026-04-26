# Stock Research Publish

Take the analysis outputs from the [stock-research-pipeline](../stock-research-pipeline/) skill
and turn them into a deliverable you can read or share — a polished print-style PDF, a
magazine-quality HTML page, or both. Optionally publish the HTML to Netlify and get a live URL
in under 30 seconds.

## Live preview

Sample output (mock GRAVITA data) — open on desktop and mobile:
**https://gravita-equity.netlify.app**

## What It Does

1. **Asks what you want** — PDF, HTML, or both, plus whether to deploy the HTML to Netlify.
   No defaults. The skill will not silently pick a format for you.
2. **Locates the analysis** — pulls the 6 NLM query answers (business model, industry,
   management, financials, growth triggers/VP, risks/scenarios) from
   `data/companies/{SYMBOL}/analysis/`. If they're missing, it tells you to run
   `/stock-research-pipeline` first instead of fabricating output.
3. **Generates PDF** (if requested) — uses the existing reportlab generator from the pipeline.
4. **Generates HTML** (if requested) — a single self-contained file with:
   - Editorial / Blueprint / Paper-ink aesthetic (no AI slop — explicit forbidden list for
     fonts, colors, gradient text, emoji headers, neon dashboards)
   - Sticky sidebar TOC on desktop, horizontal scrollable bar on mobile
   - Light + dark mode (auto from `prefers-color-scheme`)
   - Hero with drop-cap lead, KPI strip, exec summary with pull quote
   - Mermaid value-chain diagram (with proper zoom/pan container)
   - Chart.js financial trend chart
   - Peer comparison table, guidance-vs-delivery table, scenario detail table
   - 6-factor variant-perception scorecard with quantitative evidence + edge sizing
   - Bull/Base/Bear scenarios — cards + detailed P&L table + driver-assumptions sub-table +
     probability-weighted target with explicit math + "what moves us between scenarios" notes
   - Color-coded risk callouts, closing thesis, source disclosure
5. **Publishes to Netlify** (if requested) — interactive pre-flight handles:
   - **Install**: detects the CLI, asks whether to install globally, locally, or skip
   - **Auth**: detects login state, offers interactive `netlify login` or token paste
   - **Site name**: stable per-symbol (`gravita-equity`), date-stamped, custom, or reuse-existing
   - **Scope**: production (`--prod`) or preview/draft URL
   - **Sensitive content warning** before the actual deploy

**Input:** A SYMBOL with a completed stock-research-pipeline run, OR existing analysis files
in `data/companies/{SYMBOL}/analysis/`.
**Output:**
- `data/companies/{SYMBOL}/analysis/{SYMBOL}_Equity_Analysis_{date}.pdf`
- `data/companies/{SYMBOL}/analysis/{SYMBOL}_Equity_Analysis_{date}.html`
- A live Netlify URL like `https://{site-name}.netlify.app` (production) or
  `https://<hash>--{site-name}.netlify.app` (preview/draft)

## Setup

1. **Run the pipeline first** to produce analysis inputs:
   ```bash
   /stock-research-pipeline GRAVITA
   ```

2. **For PDF output** — Python venv with reportlab:
   ```bash
   pip install reportlab
   ```

3. **For Netlify deploy** — the skill installs the CLI for you (with confirmation) on first
   run. If you want to install it manually:
   ```bash
   npm install -g netlify-cli
   netlify login
   ```

## Usage

```
/stock-research-publish GRAVITA
```

Or natural language:

```
"publish the GRAVITA report"
"deploy the TCS report to netlify"
"I want an html version of the RELIANCE report"
"ship the HDFCBANK research"
"make the SAILIFE report shareable"
```

## Design philosophy

The HTML output is opinionated. The skill specifies a forbidden list (no Inter font, no
indigo/violet accents, no emoji section headers, no gradient text, no neon dashboards) because
those patterns scream "AI-generated template" and undermine trust in the analysis. Instead it
picks one of three constrained aesthetics — Editorial, Blueprint, or Paper-ink — and commits to
it.

Sections like Variant Perception and Bull/Base/Bear are intentionally heavy on data: the
scorecard demands quantitative evidence and edge sizing per factor, the scenario section
requires both a card view and a detailed table plus a driver-assumptions sub-table so every
number is challengeable. The aim is research that helps you THINK, not just read.

## Help us improve it

This skill is young. The HTML output, the design rules, and especially the section content
templates (VP scorecard, scenario assumptions) all benefit from real-world stress-testing. PRs
welcome — see [Contributing](../../README.md#contributing) on the repo root.

Specific improvement ideas:

- **More aesthetics**: the skill lists three (Editorial, Blueprint, Paper-ink) — add a fourth
  that fits commodity / cyclical names, or a Bloomberg-terminal-style data-dense option.
- **Sector-aware financial sections**: a bank report needs NIM/CASA tables, an FMCG report
  needs same-store growth, a SaaS report needs ARR/NRR. The current spec is mostly recycler-
  centric — make it adapt to sector.
- **Better mobile**: the current mobile breakpoint at 640px is functional but bland. Designer
  eyes welcome.
- **Other deploy targets**: GitHub Pages, Vercel, Cloudflare Pages — abstract the deploy step
  so users can pick their host.
- **Email-friendly HTML**: a separate output mode that inlines all CSS and works in Gmail/Outlook.
- **Slide-deck output**: hand off to the `visual-explainer:generate-slides` skill or similar
  for a magazine-style slide version.
- **Watermark / branding**: optional logo + footer line for users who want to share with a
  personal brand.
- **Compare-versions mode**: re-publishing the same symbol could diff against the previous
  report and highlight what changed quarter-on-quarter.

If any of those (or your own ideas) interest you, open a PR or an issue. Even small
improvements — better forbidden-fonts list, tighter mobile spacing, an extra VP factor — are
genuinely useful.
