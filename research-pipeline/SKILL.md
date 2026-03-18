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

Lightweight orchestrator. Each phase runs in its own Agent context.

---

## On `start [N] [paper_dir]`

1. If `paper_dir` not given, ask: "Which folder is your LaTeX project?"
2. Glob for `[paper_dir]/*.tex`. Note the main .tex file path.
3. If `[paper_dir]/ROUND_STATE.md` exists, read it first.
4. Write `[paper_dir]/ROUND_STATE.md`:
```
# Research Pipeline State
## Paper directory: [paper_dir]
## Main file: [main.tex path]
## Rounds planned: [N, default 3]
## Started: [timestamp]
## Current phase: 1
## Rounds completed: 0
## Score history: []
```
5. Write `[paper_dir]/DISCUSSION_THREAD.md` (empty, read first if exists).
6. Start the round loop below.

---

## Round Loop

For each round, run phases 1-8 by spawning an Agent for each phase.
Each Agent gets a fresh context and runs independently.

```
for round in 1..N:
    run_phase(1, "scout")
    run_phase(2, "inspect")
    run_phase(3, "panel")
    run_phase(4, "experiments")
    run_phase(5, "write")
    run_phase(6, "humanize")
    run_phase(7, "review")
    run_phase(8, "summary")
    read ROUND_STATE.md for score
    if score >= 9: stop
    ask user: "Ready for Round [N+1]?"
```

### How to run each phase

Spawn an Agent (foreground, NOT background) with this prompt:

```
You are running Phase [N] ([name]) of the PAPR research pipeline.

Paper directory: [paper_dir]
Main tex file: [main.tex path]
Round: [current round]

Read the phase instructions at: [skill_dir]/research-pipeline/phases/[phase].md
Read the paper files in: [paper_dir]/
Read the current state from: [paper_dir]/ROUND_STATE.md

Execute the phase instructions. Write all output to [paper_dir]/ROUND_STATE.md
and/or [paper_dir]/DISCUSSION_THREAD.md as instructed by the phase file.

When done, output a one-line summary of what you did.
```

Phase files:

| Phase | File | Skip condition |
|---|---|---|
| 1. Scout | `phases/scout.md` | never |
| 2. Inspect | `phases/inspect.md` | never |
| 3. Panel | `phases/panel.md` | never |
| 4. Experiments | `phases/experiments.md` | no "requires new experiment" in action list |
| 5. Write | `phases/write.md` | never |
| 6. Humanize | `phases/humanize.md` | never |
| 7. Review | `phases/review.md` | Codex MCP not available |
| 8. Summary | `phases/summary.md` | never |

After each Agent returns, update `Current phase` in ROUND_STATE.md and proceed.

---

## On `status`

Read `[paper_dir]/ROUND_STATE.md` and print summary.

## On `round`

Read ROUND_STATE.md, run the round loop for one round.

## On single phase (e.g. `scout`, `inspect`)

Spawn one Agent for that phase only.

---

## Stopping Conditions

- Score >= 9 -- paper is ready
- `SIGNAL: ESCALATE` unresolved -- surface to human
- All planned rounds complete

## Score Reference

| Score | Meaning |
|---|---|
| 9-10 | Strong accept |
| 7-8 | Weak accept |
| 5-6 | Borderline |
| 3-4 | Weak reject |
| 1-2 | Reject |
