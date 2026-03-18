---
name: research-pipeline
description: |
  Full autonomous paper improvement pipeline. Invoke as /papr pipeline start [N]
  [paper_dir] to run N rounds (default 3) on a LaTeX project folder. Each round:
  scout, inspect, panel, experiments, write, humanize, external review, summary.
  Stops early at score 9+.
allowed-tools:
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - Bash
  - Agent
  - WebSearch
  - WebFetch
  - mcp__codex__codex
  - mcp__codex__codex-reply
---

# Research Pipeline

End-to-end autonomous paper improvement. Default: 3 rounds.
Each round ends with a score /10 and a summary of what changed.
The human is only consulted between rounds (to approve continuing) and for ESCALATE issues.

---

## Subcommands

| Invocation | Behavior |
|---|---|
| `/papr pipeline start [N] [paper_dir]` | Start pipeline. N = rounds (default 3). paper_dir = LaTeX project folder (e.g. `paper/Vivar`). If paper_dir omitted, ask user. |
| `/papr pipeline round` | Run the next full round (reads current state) |
| `/papr pipeline status` | Print `ROUND_STATE.md` summary and current score |
| `/papr pipeline scout` | Run Phase 1 only |
| `/papr pipeline inspect` | Run Phase 2 only |
| `/papr pipeline panel` | Run Phase 3 only |
| `/papr pipeline experiments` | Run Phase 4 only (if experiments needed) |
| `/papr pipeline write` | Run Phase 5 only |
| `/papr pipeline humanize` | Run Phase 6 only |
| `/papr pipeline review` | Run Phase 7 only |
| `/papr pipeline summary` | Run Phase 8 only |

---

## Initialization (`start`)

Do exactly these steps. Do NOT read any other files. Do NOT run `ls` or `head` or verify anything extra.

1. If `paper_dir` was provided, use it. Otherwise ask: "Which folder is your LaTeX project?"
2. Find the main .tex file: Glob for `[paper_dir]/*.tex` (NOT recursive). Pick the one named `main.tex` (or the first result if no main.tex).
3. If `[paper_dir]/ROUND_STATE.md` or `[paper_dir]/DISCUSSION_THREAD.md` already exist, read them first (required before overwriting).
4. Write `[paper_dir]/ROUND_STATE.md`:

```markdown
# Research Pipeline State
## Paper directory: [paper_dir]
## Main file: [main.tex path]
## Rounds planned: [N]
## Started: [timestamp]
## Rounds completed: 0
## Score history: []
```

5. Write `[paper_dir]/DISCUSSION_THREAD.md` with empty content.
6. Proceed immediately to Round 1 — read `phases/scout.md` and execute it.

---

## Per-Round Phases

Run in order. Read ONLY the current phase file — do NOT pre-read later phases or any other files.

| Phase | File | What it does |
|---|---|---|
| 1. Scout | `phases/scout.md` | Find missing baselines, verify citations, check novelty |
| 2. Inspect | `phases/inspect.md` | Format + storyline audit |
| 3. Panel | `phases/panel.md` | Parallel multi-agent review (6 agents, 2 waves) |
| 4. Experiments | `phases/experiments.md` | Design + verify experiments (conditional) |
| 5. Write | `phases/write.md` | Implement text changes |
| 6. Humanize | `phases/humanize.md` | Remove AI writing patterns |
| 7. External Review | `phases/review.md` | Blind review via Codex MCP |
| 8. Summary | `phases/summary.md` | Score, log changes, present to user |

---

## Round Continuity

**Round 2+ only:** Before Phase 1, prepend the previous round's summary from
`ROUND_STATE.md` to the context. All phases should avoid re-raising issues
already marked resolved in that summary.

---

## Stopping Conditions

Stop early if:
- Score reaches 9+ -- paper is ready
- `SIGNAL: ESCALATE` unresolved after external review -- surface to human
- User invokes `/papr pipeline status` and declines to continue

---

## Score Reference

| Score | Interpretation |
|---|---|
| 9-10 | Strong accept, ready for top venue |
| 7-8 | Weak accept, minor revisions |
| 5-6 | Borderline, significant work remains |
| 3-4 | Weak reject, major revision needed |
| 1-2 | Reject, fundamental issues |
