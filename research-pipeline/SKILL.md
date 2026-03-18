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
The paper will be read later, phase by phase, as needed.

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

IMPORTANT: Read each phase file ONLY when you reach that phase.
Do NOT pre-read multiple files. Do NOT read the full paper upfront.
Each phase will tell you what parts of the paper to read.

After completing each phase, say "Phase N complete. Starting Phase N+1."
Then immediately proceed to the next phase.

### Phase 1: Scout
1. Read `phases/scout.md`
2. Now read the paper (abstract, intro, related work, evaluation sections)
3. Execute the scout instructions: use WebSearch to find missing baselines, verify citations, check novelty
4. Save output as the Round Briefing
5. Say "Phase 1 complete. Starting Phase 2."

### Phase 2: Inspect
1. Read `phases/inspect.md`
2. Read the paper sections needed for format/storyline checks
3. Run the inspect and storyline checklists
4. Append results to the Round Briefing
5. Say "Phase 2 complete. Starting Phase 3."

### Phase 3: Panel
1. Read `phases/panel.md`
2. Write Round Briefing to `DISCUSSION_THREAD.md` as Wave 0
3. Read `discussion-panel/SKILL.md` for orchestration instructions
4. Spawn all 6 agents in parallel (Wave 1), wait, merge results
5. Spawn all 6 agents again (Wave 2), wait, merge results
6. Extract: average score, AUTHOR action list
7. Say "Phase 3 complete. Starting Phase 4."

### Phase 4: Experiments (conditional)
1. Check AUTHOR action list for items marked "requires new experiment"
2. If none: say "No experiments needed. Skipping to Phase 5."
3. If yes: read `phases/experiments.md`, design and verify experiments

### Phase 5: Write
1. Read `phases/write.md`
2. Implement all text changes from the action list
3. Log every change to `ROUND_STATE.md`
4. Say "Phase 5 complete. Starting Phase 6."

### Phase 6: Humanize
1. Read `phases/humanize.md`
2. Run humanize on all sections modified in Phase 5
3. Verify: grep for em dashes returns zero in modified sections
4. Say "Phase 6 complete. Starting Phase 7."

### Phase 7: External Review
1. Read `phases/review.md`
2. If Codex MCP is available: send paper for blind review
3. If unavailable: note "External review skipped" in `ROUND_STATE.md`
4. Say "Phase 7 complete. Starting Phase 8."

### Phase 8: Summary
1. Read `phases/summary.md`
2. Write round summary to `ROUND_STATE.md`
3. Present summary to user
4. Check stopping conditions

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
