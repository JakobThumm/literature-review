---
name: literature-review
description: Conduct comprehensive, systematic literature reviews using multiple academic databases (arXiv, Semantic Scholar, IEEE Xplore, ACM DL, OpenReview, Papers With Code, etc.). This skill should be used when conducting systematic literature reviews, research synthesis, or comprehensive literature searches across robotics, AI, machine learning, and related engineering domains. Creates professionally formatted markdown documents and PDFs with verified citations in multiple citation styles (APA, IEEE, etc.).
allowed-tools: Read Write Edit Bash
license: MIT license
metadata:
    skill-author: K-Dense Inc.
---

# Literature Review

## Overview

Conduct systematic, comprehensive literature reviews following rigorous academic methodology. Search multiple literature databases, synthesize findings thematically, verify all citations for accuracy, and generate professional output documents in markdown and PDF formats.

This skill is focused on **robotics and AI** research, covering areas such as robot learning, motion planning, perception, manipulation, reinforcement learning, large language models, computer vision, and human-robot interaction.

## When to Use This Skill

Use this skill when:
- Conducting a systematic literature review for research or publication
- Synthesizing current knowledge on a robotics or AI topic across multiple sources
- Writing the related work or background section of a research paper or thesis
- Investigating the state of the art in a robotics or AI domain
- Identifying research gaps and future directions
- Finding benchmark comparisons and baseline methods
- Requiring verified citations and professional formatting

## First Step: Identify if Simple Question or Full Review

First, decide if this question requires a simple literature review or a full one.
A simple review returns 1-10 references and is targeted on a very specific sub-part of a full literature review.
A full literature review searches the web extensively and structures the result into multiple sub-sections.

---

## General Guidelines

These guidelines apply to **both** simple and full reviews.

### Database-Specific Search Guidance

#### arXiv

Primary preprint server for CS, ML, and robotics. Access via direct API:
```
GET https://export.arxiv.org/api/query?search_query=cat:cs.RO+AND+ti:manipulation&max_results=50&sortBy=relevance
```

Relevant categories:
- `cs.RO` — Robotics
- `cs.LG` — Machine Learning
- `cs.AI` — Artificial Intelligence
- `cs.CV` — Computer Vision and Pattern Recognition
- `cs.NE` — Neural and Evolutionary Computing
- `stat.ML` — Machine Learning

**Search tips**:
- `ti:` — title field, `abs:` — abstract field
- Combine: `cat:cs.RO AND (ti:"motion planning" OR abs:"trajectory optimization")`
- Date filter: `submittedDate:[20200101 TO 20250408]`

#### Semantic Scholar

Access via free API (200M+ papers, cross-disciplinary):
```
GET https://api.semanticscholar.org/graph/v1/paper/search?query=<terms>&fields=title,authors,year,citationCount,externalIds,abstract&limit=20
```

**Search tips**:
- Use `fields=` to request citation counts — essential for ranking by impact
- Retrieve recommendations for a known key paper: `/paper/{id}/recommendations`
- Use `/paper/{id}/citations` and `/paper/{id}/references` for citation chaining

#### IEEE Xplore

Covers ICRA, IROS, RA-L, T-RO, and other top robotics/systems venues:
```
GET https://ieeexploreapi.ieee.org/api/v1/search/articles?querytext=<terms>&max_records=25&sort_order=desc&sort_field=article_citation_count
```
Requires a free API key from IEEE.

#### OpenReview

Covers NeurIPS, ICML, ICLR, CoRL, and many ML/robotics workshops:
```
GET https://api2.openreview.net/notes?content.title=<terms>&details=replyCount&limit=25
```

Includes submitted, under-review, and rejected papers. Filter by `venueid` for specific conferences.

#### ACM Digital Library

For HRI, CHI, and RSS proceedings:
```
GET https://dl.acm.org/action/doSearch?query=<terms>&pageSize=10
```

#### Papers With Code

Find papers with open-source implementations and benchmark leaderboards:
```
GET https://paperswithcode.com/api/v1/papers/?q=<terms>&items_per_page=10
```

Use to verify code availability and compare results on standard benchmarks (e.g., MuJoCo locomotion, D4RL, Isaac Gym).

#### Citation Chaining

