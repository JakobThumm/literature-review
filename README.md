# Literature Review Skill

A Claude Code skill for conducting systematic, comprehensive literature reviews in robotics, AI, and machine learning. It searches multiple academic databases, synthesizes findings thematically, verifies all citations, and produces professional output documents in Markdown and PDF.

## What it does

- Searches arXiv, Semantic Scholar, IEEE Xplore, OpenReview, ACM DL, and Papers With Code
- Ranks results by citation count, venue tier, and author reputation
- Chains citations forward and backward from key papers
- Deduplicates and screens results following PRISMA-style methodology
- Writes thematic synthesis (not paper-by-paper summaries)
- Verifies all DOIs via CrossRef before finalizing
- Exports a formatted PDF with IEEE or APA citations

Supports two modes: a **simple review** (3-10 targeted references) and a **full review** (multi-section document with search flow diagram and quality assessment).

## Installation

### 1. Python dependencies

```bash
pip install requests
```

### 2. PDF generation (optional)

```bash
# macOS
brew install pandoc && brew install --cask mactex

# Linux
apt-get install pandoc texlive-xetex
```

Verify: `python scripts/generate_pdf.py --check-deps`

### 3. Semantic Scholar MCP server (recommended)

Provides structured access to 225M+ papers with citation graph traversal and author lookup.

```bash
git clone https://github.com/xiuyechen/semantic-scholar-mcp.git
cd semantic-scholar-mcp
uv pip install -e .
claude mcp add semantic-scholar -- semantic-scholar-mcp
```

With an API key (better rate limits):

```bash
claude mcp add semantic-scholar -e S2_API_KEY=your_key -- semantic-scholar-mcp
```

Get a free API key at [semanticscholar.org/product/api](https://www.semanticscholar.org/product/api).

### 4. IEEE Xplore API key (optional)

Required for searching ICRA, IROS, RA-L, and T-RO proceedings. Get a free key at [developer.ieee.org](https://developer.ieee.org/).

## Usage

Invoke the skill in Claude Code:

```
/literature-review
```

Then describe your topic. Examples:

- "Give me references supporting the use of domain randomization for sim-to-real transfer."
- "Do a full literature review on reinforcement learning for dexterous manipulation."

## Repository layout

```
skills/literature-review/
├── SKILL.md                        # Full skill instructions for Claude
├── scripts/
│   ├── search_databases.py         # Deduplicate and rank search results
│   ├── verify_citations.py         # Verify DOIs via CrossRef
│   └── generate_pdf.py             # Markdown to PDF export
├── assets/
│   └── review_template.md          # Full review document template
└── references/
    ├── citation_styles.md           # IEEE, APA, ACM formatting guide
    └── database_strategies.md       # Per-database search strategies
```
