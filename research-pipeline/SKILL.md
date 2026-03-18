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

Execute phases 1-8 in order.
CRITICAL: After reading paper files, IMMEDIATELY take the next action.
Do NOT pause, do NOT read additional files, do NOT ask what to do.

### Phase 1: Scout

Step 1a. Read ONLY the paper's abstract and introduction (2 files max).
Step 1b. From what you just read, extract: the task name, the method name, the key baselines compared against, and the main contribution claim. Write these down in your response.
Step 1c. IMMEDIATELY run WebSearch for: `[task name] state of the art 2024 2025`
Step 1d. Run WebSearch for: `[method name] [task name] baselines comparison`
Step 1e. Run WebSearch for: `[main contribution] similar prior work`
Step 1f. Write the Scout Report to a variable (do not save to file yet):
```
## Scout Report — Round [N]
### Missing baselines
| Method | Venue/Year | Performance | URL |
### Missing citations
- [title] -- relevant because [reason]
### Novelty concerns
- [paper]: similar because [description]
### Actions
- [finding] -> [route to: experiments | write | discussion-panel]
```
Step 1g. Say "Phase 1 complete." and IMMEDIATELY start Phase 2.

### Phase 2: Inspect

Step 2a. Read the paper's method and evaluation sections (you already have abstract+intro from Phase 1).
Step 2b. IMMEDIATELY list format issues found: figures, tables, citations, captions, notation.
Step 2c. IMMEDIATELY list storyline issues: logical gaps, unsupported claims, transition failures.
Step 2d. Rate each as BLOCKER/MAJOR/MINOR. Write the Inspect Report:
```
## Inspect Report — Round [N]
### Quality issues
BLOCKERS: [N] -- [list]
MAJOR: [N] -- [list]
MINOR: [N] -- [list]
### Storyline verdict: COHERENT | PARTIALLY COHERENT | NEEDS MAJOR REVISION
### Transition failures
- [concept appears without introduction]
### Unsupported claims
- [claim without evidence]
```
Step 2e. Say "Phase 2 complete." and IMMEDIATELY start Phase 3.

### Phase 3: Panel

Step 3a. Write the Scout Report + Inspect Report into `DISCUSSION_THREAD.md` as Wave 0.
Step 3b. Read `discussion-panel/SKILL.md` and follow its instructions to run the panel.
Step 3c. After panel completes, extract: average score and AUTHOR action list.
Step 3d. Say "Phase 3 complete." and IMMEDIATELY start Phase 4.

### Phase 4: Experiments (conditional)

Step 4a. Check the AUTHOR action list for "requires new experiment" items.
Step 4b. If none: say "No experiments needed. Skipping to Phase 5." and go to Phase 5.
Step 4c. If yes: read `experiments.md` (parent directory), design and verify experiments.
Step 4d. Say "Phase 4 complete." and IMMEDIATELY start Phase 5.

### Phase 5: Write

Step 5a. Collect: AUTHOR action list (text-only items), experiment results (if any), BLOCKER+MAJOR issues from Phase 2.
Step 5b. IMMEDIATELY start editing the paper. Priority order: structural, content, local, style.
Step 5c. Rules: no em dashes, no rhetorical questions, no "a reviewer might ask", define terms at first use, captions shorter than body paragraph.
Step 5d. Log every change to `ROUND_STATE.md` under `## Changes this round`.
Step 5e. Say "Phase 5 complete." and IMMEDIATELY start Phase 6.

### Phase 6: Humanize

Step 6a. Read modified sections list from `ROUND_STATE.md`.
Step 6b. For each modified section, read `humanize.md` (parent directory) and apply its 24 patterns. Only process modified sections.
Step 6c. Use Grep to confirm no em dashes remain in modified sections.
Step 6d. Append humanizer log to `ROUND_STATE.md`.
Step 6e. Say "Phase 6 complete." and IMMEDIATELY start Phase 7.

### Phase 7: External Review

Step 7a. If Codex MCP is NOT available: note "External review skipped" in `ROUND_STATE.md`, go to Phase 8.
Step 7b. If available: read `external-review.md` for the prompt template.
Step 7c. Send paper to Codex (strip author info, no round/score metadata).
Step 7d. Append review to `ROUND_STATE.md`. Compare with internal panel.
Step 7e. Say "Phase 7 complete." and IMMEDIATELY start Phase 8.

### Phase 8: Summary

Step 8a. Collect: panel scores, external review score, all changes, open issues.
Step 8b. Write round summary to `ROUND_STATE.md`:
```
## Round [N] Summary
### Scores
| Source | Score |
|---|---|
| Internal panel average | X/10 |
| External reviewer | Y/10 |
| Combined estimate | Z/10 |
### What changed this round
- [list]
### Open issues
- [ ] [issue]
### Top remaining concerns
1. [concern]
2. [concern]
3. [concern]
```
Step 8c. Update rounds completed and score history.
Step 8d. Present summary to user. Check stopping conditions below.

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