Expand search via citation networks using Semantic Scholar:

1. **Forward citations** (papers citing key papers): `/paper/{id}/citations` — finds newer work building on seminal methods
2. **Backward citations** (references from key papers): `/paper/{id}/references` — finds foundational and closely related work

### Rate Limit Error Handling

For each database request, check the HTTP response code before processing results:

| Code | Meaning | Action |
|------|---------|--------|
| 200 | OK | Process results normally |
| 429 | Rate limited | Log incident, skip to next database |
| 403 | Forbidden | Log incident, skip to next database |
| 5xx | Server error | Log incident, skip to next database |

When a request is rejected, record the following internally and continue:
```
[SKIPPED] <DatabaseName>: HTTP <code> — <reason if provided>
```

Do **not** retry the request. Move on immediately to the next database. Always include a **Search Summary** in the final output (format varies by review type — see below). If all databases were rejected, report this clearly and return no references rather than fabricating results.

### Prioritizing High-Impact Papers

**Always prioritize influential, highly-cited papers from reputable authors and top venues.** Quality matters more than quantity.

#### Citation Count Thresholds

| Paper Age | Citation Threshold | Classification |
|-----------|-------------------|----------------|
| 0-3 years | 20+ citations | Noteworthy |
| 0-3 years | 100+ citations | Highly Influential |
| 3-7 years | 100+ citations | Significant |
| 3-7 years | 500+ citations | Landmark Paper |
| 7+ years | 500+ citations | Seminal Work |
| 7+ years | 1000+ citations | Foundational |

#### Conference and Journal Tiers

- **Tier 1 (Always Prefer):** NeurIPS, ICML, ICLR, CVPR, ICRA, RSS, CoRL, Science Robotics, IJRR, T-RO, RA-L
- **Tier 2 (Strong Preference):** IROS, ECCV, ICCV, AISTATS, UAI, AAAI, IJCAI, Autonomous Robots
- **Tier 3 (Include When Relevant):** Workshops at Tier-1 venues, respected specialized journals
- **Tier 4 (Use Sparingly):** Lower-impact peer-reviewed venues, non-peer-reviewed technical reports

#### Author Reputation

Prefer papers from senior researchers with high h-index (>30), leading labs (DeepMind, OpenAI, CMU RI, MIT CSAIL, Stanford AI Lab, Berkeley AI Research, ETH Zurich, etc.), and authors with multiple Tier-1 publications.

#### Identifying Seminal Papers

1. High citation count (500+ for papers 5+ years old)
2. Frequently cited by other included papers
3. Published in Tier-1 venues
4. Written by field pioneers

### Citation Style Guide

Detailed formatting guidelines are in `references/citation_styles.md`. Quick reference:

**IEEE** (preferred for robotics/AI):
- In-text: Bracketed numbers [1], [2], [1]–[3]
- Reference: J. D. Smith, M. L. Johnson, and K. R. Williams, "Title," in *Proc. ICRA*, 2023, pp. 301–318. doi: 10.xxx/yyy

**APA (7th Edition)**:
- In-text: (Smith et al., 2023)
- Reference: Smith, J. D., Johnson, M. L., & Williams, K. R. (2023). Title. *Proc. ICRA*, 301–318. https://doi.org/10.xxx/yyy

**Always verify citations** with `verify_citations.py` before finalizing.

### Common Pitfalls to Avoid

1. **Single database search**: Misses relevant papers; always search multiple databases
2. **No search documentation**: Makes the search irreproducible; document all queries
3. **Paper-by-paper summary**: Lacks synthesis; organize thematically instead
4. **Unverified citations**: Leads to errors; always run verify_citations.py
5. **Too broad search**: Yields irrelevant results; refine with specific terms
6. **Too narrow search**: Misses relevant papers; include synonyms and related terms
7. **Ignoring arXiv**: Misses latest findings; robotics/AI moves faster than journal cycles
8. **Ignoring benchmark context**: Treats all claims equally; compare on shared benchmarks
9. **Recency bias**: Newer isn't always better; include seminal foundational work
10. **Outdated search**: Field evolves rapidly; clearly state search date

---

## Core Workflow for Simple Question

