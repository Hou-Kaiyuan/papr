# PAPR — Paper Autonomous Pipeline Review

A Claude Code skill that orchestrates all aspects of academic paper improvement through autonomous multi-agent review and iterative refinement.

## Install

```bash
cp -r papr ~/.claude/skills/
```

## Usage

```bash
# Run full 3-round improvement loop
/papr pipeline start

# Run the autonomous review panel
/papr panel

# Standalone tools
/papr inspect          # format/figure/table audit
/papr storyline        # narrative logic check
/papr scout            # find missing baselines/citations
/papr experiments      # design experiments from feedback
/papr code-inspect     # verify eval code correctness
/papr external-review  # blind review via Codex MCP
/papr write            # implement text changes
/papr humanize all     # remove AI writing patterns
```

## Subcommands

| Subcommand | Description |
|---|---|
| `pipeline` | Full end-to-end multi-round improvement loop |
| `panel` | Autonomous multi-agent review panel |
| `inspect` | Format, figure, and table quality audit |
| `storyline` | Narrative and logical flow check |
| `write` | Implement text changes to the paper |
| `humanize` | Detect and remove AI writing patterns |
| `scout` | Find missing baselines and citations |
| `experiments` | Design experiments from reviewer feedback |
| `code-inspect` | Verify evaluation code correctness |
| `external-review` | Blind review via Codex MCP |

## How It Works

**Pipeline** runs up to 3 rounds of improvement. Each round:

1. **Scout** — find missing related work and baselines
2. **Inspect** — audit formatting, figures, tables, and narrative
3. **Panel** — autonomous multi-agent discussion (Advisor, Expert/Standard/Lay Reviewers, Author)
4. **Experiments** — design new experiments if needed
5. **Write** — implement text changes
6. **Humanize** — remove AI writing artifacts
7. **External Review** — blind review via Codex MCP
8. **Summary** — score and round summary

The pipeline stops early if the score reaches 9+/10.

## File Structure

```
papr/
├── SKILL.md                         # main entry point
├── agent-protocol.md                # shared message bus protocol
├── inspect.md
├── storyline.md
├── experiments.md
├── code-inspect.md
├── external-review.md
├── write.md
├── scout.md
├── humanize.md
├── discussion-panel/
│   ├── SKILL.md                     # panel router + turn sequence
│   └── roles/
│       ├── advisor.md
│       ├── reviewer-expert.md
│       ├── reviewer-standard.md
│       ├── reviewer-lay.md
│       └── author.md
└── research-pipeline/
    ├── SKILL.md                     # pipeline router + phase sequence
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
