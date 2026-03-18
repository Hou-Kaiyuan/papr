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

When invoked as `/papr <subcommand> [args]`, read ONLY the corresponding file
and immediately execute its instructions. Do NOT pre-read other files.

| Subcommand | File to read and execute |
|---|---|
| `pipeline` | `research-pipeline/SKILL.md` |
| `panel` | `discussion-panel/SKILL.md` |
| `inspect` | `inspect.md` |
| `storyline` | `storyline.md` |
| `write` | `write.md` |
| `humanize` | `humanize.md` |
| `scout` | `scout.md` |
| `experiments` | `experiments.md` |
| `external-review` | `external-review.md` |

Pass any additional arguments (e.g. `start 5`, `score`, `all`, `focus <topic>`)
directly to the loaded file as its subcommand input.

---

## Quick Reference

```
# Full multi-round improvement (the main workflow)
/papr pipeline start 3 paper/Vivar      # 3 rounds on a LaTeX project folder
/papr pipeline start 2 paper/Vivar      # 2 rounds
/papr pipeline start                    # default 3 rounds, will ask for folder

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
