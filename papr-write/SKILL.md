---
name: papr-write
description: |
  Implement text changes to a paper and remove AI writing patterns. Takes an
  action list from papr-panel and edits the LaTeX source. Then runs a humanize
  pass on all modified sections. Invoke as /papr-write [paper_dir].
allowed-tools:
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - WebSearch
---

# Paper Writer + Humanizer

Implements text changes, then removes AI writing patterns. Two phases in one skill.

## On invocation

1. Read `[paper_dir]/ROUND_STATE.md` for the action list.
2. Read `[paper_dir]/DISCUSSION_THREAD.md` for context if needed.
3. Read the paper's .tex files that need editing.
4. Apply changes (Phase A), then humanize (Phase B).

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

Log every change to `[paper_dir]/ROUND_STATE.md` under `## Changes this round`.

---

## Phase B: Humanize

After ALL edits complete, run the humanize pass on modified sections only.

### Patterns to Remove

**Content:** inflated significance ("pivotal", "testament", "evolving landscape"), promotional language ("groundbreaking", "nestled"), vague attributions ("experts argue"), superficial -ing phrases ("highlighting", "showcasing"), formulaic challenges sections.

**Language:** AI vocabulary overuse ("Additionally", "crucial", "delve", "foster", "underscore"), copula avoidance ("serves as" -> "is"), negative parallelisms ("not just X, but Y"), rule-of-three forcing, synonym cycling, false ranges ("from X to Y").

**Style:** em dash overuse, mechanical boldface, inline-header bullet lists, title case in headings, emojis, curly quotes.

**Communication:** chatbot artifacts ("I hope this helps"), knowledge-cutoff disclaimers, sycophantic tone, filler phrases ("in order to", "it is important to note"), excessive hedging, generic positive conclusions.

### Process
1. For each modified section, scan for patterns above
2. Rewrite problematic sections preserving meaning
3. Self-test: "What makes this obviously AI generated?" Fix remaining tells.
4. Verify: grep for em dashes returns zero in modified sections

### After completion
Append to `ROUND_STATE.md`:
```
### Write + Humanize pass
Sections modified: [list]
Changes applied: [N]
Em dashes removed: [N]
AI patterns fixed: [N]
```
