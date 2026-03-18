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

# Paper Improvement Pipeline

Chains other papr skills in sequence. Each round:
1. `/papr-panel` -- multi-agent review, produces action list + score
2. `/papr-experiment` -- design experiments if needed (conditional)
3. `/papr-write` -- implement text changes + remove AI writing patterns
4. `/papr-review` -- external blind review via Codex MCP

## Output Directory

All generated files go in `.claude/papr-[MM-DD-HH]/`:
- `ROUND_STATE.md` -- pipeline state, scores, action lists (all rounds appended)
- `DISCUSSION_THREAD.md` -- panel discussion (overwritten each round for independent scoring)
- `wave1_*.md`, `wave2_*.md` -- temp agent outputs (deleted after merge)

One directory per pipeline run, NOT per round. A symlink `.claude/latest` points to it.

## On invocation

IMPORTANT: Do NOT explore the filesystem. Do NOT run `ls`. Do NOT search with multiple patterns. Follow these exact steps:

1. The user provides `paper_dir` as an argument (e.g. `Gauge_Paper`). If not given, ask: "Which folder is your LaTeX project?" and wait for answer. Do NOT guess or search.
2. Run: `mkdir -p .claude/papr-$(date +%m-%d-%H)` to create output directory.
3. Run: `ln -sfn papr-$(date +%m-%d-%H) .claude/latest` to set symlink.
4. Write `.claude/latest/ROUND_STATE.md`:
```
# Pipeline State
## Paper: [paper_dir]
## Rounds: [N, default 3]
## Started: [date]
## Completed: 0
## Scores: []
```
5. Write `.claude/latest/DISCUSSION_THREAD.md` with empty content.
6. Start the round loop immediately.

## Round loop

```
for round in 1..N:
    invoke /papr-panel [paper_dir]
    read .claude/latest/ROUND_STATE.md for action list
    if action list has "requires new experiment":
        invoke /papr-experiment [paper_dir]
    invoke /papr-write [paper_dir]
    invoke /papr-review [paper_dir]
    read .claude/latest/ROUND_STATE.md for score
    append "## Round [round] complete. Score: [X]/10" to ROUND_STATE.md
    if score >= 9: stop, say "Paper ready."
    if round < N: ask user "Ready for round [next]?"
```

To invoke each skill, use the Skill tool: `skill: "papr-panel", args: "[paper_dir]"`.

IMPORTANT: Do NOT create new directories between rounds. All rounds write to the same `.claude/latest/` directory. ROUND_STATE.md accumulates all rounds. DISCUSSION_THREAD.md is overwritten fresh by each panel invocation.

## On `status`

Read `.claude/latest/ROUND_STATE.md` and print summary.

## On `round`

Read `.claude/latest/ROUND_STATE.md`, run one iteration of the round loop.
