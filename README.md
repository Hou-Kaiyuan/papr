# PAPR — Paper Autonomous Pipeline Review

A Claude Code skill for autonomous academic paper improvement. Multi-agent review panels, iterative refinement, and AI writing removal in one pipeline.

## Install

```bash
cp -r papr ~/.claude/skills/
```

## Core Workflows

### Pipeline — full multi-round improvement

The main workflow. Runs up to 3 rounds of autonomous improvement, each scored out of 10. Stops early at 9+.

```bash
/papr pipeline start                # begin full loop
/papr pipeline status               # check score and progress
/papr pipeline round                # run next round manually
```

Each round runs: Scout → Inspect → Panel → Experiments (if needed) → Write → Humanize → External Review → Summary.

### Panel — autonomous review

6 agents (Advisor, Expert, Standard, Brief, Lay, Author) review your paper in parallel — like sitting in the same room. Two waves: independent perspectives, then cross-discussion. You see only the final summary with scores and an action list.

```bash
/papr panel                         # full 2-wave parallel session
/papr panel score                   # scores only, no discussion
/papr panel focus "related work"    # focused session
```

### Write + Humanize — revision

Implement changes from the panel's action list, then remove AI writing patterns.

```bash
/papr write                         # apply text changes
/papr humanize all                  # remove AI writing patterns
/papr humanize check                # scan without changing
```

## Other Tools

```bash
/papr inspect                       # format/figure/table audit
/papr storyline                     # narrative logic check
/papr scout                         # find missing baselines/citations
/papr experiments                   # design + verify experiments
/papr external-review               # blind review via Codex MCP
```

## How the Pipeline Works

```
Round N (repeat up to 3x)
│
├─ 1. Scout        find missing related work and baselines
├─ 2. Inspect      audit formatting, figures, tables, narrative
├─ 3. Panel        parallel multi-agent review (6 agents, 2 waves)
├─ 4. Experiments  design + verify new experiments (if needed)
├─ 5. Write        implement all text changes
├─ 6. Humanize     remove AI writing artifacts
├─ 7. Review       blind external review via Codex MCP
└─ 8. Summary      score, log changes, decide next round
```

Human is consulted only between rounds and for escalated issues.

## File Structure

```
papr/
├── SKILL.md                         # entry point + routing
├── agent-protocol.md                # shared multi-agent protocol
├── inspect.md                       # format/quality audit
├── storyline.md                     # narrative logic check
├── experiments.md                   # experiment design + code verification
├── external-review.md               # blind review via Codex MCP
├── write.md                         # text change implementation
├── scout.md                         # missing baselines/citations
├── humanize.md                      # AI writing pattern removal
├── discussion-panel/
│   ├── SKILL.md                     # panel router + turn sequence
│   └── roles/
│       ├── advisor.md               # senior professor, big-picture
│       ├── reviewer-expert.md       # domain expert, main + appendix
│       ├── reviewer-standard.md     # area chair, main + appendix
│       ├── reviewer-brief.md        # busy reviewer, main text only
│       ├── reviewer-lay.md          # non-expert, reads everything
│       └── author.md                # paper defense + action list
└── research-pipeline/
    ├── SKILL.md                     # pipeline router + phases
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

## Requirements

- [Claude Code](https://claude.ai/code) with skill support
- Codex MCP server (optional, for `/papr external-review`)
