---
name: papr-write
description: |
  Implement text changes to a paper and remove AI writing patterns. Takes an
  action list from papr-panel and edits the LaTeX source. Then invokes
  /humanizer on modified sections, then compiles PDF. Invoke as /papr-write [paper_dir].
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

Three phases: edit text, humanize, compile PDF.

## On invocation

1. Read `.claude/latest-run/latest/ROUND_STATE.md` for the action list.
2. Read `.claude/latest-run/latest/DISCUSSION_THREAD.md` for context if needed.
3. Read the paper's .tex files in `[paper_dir]` that need editing.
4. Apply changes (Phase A), then humanize (Phase B), then compile (Phase C).

---

## Phase A: Write

Apply changes in this priority order (to avoid line-number conflicts):
1. Structural -- section reordering, merging, splitting
2. Content -- paragraph rewrites, additions, removals, experiment results
3. Local -- term definitions, citation fixes, sentence repairs
4. Style -- em dashes, author name overuse, caption length

### Writing Rules (always apply)
- No em dashes -- use commas, semicolons, colons, or parentheses
- No rhetorical questions -- state findings directly
- No "a reviewer might ask" -- never
- No inline author citation overuse -- prefer \cite{} unless opening a related-work paragraph
- Define every new term at first use, not in a footnote
- Captions shorter than the body paragraph referencing the figure
- Contributions list: <=3 items, each verifiable

### Special Cases
- **Undefined terms:** Add definition at first use. If needs citation, use web search.
- **Rebuttal language:** Delete "as a reviewer noted" etc. Rewrite as if always in paper.
- **Merging subsections:** Unified topic sentence, integrate, remove second heading.
- **Caption shortening:** Captions describe *what is shown*. Move interpretation to body.

### Figure Style (when generating or editing figures)
- Use a consistent color palette (e.g. seaborn, matplotlib default, or venue-specific)
- NO titles on figures; the caption provides the title
- Legends must NOT overlay on data (curves, bars, etc.); place outside or in empty space
- Legend font size should match body text after embedding (~9-10pt)
- Use hatching patterns on bars/areas for B&W printing readability
- Axis labels and tick labels must be readable at column width

### Information Density (conference space is limited)
- Do NOT just translate a table into a bar plot; that wastes space
- Figures should show relationships, trends, or visual patterns that tables cannot
- Good: scatter with trends, heatmaps, qualitative comparisons, ablation curves, attention maps
- Bad: simple bar plots restating table numbers
- If a result is extraordinary, visualize it to make the impact obvious
- Combine related results into multi-panel figures rather than separate figures
- No duplicate information between figures and tables

### Reference Verification (CRITICAL)
- NEVER write a \cite{} or bib entry from memory. Every reference MUST be verified.
- Use WebSearch to find the actual paper and its correct bibtex entry.
- Verify: title, authors, venue, year all match the real publication.
- If a reference cannot be found via web search, flag it and do NOT include it.
- No hallucinated references. This is a hard rule.

### Depth Over Breadth
- Top venues value deep analysis of a phenomenon, not many shallow observations.
- When writing results/analysis, focus on insightful findings and deep explanations.
- Prefer 2-3 deeply analyzed results over 10 superficial statistical observations.
- Every claim should lead to a "why" or "what does this tell us" discussion.

Log every change to `.claude/latest-run/latest/ROUND_STATE.md` under `## Changes this round`.

---

## Phase B: Humanize

After ALL text edits complete, invoke `/humanizer` on each modified section.

The `/humanizer` skill handles 24 AI writing patterns including inflated significance,
promotional language, em dash overuse, AI vocabulary, and more. It must be installed
separately (see README).

After humanizing, use Grep to confirm no em dashes remain in modified sections.

---

## Phase C: Compile PDF

After humanizing, compile the paper so reviewers see the final rendered output.

```bash
cd [paper_dir] && latexmk -pdf -interaction=nonstopmode main.tex 2>&1 | tail -20
```

If compilation fails, check the error output. Fix LaTeX errors in the .tex files
(missing braces, undefined commands, etc.) and retry. Do NOT skip compilation.

After successful compilation, verify:
```bash
pdfinfo [paper_dir]/main.pdf | head -5
```

---

## After completion

Append to `.claude/latest-run/latest/ROUND_STATE.md`:
```
### Write + Humanize + Compile pass
Sections modified: [list]
Changes applied: [N]
Em dashes removed: [N]
AI patterns fixed: [N]
PDF compiled: [YES/NO]
```
