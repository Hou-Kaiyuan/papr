---
name: papr-write
description: |
  Implement text changes to a paper and remove AI writing patterns. Takes an
  action list from papr-panel and edits the LaTeX source. Then invokes
  /humanizer on modified sections. Invoke as /papr-write [paper_dir].
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

# Paper Writer + Humanizer

Implements text changes, then invokes `/humanizer` to remove AI writing patterns.

## On invocation

1. Read `.claude/latest-run/latest/ROUND_STATE.md` for the action list.
2. Read `.claude/latest-run/latest/DISCUSSION_THREAD.md` for context if needed.
3. Read the paper's .tex files in `[paper_dir]` that need editing.
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

Log every change to `.claude/latest-run/latest/ROUND_STATE.md` under `## Changes this round`.

---

## Phase B: Humanize

After ALL edits complete, invoke `/humanizer` on each modified section.

The `/humanizer` skill handles 24 AI writing patterns including inflated significance,
promotional language, em dash overuse, AI vocabulary, and more. It must be installed
separately (see README).

After humanizing, use Grep to confirm no em dashes remain in modified sections.

Append to `.claude/latest-run/latest/ROUND_STATE.md`:
```
### Write + Humanize pass
Sections modified: [list]
Changes applied: [N]
Em dashes removed: [N]
AI patterns fixed: [N]
```
