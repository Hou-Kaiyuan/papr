---
name: papr-pipeline
description: |
  Autonomous multi-round paper improvement pipeline. Chains /papr-panel,
  /papr-experiment, /papr-write, and /papr-review in sequence. Each round
  produces a score out of 10. Stops early at 9+. Invoke as
  /papr-pipeline [N] [paper_dir] where N is rounds (default 3).
allowed-tools:
  - Read
  - Write
  - Glob
  - Bash
  - Agent
---

# Paper Improvement Pipeline

Chains other papr skills in sequence. Each round:
1. `/papr-panel` -- multi-agent review, produces action list + score
2. `/papr-experiment` -- design experiments if needed (conditional)
3. `/papr-write` -- implement text changes + remove AI writing patterns
4. `/papr-review` -- external blind review via Codex MCP

## On invocation

IMPORTANT: Do NOT explore the filesystem. Do NOT run `ls`. Do NOT search with multiple patterns. Follow these exact steps:

1. The user provides `paper_dir` as an argument (e.g. `Gauge_Paper`). If not given, ask: "Which folder is your LaTeX project?" and wait for answer. Do NOT guess or search.
2. Read `[paper_dir]/ROUND_STATE.md` and `[paper_dir]/DISCUSSION_THREAD.md` if they exist (use Read tool, it will error if not found -- that is fine, ignore the error).
3. Write `[paper_dir]/ROUND_STATE.md`:
```
# Pipeline State
## Paper: [paper_dir]
## Rounds: [N, default 3]
## Started: [date]
## Completed: 0
## Scores: []
```
4. Write `[paper_dir]/DISCUSSION_THREAD.md` with empty content.
5. Start the round loop immediately.

## Round loop

```
for round in 1..N:
    invoke /papr-panel [paper_dir]
    read ROUND_STATE.md for action list
    if action list has "requires new experiment":
        invoke /papr-experiment [paper_dir]
    invoke /papr-write [paper_dir]
    invoke /papr-review [paper_dir]
    read ROUND_STATE.md for score
    update ROUND_STATE.md (increment completed, append score)
    if score >= 9: stop, say "Paper ready."
    if round < N: ask user "Ready for round [next]?"
```

To invoke each skill, use the Skill tool: `skill: "papr-panel", args: "[paper_dir]"`.

## On `status`

Read `[paper_dir]/ROUND_STATE.md` and print summary.

## On `round`

Read `ROUND_STATE.md`, run one iteration of the round loop.