Use this for targeted questions like "give me references that support X" or "what are the most relevant papers on Y approach". Returns 3-10 high-quality references.

### Step 1: Extract Search Query
Identify the core concept from the user's question or statement. Formulate a **single focused search string** (use AND/OR to combine 2-5 key terms). Do not run multiple queries per database.

### Step 2: Search Databases (one query each)
Run the same query across 2-3 databases. For robotics and AI, default to:

| Domain | Databases |
|--------|-----------|
| ML / Deep Learning | Semantic Scholar + arXiv |
| Robotics / Control | Semantic Scholar + arXiv |
| Computer Vision | Semantic Scholar + arXiv |
| HRI / HCI | Semantic Scholar + ACM DL |

See **General Guidelines → Database-Specific Search Guidance** for API endpoints and search tips. Apply rate limit error handling from **General Guidelines → Rate Limit Error Handling**.

Include a Search Summary before the references:
```
### Search Summary
- Semantic Scholar: ✓ 10 results
- arXiv:           ✗ HTTP 429 (rate limited) — skipped
- ACM DL:          ✓ 6 results
```

### Step 3: Select and Return References
From the combined results, pick the **3-10 most relevant** papers using the criteria in **General Guidelines → Prioritizing High-Impact Papers**. Also prefer:
- Papers whose title/abstract directly address the user's question
- Recent papers (last 5 years) unless the question asks for foundational work
- Papers with available code (check Papers With Code)

For each selected paper, provide:
- Full citation (authors, year, title, venue) — see **General Guidelines → Citation Style Guide**
- DOI or arXiv ID
- 1-sentence explanation of why it supports the statement/approach

Do **not** run deduplication scripts, generate PDFs, or create flow diagrams for simple reviews.

---

## Core Workflow for Full Review

Literature reviews follow a structured, multi-phase workflow:

### Phase 1: Planning and Scoping

1. **Define Research Question**: Frame the question clearly around a system, method, or problem:
   - Example: "What reinforcement learning approaches have been applied to dexterous robotic manipulation, and how do they compare in terms of sample efficiency and sim-to-real transfer?"

2. **Establish Scope and Objectives**:
   - Define clear, specific research questions
   - Determine review type (narrative, systematic, benchmark survey)
   - Set boundaries (time period, task domain, evaluation metrics)

3. **Develop Search Strategy**:
   - Identify 2-4 main concepts from research question
   - List synonyms, abbreviations, and related terms for each concept
   - Plan Boolean operators (AND, OR, NOT) to combine terms
   - Select minimum 3 complementary databases

4. **Set Inclusion/Exclusion Criteria**:
   - Date range (e.g., last 5 years: 2020-2025; robotics/AI moves fast)
   - Language (typically English)
   - Publication types (peer-reviewed conference papers, journals, preprints)
   - Study types (empirical, theoretical, survey, benchmark)
   - Document all criteria clearly

### Phase 2: Systematic Literature Search

1. **Multi-Database Search** — see **General Guidelines → Database-Specific Search Guidance** for API endpoints:

   **Primary Sources (always search):**
   - arXiv — dominant preprint venue for CS/ML/robotics
   - Semantic Scholar — 200M+ papers, cross-disciplinary
   - IEEE Xplore — ICRA, IROS, RA-L, T-RO, and other robotics venues
   - OpenReview — NeurIPS, ICML, ICLR, CoRL submissions and reviews

   **Secondary Sources (search when relevant):**
   - ACM Digital Library — HRI, CHI, RSS proceedings
   - Papers With Code — benchmark results and reproducibility info
   - Google Scholar — broad coverage including workshop papers

2. **Document Search Parameters**:
   ```markdown
   ### Database: arXiv
   - **Date searched**: 2025-04-08
   - **Date range**: 2020-01-01 to 2025-04-08
   - **Search string**: cat:cs.RO AND (ti:"dexterous manipulation") AND (abs:"reinforcement learning")
   - **Status**: ✓ 183 articles  (or ✗ HTTP 429 — skipped)
   ```

   Apply rate limit handling from **General Guidelines → Rate Limit Error Handling**. Include a Search Summary table in the Methodology section:

   | Database | Status | Results |
   |----------|--------|---------|
   | arXiv | ✓ | 183 |
   | Semantic Scholar | ✓ | 247 |
   | IEEE Xplore | ✗ HTTP 429 (rate limited) | — |
   | OpenReview | ✓ | 94 |

