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

**Semantic Scholar** (always include — free, no key needed):
```
GET https://api.semanticscholar.org/graph/v1/paper/search?query=<terms>&fields=title,authors,year,citationCount,externalIds,abstract&limit=10
```

**arXiv** (primary preprint server for CS, ML, robotics):
```
GET https://export.arxiv.org/api/query?search_query=<terms>&max_results=10&sortBy=relevance
```

Relevant arXiv categories:
- `cs.RO` — Robotics
- `cs.LG` — Machine Learning
- `cs.AI` — Artificial Intelligence
- `cs.CV` — Computer Vision
- `cs.NE` — Neural and Evolutionary Computing
- `stat.ML` — Machine Learning (Statistics)

**ACM Digital Library** (for HRI, HCI, systems):
```
GET https://dl.acm.org/action/doSearch?query=<terms>&pageSize=10
```

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

Do **not** retry the request. Move on immediately to the next database.

At the end of Step 3, include a **Search Summary** section before the references:

```
### Search Summary
- Semantic Scholar: ✓ 10 results
- arXiv:           ✗ HTTP 429 (rate limited) — skipped
- ACM DL:          ✓ 6 results
```

If all databases were rejected, report this clearly and return no references rather than fabricating results.

### Step 3: Select and Return References
From the combined results, pick the **3-10 most relevant** papers. Prefer:
- Higher citation counts (see thresholds in Full Review section)
- Papers whose title/abstract directly address the user's question
- Recent papers (last 5 years) unless the question asks for foundational work
- Papers with available code (check Papers With Code)

For each selected paper, provide:
- Full citation (authors, year, title, venue)
- DOI or arXiv ID
- 1-sentence explanation of why it supports the statement/approach

Do **not** run deduplication scripts, generate PDFs, or create flow diagrams for simple reviews.

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

1. **Multi-Database Search**:

   **Primary Sources (always search):**
   - Search arXiv via API — dominant preprint venue for CS/ML/robotics
   - Search Semantic Scholar via API — 200M+ papers, cross-disciplinary
   - Search IEEE Xplore — ICRA, IROS, RA-L, T-RO, and other robotics venues
   - Search OpenReview — NeurIPS, ICML, ICLR, CoRL submissions and reviews

   **Secondary Sources (search when relevant):**
   - ACM Digital Library — HRI, CHI, RSS proceedings
   - Papers With Code — benchmark results and reproducibility info
   - Google Scholar — broad coverage including workshop papers

2. **Document Search Parameters**:
   ```markdown
   ## Search Strategy

   ### Database: arXiv
   - **Date searched**: 2025-04-08
   - **Date range**: 2020-01-01 to 2025-04-08
   - **Search string**:
     ```
     cat:cs.RO AND (ti:"dexterous manipulation" OR ti:"robotic hand")
     AND (abs:"reinforcement learning" OR abs:"sim-to-real")
     ```
   - **Status**: ✓ 183 articles  (or ✗ HTTP 429 — skipped)
   ```

   Repeat for each database searched. If a database returns HTTP 429, 403, or 5xx, log it as skipped and continue to the next. Do not retry. Include all skipped databases in a **Search Summary** table in the Methodology section:

   | Database | Status | Results |
   |----------|--------|---------|
   | arXiv | ✓ | 183 |
   | Semantic Scholar | ✓ | 247 |
   | IEEE Xplore | ✗ HTTP 429 (rate limited) | — |
   | OpenReview | ✓ | 94 |

3. **Export and Aggregate Results**:
   - Export results in JSON format from each database
   - Combine all results into a single file
   - Use `scripts/search_databases.py` for post-processing:
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

2. **Title Screening**:
   - Review all titles against inclusion/exclusion criteria
   - Exclude obviously irrelevant studies
   - Document number excluded at this stage

3. **Abstract Screening**:
   - Read abstracts of remaining studies
   - Apply inclusion/exclusion criteria rigorously
   - Document reasons for exclusion

4. **Full-Text Screening**:
   - Obtain full texts of remaining studies
   - Conduct detailed review against all criteria
   - Document specific reasons for exclusion
   - Record final number of included studies

5. **Create Search Flow Diagram**:
   ```
   Initial search: n = X
   ├─ After deduplication: n = Y
   ├─ After title screening: n = Z
   ├─ After abstract screening: n = A
   └─ Included in review: n = B
   ```

