---
name: papr
description: Autonomous paper improvement system for academic ML/CV research. Invoke as /papr <subcommand> [args]. Subcommands: pipeline (full end-to-end loop), panel (autonomous review panel), inspect (format/quality audit), storyline (narrative logic check), write (implement text changes), humanize (remove AI writing patterns), scout (find missing baselines/citations), experiments (design experiments from feedback), code-inspect (verify eval correctness), external-review (blind review via Codex MCP).
---

# PAPR — Paper Autonomous Pipeline Review

A single skill that orchestrates all aspects of academic paper improvement.
Install: `cp -r papr ~/.claude/skills/`

---

## Subcommand Routing

When invoked as `/papr <subcommand> [args]`, read the corresponding file and follow its instructions:

| Subcommand | File to read | Example |
|---|---|---|
| `pipeline` | `research-pipeline/SKILL.md` | `/papr pipeline start` |
| `panel` | `discussion-panel/SKILL.md` | `/papr panel` |
| `inspect` | `paper-quality-inspect.md` | `/papr inspect` |
| `storyline` | `paper-storyline.md` | `/papr storyline` |
| `write` | `paper-writer.md` | `/papr write` |
| `humanize` | `humanizer.md` | `/papr humanize all` |
| `scout` | `related-work-scout.md` | `/papr scout` |
| `experiments` | `experiment-designer.md` | `/papr experiments` |
| `code-inspect` | `code-inspector.md` | `/papr code-inspect` |
| `external-review` | `external-reviewer.md` | `/papr external-review` |

Pass any additional arguments (e.g. `start`, `score`, `all`, `focus <topic>`) directly to the loaded file as its subcommand input.

---

## Quick Reference

```
# Run full 3-round improvement loop
/papr pipeline start

# Run just the review panel
/papr panel

# Panel — scores only, no discussion
/papr panel score

# Panel — single role turn
/papr panel role ADVISOR

# Panel — focused on a specific topic
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
/papr experiments                # design experiments from feedback
/papr code-inspect               # verify eval code before running
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
├── SKILL.md                         ← you are here
├── agent-protocol.md                ← shared message bus protocol
├── paper-quality-inspect.md
├── paper-storyline.md
├── experiment-designer.md
├── code-inspector.md
├── external-reviewer.md
├── paper-writer.md
├── related-work-scout.md
├── humanizer.md
├── discussion-panel/
│   ├── SKILL.md                     ← panel router + turn sequence
│   └── roles/
│       ├── advisor.md
│       ├── reviewer-expert.md
│       ├── reviewer-standard.md
│       ├── reviewer-lay.md
│       └── author.md
└── research-pipeline/
    ├── SKILL.md                     ← pipeline router + phase sequence
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