3. **Export and Aggregate Results**:
   ```bash
   python search_databases.py combined_results.json \
     --deduplicate \
     --format markdown \
     --output aggregated_results.md
   ```

### Phase 3: Screening and Selection

1. **Deduplication**:
   ```bash
   python search_databases.py results.json --deduplicate --output unique_results.json
   ```
   - Removes duplicates by DOI (primary) or title (fallback)
   - Document number of duplicates removed

2. **Title Screening**: Review all titles against inclusion/exclusion criteria; document exclusions.

3. **Abstract Screening**: Read abstracts of remaining papers; apply criteria rigorously; document reasons for exclusion.

4. **Full-Text Screening**: Obtain full texts; conduct detailed review; record final number of included papers.

5. **Create Search Flow Diagram**:
   ```
   Initial search: n = X
   ├─ After deduplication: n = Y
   ├─ After title screening: n = Z
   ├─ After abstract screening: n = A
   └─ Included in review: n = B
   ```

### Phase 4: Data Extraction and Quality Assessment

1. **Extract Key Data** from each included paper:
   - Metadata (authors, year, venue, DOI/arXiv ID)
   - Problem formulation and task domain
   - Method/architecture and key design choices
   - Evaluation setup (benchmarks, baselines, metrics)
   - Main results and reported performance numbers
   - Limitations noted by authors
   - Code/data availability

2. **Assess Paper Quality**:
   - **Empirical papers**: Are baselines fair and up-to-date? Are ablations provided? Is evaluation reproducible?
   - **Theoretical papers**: Are proofs complete? Are assumptions realistic?
   - **Survey/benchmark papers**: Is coverage comprehensive? Are comparisons standardized?
   - Note code availability (check Papers With Code)
   - Rate each paper: Strong, Moderate, or Weak contribution

3. **Organize by Themes**:
   - Identify 3-5 major themes across papers
   - Group papers by theme (papers may appear in multiple themes)
   - Note patterns, consensus methods, and open controversies

### Phase 5: Synthesis and Analysis

1. **Create Review Document** from template:
   ```bash
   cp assets/review_template.md my_literature_review.md
   ```

2. **Write Thematic Synthesis** (NOT paper-by-paper summaries):
   - Organize Results section by themes or research questions
   - Synthesize findings across multiple papers within each theme
   - Compare and contrast different approaches and results
   - Identify consensus areas and points of controversy
   - Highlight the strongest evidence and benchmark leaders

   Example structure:
   ```markdown
   #### 3.3.1 Theme: Sim-to-Real Transfer for Manipulation

   Domain randomization has become the dominant approach for sim-to-real
   transfer in dexterous manipulation^1-8^. OpenAI Dactyl^1^ demonstrated
   that randomizing physics parameters (friction, mass, damping) at training
   time enables zero-shot transfer to real hardware for in-hand rotation.
   Subsequent work showed that visual domain randomization^4,6^ further
   reduces the reality gap for camera-based policies. In contrast, system
   identification approaches^9-12^ achieve better sample efficiency on the
   target system but require real-world data collection.
   ```

3. **Critical Analysis**:
   - Evaluate methodological strengths and limitations across papers
   - Assess reproducibility and benchmark fairness
   - Identify open problems and under-explored directions

4. **Write Discussion**:
   - Interpret findings in broader research context
   - Discuss practical implications for deployment or future research
   - Acknowledge limitations of the review itself
   - Compare with previous surveys if applicable
   - Propose specific future research directions

### Phase 6: Citation Verification

**CRITICAL**: All citations must be verified before final submission.

```bash
python scripts/verify_citations.py my_literature_review.md
```

This script extracts all DOIs, verifies each resolves correctly, retrieves metadata from CrossRef, and outputs properly formatted citations. Re-run until all citations pass. Choose one citation style throughout — see **General Guidelines → Citation Style Guide**.

### Phase 7: Document Generation

```bash
python scripts/generate_pdf.py my_literature_review.md \
  --citation-style ieee \
  --output my_review.pdf
```

