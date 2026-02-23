# Awesome Stock Skills

A curated collection of [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skills for Indian stock market research and analysis.

These skills work as plug-and-play modules for Claude Code — give it domain expertise for equity research, concall analysis, financial data collection, and more.

## Skills

| Skill | Description |
|-------|-------------|
| [fetch-concalls](skills/fetch-concalls/) | Fetch & download conference call transcript PDFs for any Indian listed company. Uses a 3-tier fallback: screener.in → official BSE/NSE filings → third-party aggregators. |
| [growth-trigger-analysis](skills/growth-trigger-analysis/) | SOIC-style variant perception scorecard & growth trigger extraction from concall transcripts. Ranks VP factors by probability × impact, deep dives top 3, and produces forward-looking trigger list. |
| [nlm-skill](skills/nlm-skill/) | NotebookLM CLI & MCP expert — create notebooks, add sources (URLs, Drive, text), generate podcasts, reports, quizzes, flashcards, slides, mind maps, infographics, videos, and data tables. Chat with sources programmatically. Requires [`notebooklm-mcp-cli`](https://github.com/nicholasgriffintn/notebooklm-mcp-cli). |

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

## Contributing

Have a useful stock analysis skill? Open a PR! Each skill should:

1. Live in its own folder under `skills/`
2. Contain a `SKILL.md` with proper YAML frontmatter (`name` and `description`)
3. Be self-contained — no external dependencies beyond Claude Code's built-in tools

## License

MIT
