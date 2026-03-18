# PAPR — Paper Autonomous Pipeline Review

A Claude Code skill for autonomous academic paper improvement. Multi-agent review panels, iterative refinement, and AI writing removal in one pipeline.

## Install

```bash
git clone https://github.com/Hou-Kaiyuan/papr.git ~/.claude/skills/papr
```

### Codex MCP Setup (for external review)

The `external-review` phase uses OpenAI Codex as a blind reviewer via MCP. This gives you an independent second opinion from a different AI model.

```bash
# 1. Install and authenticate Codex CLI
npm install -g @openai/codex
codex setup

# 2. Register as MCP server in Claude Code
claude mcp add codex -s user -- codex mcp-server
```

Verify with `claude mcp list` or `/mcp` inside Claude Code. The pipeline works without Codex, it just skips the external review phase.

---

## Usage

### Pipeline — full multi-round improvement

The main workflow. Runs multiple rounds of autonomous improvement, each scored out of 10. Stops early at 9+.

```bash
/papr pipeline start 3 path/to/latex-project      # 3 rounds on a LaTeX project folder
/papr pipeline start 2 path/to/latex-project      # 2 rounds
/papr pipeline start                    # default 3 rounds, will ask for folder
/papr pipeline status                   # check score and progress
/papr pipeline round                    # run next round manually
```

Each round: Scout, Inspect, Panel, Experiments (if needed), Write, Humanize, External Review, Summary.

### Panel — parallel multi-agent review

6 agents review your paper simultaneously in two waves. Wave 1: independent perspectives. Wave 2: cross-discussion. You see the final summary with scores and an action list.

| Agent | Reads | Perspective |
|---|---|---|
| Advisor | Full paper | Senior professor, big-picture framing |
| Expert | Full paper | Domain expert, technical rigor |
| Standard | Full paper | Area chair, overall quality |
| Brief | Main text only | Busy reviewer, self-containedness |
| Lay | Full paper | Non-expert, accessibility |
| Author | Full paper + thread | Defends paper, posts action list |

```bash
/papr panel                         # full 2-wave parallel session
/papr panel score                   # scores only
/papr panel focus "related work"    # focused session
```

### Write + Humanize — revision

```bash
/papr write                         # apply text changes from action list
/papr humanize all                  # remove AI writing patterns (24 checks)
/papr humanize check                # scan without changing
```

### Other tools

```bash
/papr inspect                       # format/figure/table audit
/papr storyline                     # narrative logic check
/papr scout                         # find missing baselines/citations
/papr experiments                   # design + verify experiments
/papr external-review               # blind review via Codex MCP
```

---

## How the Pipeline Works

```
Round N (repeat up to N rounds, default 3)
|
|-- 1. Scout        find missing related work and baselines
|-- 2. Inspect      audit formatting, figures, tables, narrative
|-- 3. Panel        parallel multi-agent review (6 agents, 2 waves)
|-- 4. Experiments  design + verify new experiments (if needed)
|-- 5. Write        implement all text changes
|-- 6. Humanize     remove AI writing artifacts
|-- 7. Review       blind external review via Codex MCP
+-- 8. Summary      score, log changes, decide next round
```

Human is consulted only between rounds and for escalated issues. Pipeline stops early if score reaches 9+/10.

---

## File Structure

```
papr/
├── SKILL.md                         # entry point + subcommand routing
├── agent-protocol.md                # shared multi-agent communication protocol
├── inspect.md                       # format/quality audit
├── storyline.md                     # narrative logic check
├── experiments.md                   # experiment design + code verification
├── external-review.md               # blind review via Codex MCP
├── write.md                         # text change implementation
├── scout.md                         # missing baselines/citations
├── humanize.md                      # AI writing pattern removal (24 patterns)
├── discussion-panel/
│   ├── SKILL.md                     # panel orchestration (2-wave parallel)
│   └── roles/
│       ├── advisor.md               # senior professor, big-picture
│       ├── reviewer-expert.md       # domain expert, main + appendix
│       ├── reviewer-standard.md     # area chair, main + appendix
│       ├── reviewer-brief.md        # busy reviewer, main text only
│       ├── reviewer-lay.md          # non-expert, reads everything
│       └── author.md                # paper defense + action list
└── research-pipeline/
    ├── SKILL.md                     # pipeline router + phase sequencing
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

- [Claude Code](https://claude.ai/code)
- [Codex CLI](https://github.com/openai/codex) + MCP server (optional, for external review)
