---
name: research-pipeline
description: |
  Full autonomous paper improvement pipeline. Invoke as /papr pipeline start [N]
  to run N rounds (default 3). Each round: scout, inspect, panel, experiments,
  write, humanize, external review, summary. Stops early at score 9+.
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

End-to-end autonomous paper improvement.

---

## On `start [N]` — DO THIS IMMEDIATELY:

**Step 1: Find the paper path (DO NOT read the paper yet).**
Use Glob to search for `**/*.tex` and `**/*.pdf` in the working directory.
If multiple .tex files found, pick the main one (the one with `\documentclass`).
If none found, ask the user for the paper path and stop.
IMPORTANT: Only note the file path. Do NOT read the paper contents yet.

**Step 2: Create state files.**
Write `ROUND_STATE.md`:
```markdown
# Research Pipeline State
## Paper: [filename]
## Rounds planned: [N, default 3]
## Started: [timestamp]
## Rounds completed: 0
## Score history: []
```
Write an empty `DISCUSSION_THREAD.md`.

**Step 3: Start Round 1.**
Go to "Run a Round" below and execute Phase 1 immediately.

---

## On `round` — run the next round:

Read `ROUND_STATE.md` to get current state. Go to "Run a Round" below.

---

## On `status` — print current state:

Read `ROUND_STATE.md` and print a summary. Ask user if they want to continue.

---

## Run a Round

Execute these 8 phases IN ORDER. Do not skip ahead.
Do NOT read any other files unless explicitly told to below.
After each phase, say "Phase N complete." and immediately start the next.

### Phase 1: Scout

Read the paper's abstract, introduction, related work, and evaluation sections.
Then use WebSearch to check three things:

**A. State of the art** — search `[task] state of the art 2024 2025` and `[task] benchmark leaderboard`. Flag top methods not compared against.

**B. Missing baselines** — search `[method category] alternatives comparison`. Flag methods that should be baselines but aren't.

**C. Novelty check** — search `[core contribution] similar prior work`. Flag papers that could undermine novelty claims.

Also verify key citations exist and claims match the cited papers.

Save results as the Round Briefing:
```
## Scout Report — Round [N]
### Missing baselines
| Method | Venue/Year | Performance | URL |
### Missing citations
- [title] — relevant because [reason]
### Novelty concerns
- [paper]: similar because [description]
### Actions
- [finding] → [route to: experiments | write | discussion-panel]
```

Say "Phase 1 complete." then start Phase 2.

### Phase 2: Inspect

Read the full paper. Check for format and storyline issues:

**Format checks:** figures (sizing, legends, readability), tables (arrows, bolding, width), citations (resolved refs, consistent style), captions (shorter than body paragraph), notation (symbols defined at first use).

**Storyline checks:** abstract (problem + result quantified), introduction (motivation → gap → prior work → approach → contributions), method (no forward dependencies, assumptions supported), evaluation (experiments map to claims, baselines current).

Rate each issue as BLOCKER, MAJOR, or MINOR. Append to Round Briefing:
```
## Inspect Report — Round [N]
### Quality issues
BLOCKERS: [N] — [list]
MAJOR: [N] — [list]
MINOR: [N] — [list]
### Storyline verdict: COHERENT | PARTIALLY COHERENT | NEEDS MAJOR REVISION
### Transition failures
- [concept appears without introduction]
### Unsupported claims
- [claim without evidence]
```

Say "Phase 2 complete." then start Phase 3.

### Phase 3: Panel

Write Round Briefing into `DISCUSSION_THREAD.md` as a Wave 0 context block.
Then read `discussion-panel/SKILL.md` and follow its orchestration instructions
to run the 2-wave parallel panel. Do NOT read any other files until that file
tells you to.

Extract from the panel output: average score and AUTHOR action list.

Say "Phase 3 complete." then start Phase 4.

### Phase 4: Experiments (conditional)

Check the AUTHOR action list for items marked "requires new experiment".
If none exist, say "No experiments needed. Skipping to Phase 5."

