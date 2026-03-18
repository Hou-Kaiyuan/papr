---
name: papr
description: |
  Autonomous paper improvement system for academic ML/CV research.
  Invoke as /papr <subcommand> [args]. Core workflows: pipeline (full
  multi-round improvement loop), panel (autonomous review panel),
  write (implement text changes), humanize (remove AI writing patterns).
  Also: inspect, storyline, scout, experiments, external-review.
allowed-tools:
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - Bash
  - WebSearch
  - WebFetch
  - Agent
  - mcp__codex__codex
  - mcp__codex__codex-reply
---

# PAPR — Paper Autonomous Pipeline Review

A single skill that orchestrates all aspects of academic paper improvement.
Install: `git clone https://github.com/Hou-Kaiyuan/papr.git ~/.claude/skills/papr`

---

## Subcommand Routing

When invoked as `/papr <subcommand> [args]`, read the corresponding file and follow its instructions:

| Subcommand | File to read | Example |
|---|---|---|
| `pipeline` | `research-pipeline/SKILL.md` | `/papr pipeline start` |
| `panel` | `discussion-panel/SKILL.md` | `/papr panel` |
| `inspect` | `inspect.md` | `/papr inspect` |
| `storyline` | `storyline.md` | `/papr storyline` |
| `write` | `write.md` | `/papr write` |
| `humanize` | `humanize.md` | `/papr humanize all` |
| `scout` | `scout.md` | `/papr scout` |
| `experiments` | `experiments.md` | `/papr experiments` |
| `external-review` | `external-review.md` | `/papr external-review` |

Pass any additional arguments (e.g. `start`, `score`, `all`, `focus <topic>`) directly to the loaded file as its subcommand input.

---

## Quick Reference

```
# Full multi-round improvement (the main workflow)
/papr pipeline start                 # default 3 rounds
/papr pipeline start 5               # custom number of rounds

# Review panel only
/papr panel
/papr panel score
/papr panel focus "introduction motivation"

# Run one pipeline phase manually
/papr pipeline inspect
/papr pipeline write
/papr pipeline humanize

# Check current pipeline state
/papr pipeline status

# Standalone tools
/papr inspect                    # format/figure/table audit
/papr storyline                  # narrative logic check
/papr scout                      # find missing baselines
/papr experiments                # design + verify experiments
/papr external-review            # blind review via Codex MCP
/papr write                      # implement text changes
/papr humanize all               # remove AI writing patterns
/papr humanize check             # scan without changing
```

---

## Shared Protocol

All multi-agent subcommands (`panel`, `pipeline`) use `agent-protocol.md`.
Read it before running any panel or pipeline session.

File: `agent-protocol.md`

---

## File Structure

```
papr/
├── SKILL.md                         <- you are here
├── agent-protocol.md                <- shared message bus protocol
├── inspect.md
├── storyline.md
├── experiments.md                   <- design + code verification
├── external-review.md
├── write.md
├── scout.md
├── humanize.md
├── discussion-panel/
│   ├── SKILL.md                     <- panel router + turn sequence
│   └── roles/
│       ├── advisor.md
│       ├── reviewer-expert.md
│       ├── reviewer-standard.md
│       ├── reviewer-brief.md
│       ├── reviewer-lay.md
│       └── author.md
└── research-pipeline/
    ├── SKILL.md                     <- pipeline router + phase sequence
    └── phases/
        ├── scout.md
        ├── inspect.md
        ├── panel.md
        ├── experiments.md
        ├── write.md
        ├── humanize.md
        ├── review.md
        └── summary.md
```
