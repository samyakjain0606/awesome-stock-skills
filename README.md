# Awesome Stock Skills

A curated collection of [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skills for Indian stock market research and analysis.

These skills work as plug-and-play modules for Claude Code — give it domain expertise for equity research, concall analysis, financial data collection, and more.

## Skills

| Skill | Description |
|-------|-------------|
| [fetch-concalls](skills/fetch-concalls/) | Fetch & download conference call transcript PDFs for any Indian listed company. Uses a 3-tier fallback: screener.in → official BSE/NSE filings → third-party aggregators. |
| [growth-trigger-analysis](skills/growth-trigger-analysis/) | SOIC-style variant perception scorecard & growth trigger extraction from concall transcripts. Ranks VP factors by probability × impact, deep dives top 3, and produces forward-looking trigger list. |
| [nlm-skill](skills/nlm-skill/) | NotebookLM CLI & MCP expert — create notebooks, add sources (URLs, Drive, text), generate podcasts, reports, quizzes, flashcards, slides, mind maps, infographics, videos, and data tables. Chat with sources programmatically. Requires [`notebooklm-mcp-cli`](https://github.com/nicholasgriffintn/notebooklm-mcp-cli). |
| [stock-research-pipeline](skills/stock-research-pipeline/) | End-to-end equity research pipeline. Downloads concalls & investor presentations from screener.in, fetches financial data (P&L, balance sheet, ratios, shareholding), uploads everything to NotebookLM, runs 6 tailored analysis queries (business model, industry, management, financials, growth triggers, scenarios), and generates a professional PDF report with variant perception scorecard and Bull/Base/Bear scenarios. |
| [stock-research-publish](skills/stock-research-publish/) | Publish a generated equity report as a PDF, a magazine-quality HTML page, or both. Optionally one-command deploys the HTML to Netlify with an interactive setup that handles CLI install, auth, site name, and deploy scope. [Live preview →](https://gravita-equity.netlify.app) |

## Installation

### As a Plugin (recommended)

Install directly and get all skills + automatic updates:

```bash
claude plugin add github:samyakjain0606/awesome-stock-skills
```

### Manual

Copy individual skills into your project's `.claude/skills/` directory:

```bash
# Clone the repo
git clone https://github.com/samyakjain0606/awesome-stock-skills.git

# Copy a skill to your project
cp -r awesome-stock-skills/skills/fetch-concalls /path/to/your-project/.claude/skills/
```

## Usage

Once installed, Claude Code automatically picks up the skill. Just ask naturally:

```
> fetch concalls for YATHARTH
> get concall links for GRSE
> download concall transcripts for TCS
> growth triggers for SAILIFE
> VP analysis for HDFCBANK
> variant perception scorecard for TATAELXSI
> create a notebook for HDFCBANK concalls
> add this URL as source to my notebook
> generate a podcast for my notebook
> query my notebook about growth triggers
> /stock-research-pipeline GRAVITA
> deep dive on TCS
> equity report for RELIANCE
> run the stock research pipeline on HDFCBANK
> /stock-research-publish GRAVITA
> publish the TCS report
> deploy the RELIANCE report to netlify
> I want an html version of the HDFCBANK report
```

## Data Sources

- [screener.in](https://www.screener.in) — Primary source for aggregated BSE concall links
- [BSE India](https://www.bseindia.com) — Official regulatory filings
- [NSE India](https://www.nseindia.com) — Official regulatory filings

### nlm-skill prerequisites

The nlm-skill requires the `notebooklm-mcp-cli` CLI tool:

```bash
uv tool install notebooklm-mcp-cli
nlm login  # authenticate with Google
```

GitHub: [nicholasgriffintn/notebooklm-mcp-cli](https://github.com/nicholasgriffintn/notebooklm-mcp-cli)

### stock-research-pipeline prerequisites

Requires both the `nlm` CLI (see above) and Python with `reportlab` for PDF generation:

```bash
pip install reportlab
```

### stock-research-publish prerequisites

For HTML deploys to Netlify, the skill installs the CLI for you on first run (with
confirmation). To install manually:

```bash
npm install -g netlify-cli
netlify login
```

PDF output uses the same `reportlab` setup as `stock-research-pipeline`.

## Contributing

These skills are young and the best ones come from real-world stress-testing on dozens of
companies — different sectors, different reporting styles, different sources of edge. **Your
PRs make these skills better for everyone.**

### Add a new skill

1. Create a folder under `skills/your-skill-name/`
2. Add a `SKILL.md` with YAML frontmatter (`name` and `description`)
3. Add a `README.md` with a quick "what it does / setup / usage" section
4. Make it self-contained — no external dependencies beyond Claude Code's built-in tools and
   commonly available CLIs (Python, `nlm`, `netlify`, etc.)
5. Open a PR — include a short example of the skill running on a real company

### Improve an existing skill

Even small contributions help:

- **More sectors covered** — most skills lean industrial/manufacturing today; banks, NBFCs,
  insurance, FMCG, IT services, pharma, and SaaS each need sector-specific tweaks (NIM/CASA
  for banks, SSG for retail, ARR/NRR for SaaS, etc.).
- **Better data sources** — fallbacks for when screener.in is down, BSE/NSE direct scrapers,
  alternative financial data providers.
- **Tighter design rules** — particularly for the HTML/PDF outputs in `stock-research-publish`.
  Fonts, color palettes, mobile breakpoints, dark-mode polish.
- **More analysis frameworks** — DCF helpers, peer-comparison automation, technical analysis
  layers, ownership/insider tracking.
- **Bug reports with reproduction steps** — even if you can't fix it, a clear bug report on a
  named company is a real contribution.

### Ground rules for PRs

- **No fabricated outputs** — skills must fail loud when inputs are missing, never invent data.
- **No private data leakage** — be careful with example outputs; mock or anonymize anything
  proprietary.
- **Forbidden in HTML/visual outputs** — no Inter/Roboto fonts as primary, no indigo/violet
  gradient text, no emoji section headers, no glowing-shadow animations. These signal "AI
  template" and undermine trust. See `stock-research-publish/SKILL.md` for the full list.

If you have an idea but aren't sure how to scope it, open an issue first — happy to discuss.

## License

MIT