### Phase 4: Data Extraction and Quality Assessment

1. **Extract Key Data** from each included study:
   - Paper metadata (authors, year, venue, DOI/arXiv ID)
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
   - Note if code is publicly available (check Papers With Code)
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
   - Note areas requiring future research

4. **Write Discussion**:
   - Interpret findings in broader research context
   - Discuss practical implications for deployment or future research
   - Acknowledge limitations of the review itself
   - Compare with previous surveys if applicable
   - Propose specific future research directions

### Phase 6: Citation Verification

**CRITICAL**: All citations must be verified for accuracy before final submission.

1. **Verify All DOIs**:
   ```bash
   python scripts/verify_citations.py my_literature_review.md
   ```

   This script:
   - Extracts all DOIs from the document
   - Verifies each DOI resolves correctly
   - Retrieves metadata from CrossRef
   - Generates verification report
   - Outputs properly formatted citations

2. **Review Verification Report**:
   - Check for any failed DOIs
   - Verify author names, titles, and publication details match
   - Correct any errors in the original document
   - Re-run verification until all citations pass

3. **Format Citations Consistently**:
   - Choose one citation style and use throughout (see `references/citation_styles.md`)
   - Common styles for robotics/AI: IEEE, APA, ACM
   - Use verification script output to format citations correctly
   - Ensure in-text citations match reference list format

### Phase 7: Document Generation

1. **Generate PDF**:
   ```bash
   python scripts/generate_pdf.py my_literature_review.md \
     --citation-style ieee \
     --output my_review.pdf
   ```

   Options:
   - `--citation-style`: apa, ieee, acm, chicago
   - `--no-toc`: Disable table of contents
   - `--no-numbers`: Disable section numbering
   - `--check-deps`: Check if pandoc/xelatex are installed

2. **Review Final Output**:
   - Check PDF formatting and layout
   - Verify all sections are present
   - Ensure citations render correctly
   - Check that figures/tables appear properly
   - Verify table of contents is accurate

3. **Quality Checklist**:
   - [ ] All DOIs verified with verify_citations.py
   - [ ] Citations formatted consistently
   - [ ] Search flow diagram included
   - [ ] Search methodology fully documented
   - [ ] Inclusion/exclusion criteria clearly stated
   - [ ] Results organized thematically (not paper-by-paper)
   - [ ] Quality assessment completed
   - [ ] Limitations acknowledged
   - [ ] References complete and accurate
   - [ ] PDF generates without errors

## Database-Specific Search Guidance

### arXiv

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

### Semantic Scholar

Access via free API (200M+ papers, cross-disciplinary):
```
GET https://api.semanticscholar.org/graph/v1/paper/search?query=<terms>&fields=title,authors,year,citationCount,externalIds,abstract&limit=20
```

**Search tips**:
- Use `fields=` to request citation counts — essential for ranking by impact
- Retrieve recommendations for a known key paper: `/paper/{id}/recommendations`
- Use `/paper/{id}/citations` and `/paper/{id}/references` for citation chaining

### IEEE Xplore

Covers ICRA, IROS, RA-L, T-RO, and other top robotics/systems venues:
```
GET https://ieeexploreapi.ieee.org/api/v1/search/articles?querytext=<terms>&max_records=25&sort_order=desc&sort_field=article_citation_count
```
Requires a free API key from IEEE.

### OpenReview

Covers NeurIPS, ICML, ICLR, CoRL, and many ML/robotics workshops:
```
GET https://api2.openreview.net/notes?content.title=<terms>&details=replyCount&limit=25
```

**Notes**: Includes submitted, under-review, and rejected papers. Filter by `venueid` for specific conferences.

### Papers With Code

Find papers with open-source implementations and benchmark leaderboards:
```
GET https://paperswithcode.com/api/v1/papers/?q=<terms>&items_per_page=10
```

Use to verify code availability and compare results on standard benchmarks (e.g., MuJoCo locomotion, D4RL, Isaac Gym).

### Citation Chaining

Expand search via citation networks:

1. **Forward citations** (papers citing key papers):
   - Use Semantic Scholar `/paper/{id}/citations`
   - Identifies newer work building on seminal methods