Options: `--citation-style` (apa, ieee, acm, chicago), `--no-toc`, `--no-numbers`, `--check-deps`.

**Quality Checklist**:
- [ ] All DOIs verified with verify_citations.py
- [ ] Citations formatted consistently (IEEE preferred)
- [ ] Search flow diagram included
- [ ] Search methodology and Summary table documented
- [ ] Inclusion/exclusion criteria clearly stated
- [ ] Results organized thematically (not paper-by-paper)
- [ ] Quality assessment completed
- [ ] Limitations acknowledged
- [ ] PDF generates without errors

### Best Practices (Full Review)

**Search**: Use minimum 3 databases; include arXiv; document all search strings and result counts; test and refine queries; sort by citations to surface influential work first.

**Screening**: Document inclusion/exclusion criteria upfront; screen Title → Abstract → Full text; record reasons for exclusions; prioritize reproducible work with public code.

**Synthesis**: Group by themes, not individual papers; compare and contrast across papers; evaluate reproducibility and benchmark fairness; identify gaps.

**Writing**: Be objective, systematic, and specific (include performance numbers, benchmark names, dataset sizes); use clear headings and logical flow.

---

## Example Workflow

```bash
# 1. Create review document from template
cp assets/review_template.md sim_to_real_rl_review.md

# 2. Search multiple databases (see General Guidelines for API endpoints)
# - arXiv: cat:cs.RO AND (abs:"sim-to-real" OR abs:"domain randomization")
# - Semantic Scholar: "sim-to-real transfer reinforcement learning robotics"
# - IEEE Xplore: "sim-to-real" AND "robot"
# - OpenReview: search CoRL and ICML venues

# 3. Aggregate and process results
python scripts/search_databases.py combined_results.json \
  --deduplicate \
  --rank citations \
  --year-start 2018 \
  --year-end 2025 \
  --format markdown \
  --output search_results.md \
  --summary

# 4. Screen results and extract data
# - Screen titles, abstracts, full texts
# - Extract key data into the review document
# - Organize by themes (domain randomization, system ID, adaptive methods, etc.)

# 5. Write the review (introduction, methodology, thematic results, discussion, conclusions)

# 6. Verify all citations
python scripts/verify_citations.py sim_to_real_rl_review.md
cat sim_to_real_rl_review_citation_report.json
# Fix any failed citations and re-verify

# 7. Generate professional PDF
python scripts/generate_pdf.py sim_to_real_rl_review.md \
  --citation-style ieee \
  --output sim_to_real_rl_review.pdf
```

## Resources

### Bundled Resources

- `scripts/verify_citations.py`: Verify DOIs and generate formatted citations
- `scripts/generate_pdf.py`: Convert markdown to professional PDF
- `scripts/search_databases.py`: Process, deduplicate, and format search results
- `references/citation_styles.md`: Detailed citation formatting guide (APA, IEEE, ACM)
- `references/database_strategies.md`: Comprehensive database search strategies
- `assets/review_template.md`: Complete literature review template with all sections

### External Resources

**Databases:**
- arXiv CS: https://arxiv.org/search/cs
- Semantic Scholar: https://www.semanticscholar.org/
- IEEE Xplore: https://ieeexplore.ieee.org/
- OpenReview: https://openreview.net/
- Papers With Code: https://paperswithcode.com/
- ACM Digital Library: https://dl.acm.org/

**Top Venue Proceedings:**
- ICRA: https://ieeexplore.ieee.org/xpl/conhome/1000639/all-proceedings
- RSS: https://roboticsconference.org/
- CoRL: https://www.corl.org/
- NeurIPS: https://papers.nips.cc/
- ICML / ICLR: https://openreview.net/

**Citation Styles:**
- IEEE Style: https://ieeeauthorcenter.ieee.org/wp-content/uploads/IEEE-Reference-Guide.pdf
- APA Style: https://apastyle.apa.org/

## Dependencies

```bash
pip install requests          # For citation verification
brew install pandoc           # macOS — PDF generation
apt-get install pandoc        # Linux — PDF generation
brew install --cask mactex    # macOS — LaTeX
apt-get install texlive-xetex # Linux — LaTeX
```

Check dependencies: `python scripts/generate_pdf.py --check-deps`
