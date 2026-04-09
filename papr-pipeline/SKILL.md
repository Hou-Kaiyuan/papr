---
name: papr-pipeline
description: |
  Run autonomous multi-round paper improvement on a LaTeX project. Use when
  the user says "improve my paper", "review pipeline", "run papr", or wants
  iterative paper improvement. Chains /papr-panel -> /papr-experiment ->
  /papr-write -> /papr-review per round. Stops at score 9+/10.
argument-hint: "[rounds] [paper_dir]"
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

Each round: `/papr-panel` -> `/papr-experiment` (if needed) -> `/papr-write` -> `/papr-review` (GPT-5.4 via Codex)

## On invocation

1. `paper_dir` = second argument. If missing, ask once and wait.
2. `N` = first argument (default 3).
3. Setup directories and state:
```bash
BASE=.claude/papr-$(date +%m-%d-%H)
mkdir -p $BASE/round-{1..$N}
ln -sfn round-1 $BASE/latest
ln -sfn $BASE .claude/latest-run
```
4. Write `.claude/latest-run/latest/ROUND_STATE.md`:
```
# Pipeline State
## Paper: [paper_dir]
## Rounds: [N]
## Started: [date]
```
5. Write empty `.claude/latest-run/latest/DISCUSSION_THREAD.md`.
6. Start round loop.

## Round loop

Each round MUST complete ALL 4 phases before starting the next.

CRITICAL ISOLATION: Panel and external review MUST run without any prior-round
context. Spawn them as isolated Agents so they get a completely fresh context
window with zero knowledge of previous rounds. This prevents score inflation.

```
for round in 1..N:
    ln -sfn round-{round} [BASE]/latest

    # Panel: spawn as ISOLATED Agent to prevent context contamination
    # The agent gets a fresh context -- no prior round scores, no history
    Agent(prompt: "Run /papr-panel [paper_dir]. This is a fresh first-time
      review. You have no knowledge of previous rounds or scores.")

    # Only read state AFTER panel writes it
    read ROUND_STATE.md for action list
    if action list has "requires new experiment":
        invoke /papr-experiment [paper_dir]
    invoke /papr-write [paper_dir]

    # External review: also spawn as ISOLATED Agent
    Agent(prompt: "Run /papr-review [paper_dir]. This paper has never been
      reviewed before. Do not read any pipeline state before constructing
      the review prompt.")

    # Verify BOTH scores exist before proceeding
    read ROUND_STATE.md -> confirm panel score + external score (or "skipped")
    if combined score >= 9: stop, print "Paper ready."

# Print final table: Round | Internal | External
```

Panel and review are invoked via Agent tool (fresh context), NOT via Skill tool
(which would inherit the pipeline's contaminated context).
Experiment and write CAN use Skill tool since they need prior round context.

## Output

```
.claude/papr-[MM-DD-HH]/
├── round-1/   ROUND_STATE.md, DISCUSSION_THREAD.md
├── round-2/   ...
└── latest -> round-N/
```