2. **Backward citations** (references from key papers):
   - Use Semantic Scholar `/paper/{id}/references`
   - Identifies foundational and closely related work

## Citation Style Guide

Detailed formatting guidelines are in `references/citation_styles.md`. Quick reference:

### IEEE
- In-text: Bracketed numbers [1], [2], [1]–[3]
- Reference: J. D. Smith, M. L. Johnson, and K. R. Williams, "Title," in *Proc. ICRA*, 2023, pp. 301–318. doi: 10.xxx/yyy

### APA (7th Edition)
- In-text: (Smith et al., 2023)
- Reference: Smith, J. D., Johnson, M. L., & Williams, K. R. (2023). Title. *Proc. ICRA*, 301–318. https://doi.org/10.xxx/yyy

**Always verify citations** with verify_citations.py before finalizing.

### Prioritizing High-Impact Papers (CRITICAL)

**Always prioritize influential, highly-cited papers from reputable authors and top venues.** Quality matters more than quantity in literature reviews.

#### Citation Count Thresholds

Use citation counts to identify the most impactful papers:

| Paper Age | Citation Threshold | Classification |
|-----------|-------------------|----------------|
| 0-3 years | 20+ citations | Noteworthy |
| 0-3 years | 100+ citations | Highly Influential |
| 3-7 years | 100+ citations | Significant |
| 3-7 years | 500+ citations | Landmark Paper |
| 7+ years | 500+ citations | Seminal Work |
| 7+ years | 1000+ citations | Foundational |

#### Conference and Journal Tiers

Prioritize papers from higher-tier venues:

- **Tier 1 (Always Prefer):** NeurIPS, ICML, ICLR, CVPR, ICRA, RSS, CoRL, Science Robotics, IJRR, T-RO, RA-L
- **Tier 2 (Strong Preference):** IROS, ECCV, ICCV, AISTATS, UAI, AAAI, IJCAI, Autonomous Robots
- **Tier 3 (Include When Relevant):** Workshops at Tier-1 venues, respected specialized journals
- **Tier 4 (Use Sparingly):** Lower-impact peer-reviewed venues, non-peer-reviewed technical reports

#### Author Reputation Assessment

Prefer papers from:
- **Senior researchers** with high h-index (>30 in established subfields)
- **Leading research groups** at recognized institutions and labs (DeepMind, OpenAI, CMU RI, MIT CSAIL, Stanford AI Lab, Berkeley AI Research, ETH Zurich, etc.)
- **Authors with multiple Tier-1 publications** in the relevant field
- **Researchers with recognized expertise** (awards, editorial positions, distinguished speakers)

#### Identifying Seminal Papers

For any topic, identify foundational work by:
1. **High citation count** (typically 500+ for papers 5+ years old)
2. **Frequently cited by other included papers** (appears in many reference lists)
3. **Published in Tier-1 venues** (NeurIPS, ICRA, RSS, etc.)
4. **Written by field pioneers** (often cited as establishing the concept or method)

## Best Practices

### Search Strategy
1. **Use multiple databases** (minimum 3): Ensures comprehensive coverage across venues
2. **Include arXiv**: Captures latest preprints, often months ahead of conference proceedings
3. **Document everything**: Search strings, dates, result counts for reproducibility
4. **Test and refine**: Run pilot searches, review results, adjust search terms
5. **Sort by citations**: Surface influential work first; also check recent papers with fewer citations

### Screening and Selection
1. **Use clear criteria**: Document inclusion/exclusion criteria before screening
2. **Screen systematically**: Title → Abstract → Full text
3. **Document exclusions**: Record reasons for excluding papers
4. **Prioritize reproducible work**: Prefer papers with public code and standard benchmarks

### Synthesis
1. **Organize thematically**: Group by themes or method families, NOT by individual papers
2. **Synthesize across papers**: Compare, contrast, identify patterns and trends
3. **Be critical**: Evaluate reproducibility, benchmark fairness, and real-world applicability
4. **Identify gaps**: Note what's missing, understudied, or contradicted by recent work

### Quality and Reproducibility
1. **Check code availability**: Use Papers With Code; note if results are reproducible
2. **Verify all citations**: Run verify_citations.py script
3. **Document methodology**: Provide enough detail for others to reproduce the search
4. **Compare on shared benchmarks**: Prefer papers that evaluate on standard tasks

