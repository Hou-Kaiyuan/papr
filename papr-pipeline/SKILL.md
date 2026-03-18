---
name: papr-pipeline
description: |
  Autonomous multi-round paper improvement pipeline. Chains /papr-panel,
  /papr-experiment, /papr-write, and /papr-review in sequence. Each round
  produces a score out of 10. Stops early at 9+. Fully autonomous — does
  NOT ask user between rounds. Invoke as /papr-pipeline [N] [paper_dir].
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

Fully autonomous. Do NOT ask the user between rounds. Do NOT pause for confirmation.

Each round chains:
1. `/papr-panel` -- multi-agent review (reads PDF + figures + tex), produces action list + score
2. `/papr-experiment` -- design experiments if needed (conditional)
3. `/papr-write` -- implement text changes + humanize + compile PDF
4. `/papr-review` -- external blind review via Codex MCP

## Output Directory

Each round gets its own directory for clarity:
```
.claude/papr-[MM-DD-HH]/
├── round-1/
│   ├── ROUND_STATE.md
│   └── DISCUSSION_THREAD.md
├── round-2/
│   └── ...
├── round-3/
│   └── ...
└── latest -> round-1/    (updated as rounds progress)
```

## On invocation

IMPORTANT: Do NOT explore the filesystem. Do NOT run `ls`. Follow these exact steps:

1. The user provides `paper_dir` as an argument. If not given, ask: "Which folder is your LaTeX project?" and wait. Do NOT guess or search.
2. Create ALL directories upfront in a single bash command:
```bash
BASE=.claude/papr-$(date +%m-%d-%H)
mkdir -p $BASE/round-1 $BASE/round-2 $BASE/round-3  # (adjust to N rounds)
ln -sfn round-1 $BASE/latest
ln -sfn $BASE .claude/latest-run
```
3. Write `.claude/latest-run/latest/ROUND_STATE.md`:
```
# Pipeline State
## Paper: [paper_dir]
## Rounds: [N, default 3]
## Started: [date]
```
4. Write `.claude/latest-run/latest/DISCUSSION_THREAD.md` with empty content.
5. Start the round loop immediately. Do NOT ask the user anything.

## Round loop

Fully autonomous. No user interaction between rounds.

```
for round in 1..N:
    # Point latest to this round's directory
    ln -sfn round-{round} .claude/latest-run/latest

    invoke /papr-panel [paper_dir]
    read .claude/latest-run/latest/ROUND_STATE.md for action list
    if action list has "requires new experiment":
        invoke /papr-experiment [paper_dir]
    invoke /papr-write [paper_dir]
    invoke /papr-review [paper_dir]
    read .claude/latest-run/latest/ROUND_STATE.md for score
    if score >= 9: stop early, say "Paper ready. Score: [X]/10"

# After all rounds complete, print final summary with score progression.
```

To invoke each skill, use the Skill tool: `skill: "papr-panel", args: "[paper_dir]"`.

## On `status`

Read `.claude/latest-run/latest/ROUND_STATE.md` and print summary.

## On `round`

Read current state, run one iteration of the round loop.
