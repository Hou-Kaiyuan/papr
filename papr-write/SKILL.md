---
name: papr-write
description: |
  Implement text changes to a LaTeX paper, remove AI writing patterns, and
  compile PDF. Use when the user says "apply changes", "fix the paper",
  "implement feedback", or after a review panel produces an action list.
  Invokes /humanizer on modified sections.
argument-hint: "[paper_dir]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - Bash
  - WebSearch
  - Agent
---

# Paper Writer + Humanizer + Compile

Three phases: A) edit text, B) humanize, C) compile PDF.

## On invocation

1. Read `.claude/latest-run/latest/ROUND_STATE.md` for action list.
2. Read `.claude/latest-run/latest/DISCUSSION_THREAD.md` for context.
3. Read the paper's .tex files that need editing.

## Phase A: Write

Priority order (avoids line-number conflicts):
1. Structural (section reordering, merging, splitting)
2. Content (paragraph rewrites, additions, experiment results)
3. Local (term definitions, citation fixes, sentence repairs)
4. Style (em dashes, author names, caption length)

### Rules
- No em dashes; use commas, semicolons, colons, or parentheses
- No rhetorical questions; state findings directly
- No "a reviewer might ask"; never
- Prefer `\cite{}` over inline "Author et al." unless opening a related-work paragraph
- Define every new term at first use
- Captions shorter than referencing body paragraph
- Contributions: <=3 items, each verifiable

### References (CRITICAL)
Every `\cite{}` and bib entry MUST be verified via WebSearch. Find the actual paper,
confirm title/authors/venue/year, and use the real bibtex. No hallucinated references.

### Figures
- Consistent color palette, NO titles (caption is the title)
- Legends outside data area, font size matching body text (~9-10pt)
- Hatching for B&W readability
- Do NOT make bar plots that just restate table numbers
- Figures should show what tables cannot: trends, patterns, qualitative comparisons
- Maximize information density (conference space is limited)
- No duplicate info between figures and tables

### Depth Over Breadth
- Prefer 2-3 deeply analyzed results over 10 shallow observations
- Every claim should lead to "why" or "what does this tell us"
- Do NOT add Spearman/Pearson correlation, Cohen's d, Wilson CI, bootstrap CI,
  t-tests, or p-values as a substitute for analysis. These are SHALLOW and
  ruin paper structure. Only include if absolutely central to a claim.

### Writing Style (model after top-tier papers)

Before writing or rewriting any major section (especially abstract, intro,
conclusion), use WebSearch to find 2-3 recent published papers from the same
venue (NeurIPS, ICLR, CVPR, etc.) on similar topics. Read how they:
- Structure the abstract (4-sentence pattern: problem -> gap -> approach -> result)
- Open the introduction (concrete motivation, not generic)
- Frame the contributions (specific and verifiable, not vague)
- Write the conclusion (synthesizes, does NOT just restate)

Match the tone, density, and structure of accepted papers. Do NOT generate
generic AI-style "this paper presents a novel framework for..." text.

Specific section guidance:
- **Abstract**: ~150-200 words, 4-5 sentences. Problem -> approach -> result
  with quantified numbers. No jargon, no method names without context.
- **Introduction**: 4 paragraphs typically. Real-world motivation, technical
  gap, prior work shortcomings, proposed approach. NO equations.
- **Conclusion**: Synthesizes contributions and points to future work.
  Does NOT restate the abstract. ~1 paragraph.

Log changes to `.claude/latest-run/latest/ROUND_STATE.md`.

## Phase B: Humanize

Invoke `/humanizer` on each modified section. Verify: `grep "—"` returns zero.

## Phase C: Compile PDF

```bash
cd [paper_dir] && latexmk -pdf -interaction=nonstopmode main.tex 2>&1 | tail -20
```

If compilation fails, fix LaTeX errors and retry. Verify with `pdfinfo [paper_dir]/main.pdf | head -5`.

## Output

Append to `.claude/latest-run/latest/ROUND_STATE.md`:
```
### Write + Humanize + Compile
Sections modified: [list]
Changes: [N], Em dashes removed: [N], AI patterns fixed: [N]
PDF compiled: [YES/NO]
```