### Writing
1. **Be objective**: Present evidence fairly, acknowledge limitations
2. **Be systematic**: Follow structured template
3. **Be specific**: Include performance numbers, benchmark names, and dataset sizes where available
4. **Be clear**: Use clear headings, logical flow, thematic organization

## Common Pitfalls to Avoid

1. **Single database search**: Misses relevant papers; always search multiple databases
2. **No search documentation**: Makes review irreproducible; document all searches
3. **Paper-by-paper summary**: Lacks synthesis; organize thematically instead
4. **Unverified citations**: Leads to errors; always run verify_citations.py
5. **Too broad search**: Yields thousands of irrelevant results; refine with specific terms
6. **Too narrow search**: Misses relevant papers; include synonyms and related terms
7. **Ignoring arXiv**: Misses latest findings; robotics/AI moves faster than journal cycles
8. **Ignoring benchmark context**: Treats all claims equally; compare on shared benchmarks
9. **Recency bias**: Newer isn't always better; include seminal foundational work
10. **Outdated search**: Field evolves rapidly; clearly state search date

## Example Workflow

Complete workflow for a robotics/AI literature review:

```bash
# 1. Create review document from template
cp assets/review_template.md sim_to_real_rl_review.md

# 2. Search multiple databases
# - arXiv: cat:cs.RO AND (abs:"sim-to-real" OR abs:"domain randomization")
# - Semantic Scholar: "sim-to-real transfer reinforcement learning robotics"
# - IEEE Xplore: "sim-to-real" AND "robot"
# - OpenReview: search CoRL and ICML venues
# - Export results in JSON format

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
# - Manually screen titles, abstracts, full texts
# - Extract key data into the review document
# - Organize by themes (domain randomization, system ID, adaptive methods, etc.)

# 5. Write the review following template structure
# - Introduction with clear objectives
# - Detailed methodology section
# - Results organized thematically
# - Critical discussion
# - Clear conclusions

# 6. Verify all citations
python scripts/verify_citations.py sim_to_real_rl_review.md

# Review the citation report
cat sim_to_real_rl_review_citation_report.json

# Fix any failed citations and re-verify
python scripts/verify_citations.py sim_to_real_rl_review.md

# 7. Generate professional PDF
python scripts/generate_pdf.py sim_to_real_rl_review.md \
  --citation-style ieee \
  --output sim_to_real_rl_review.pdf

# 8. Review final PDF and markdown outputs
```

## Integration with Other Skills

This skill works with other scientific and writing skills:

### Visualization Skills
- **matplotlib**: Generate comparison figures, benchmark plots
- **seaborn**: Statistical visualizations for performance comparisons

### Writing Skills
- **brand-guidelines**: Apply institutional branding to PDF
- **internal-comms**: Adapt review for different audiences

## Resources

### Bundled Resources

**Scripts:**
- `scripts/verify_citations.py`: Verify DOIs and generate formatted citations
- `scripts/generate_pdf.py`: Convert markdown to professional PDF
- `scripts/search_databases.py`: Process, deduplicate, and format search results

**References:**
- `references/citation_styles.md`: Detailed citation formatting guide (APA, IEEE, ACM)
- `references/database_strategies.md`: Comprehensive database search strategies

**Assets:**
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

### Required Python Packages
```bash
pip install requests  # For citation verification
```

### Required System Tools
```bash
# For PDF generation
brew install pandoc  # macOS
apt-get install pandoc  # Linux

# For LaTeX (PDF generation)
brew install --cask mactex  # macOS
apt-get install texlive-xetex  # Linux
```

Check dependencies:
```bash
python scripts/generate_pdf.py --check-deps
```

## Summary

This literature-review skill provides:

1. **Systematic methodology** following academic best practices for robotics and AI
2. **Multi-database integration** covering arXiv, Semantic Scholar, IEEE Xplore, OpenReview, and Papers With Code
3. **Citation verification** ensuring accuracy and credibility
4. **Professional output** in markdown and PDF formats
5. **Comprehensive guidance** covering the entire review process
6. **Quality assurance** with reproducibility checks and benchmark-aware evaluation
7. **Reproducibility** through detailed documentation requirements

Conduct thorough, rigorous literature reviews that meet the standards of top robotics and AI venues.
