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

**Step 1: Find the paper.**
Use Glob to search for `**/*.tex` and `**/*.pdf` in the working directory.
If multiple found, pick the main paper file (look for `\begin{document}` in .tex files).
If none found, ask the user for the paper path and stop.

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
Go to "Run a Round" below and execute it.

---

## On `round` — run the next round:

Read `ROUND_STATE.md` to get current state. Go to "Run a Round" below.

---

## On `status` — print current state:

Read `ROUND_STATE.md` and print a summary. Ask user if they want to continue.

---

## Run a Round

Execute these 8 phases IN ORDER. Do not skip ahead. Read each phase file
ONLY when you reach that phase, not before.

### Phase 1: Scout
Read `phases/scout.md`, then read the paper and execute the scout instructions.
Use WebSearch to find missing baselines, verify citations, check novelty.
Save output as the Round Briefing.

### Phase 2: Inspect
Read `phases/inspect.md`, then run the inspect and storyline checklists on the paper.
Append results to the Round Briefing.

### Phase 3: Panel
Read `phases/panel.md`, then run the discussion panel.
Write Round Briefing to `DISCUSSION_THREAD.md` as Wave 0.
Read `discussion-panel/SKILL.md` for orchestration instructions.
Spawn all 6 agents in parallel for Wave 1, merge results, then Wave 2, merge results.
Extract: average score, AUTHOR action list.

### Phase 4: Experiments (conditional)
Read `phases/experiments.md`. If the AUTHOR action list has items marked
"requires new experiment", design and verify experiments. Otherwise skip to Phase 5.

### Phase 5: Write
Read `phases/write.md`, then implement all text changes from the action list.
Log every change to `ROUND_STATE.md`.

### Phase 6: Humanize
Read `phases/humanize.md`, then run humanize on all sections modified in Phase 5.
Verify: `grep` for em dashes returns zero results in modified sections.

### Phase 7: External Review
Read `phases/review.md`. If Codex MCP is available, send paper for blind review.
If unavailable, note "External review skipped" in `ROUND_STATE.md` and continue.

### Phase 8: Summary
Read `phases/summary.md`, then write the round summary to `ROUND_STATE.md`.
Present summary to user. Check stopping conditions.

---

## After Each Round

Update `ROUND_STATE.md`: increment rounds completed, append score to history.

**Stop if:**
- Score reaches 9+ (paper is ready)
- `SIGNAL: ESCALATE` unresolved (surface to human)
- All planned rounds complete

**Continue if:**
- Rounds remain and score < 9
- Ask user: "Ready for Round [N+1]?"
- For Round 2+: read previous round summary from `ROUND_STATE.md` before Phase 1.
  All phases should avoid re-raising issues already resolved.

---

## On single phase commands (e.g. `scout`, `inspect`, etc.):

Read the corresponding phase file and execute just that phase on the paper.

| Command | Phase file |
|---|---|
| `scout` | `phases/scout.md` |
| `inspect` | `phases/inspect.md` |
| `panel` | `phases/panel.md` |
| `experiments` | `phases/experiments.md` |
| `write` | `phases/write.md` |
| `humanize` | `phases/humanize.md` |
| `review` | `phases/review.md` |
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