If experiments needed, read `experiments.md` (the standalone skill in the
parent directory) and follow its instructions to design and verify experiments.

Say "Phase 4 complete." then start Phase 5.

### Phase 5: Write

Collect all inputs:
- AUTHOR action list (text-only items)
- Experiment results from Phase 4 (if any)
- BLOCKER + MAJOR quality issues from Phase 2

Implement changes to the paper in priority order:
1. Structural changes (section reorders, merges)
2. Content changes (rewrites, additions, removals)
3. Local fixes (terms, citations, captions)
4. Style fixes (em dashes, author names)

Follow these rules for ALL edits:
- No em dashes — use commas, semicolons, colons, or parentheses
- No rhetorical questions — state findings directly
- No "a reviewer might ask" — never
- Define every new term at first use
- Captions shorter than the body paragraph referencing the figure

Log every change to `ROUND_STATE.md` under `## Changes this round`.
Do NOT run humanizer here — that is Phase 6.

Say "Phase 5 complete." then start Phase 6.

### Phase 6: Humanize

Read the list of modified sections from `ROUND_STATE.md`.
For each modified section, read `humanize.md` (the standalone skill in the
parent directory) and apply it. Only process sections modified in Phase 5.

After each section, use Grep to confirm no em dashes remain.

Append to `ROUND_STATE.md`:
```
### Humanizer pass
Sections processed: [list]
Em dashes removed: [N]
Rhetorical questions converted: [N]
```

Say "Phase 6 complete." then start Phase 7.

### Phase 7: External Review

If Codex MCP is available (tools mcp__codex__codex and mcp__codex__codex-reply):
1. Read `external-review.md` for the review prompt template
2. Send the paper text to Codex (strip author info, no round/score metadata)
3. Append review to `ROUND_STATE.md`
4. Compare with internal panel: note new issues, confirmed issues, internal-only concerns

If Codex MCP is NOT available:
Note "External review skipped — Codex MCP not connected." in `ROUND_STATE.md`.

Say "Phase 7 complete." then start Phase 8.

### Phase 8: Summary

Collect from this round: panel scores, external review score, all changes, open issues.

Write to `ROUND_STATE.md`:
```markdown
## Round [N] Summary
### Scores
| Source | Score |
|---|---|
| Internal panel average | X/10 |
| External reviewer | Y/10 |
| Combined estimate | Z/10 |
### What changed this round
- [list of changes from Phase 5]
### Open issues
- [ ] [issue — source]
### Top remaining concerns
1. [concern]
2. [concern]
3. [concern]
```

Update rounds completed and score history in `ROUND_STATE.md`.

Present summary to user. Then check stopping conditions (see below).

---

## After Each Round

**Stop if:**
- Score reaches 9+ (paper is ready)
- `SIGNAL: ESCALATE` unresolved (surface to human)
- All planned rounds complete

**Continue if:**
- Rounds remain and score < 9
- Ask user: "Ready for Round [N+1]?"
- For Round 2+: read previous round summary before Phase 1.
  All phases should avoid re-raising resolved issues.

---

## On single phase commands (e.g. `scout`, `inspect`, etc.):

Read the corresponding standalone skill file and execute just that phase.

| Command | File to read |
|---|---|
| `scout` | `scout.md` (parent dir) |
| `inspect` | `inspect.md` (parent dir) |
| `panel` | `discussion-panel/SKILL.md` |
| `experiments` | `experiments.md` (parent dir) |
| `write` | `write.md` (parent dir) |
| `humanize` | `humanize.md` (parent dir) |
| `review` | `external-review.md` (parent dir) |
| `summary` | `phases/summary.md` |

---

## Score Reference

| Score | Interpretation |
|---|---|
| 9-10 | Strong accept, ready for top venue |
| 7-8 | Weak accept, minor revisions |
| 5-6 | Borderline, significant work remains |
| 3-4 | Weak reject, major revision needed |
| 1-2 | Reject, fundamental issues |
