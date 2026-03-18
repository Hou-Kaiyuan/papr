# PAPR -- Paper Autonomous Pipeline Review

5 Claude Code skills for autonomous academic paper improvement. Each skill runs independently with its own context -- no interruptions from context overload.

## Install

```bash
git clone https://github.com/Hou-Kaiyuan/papr.git /tmp/papr
cp -r /tmp/papr/papr-* ~/.claude/skills/
```

### Codex MCP (optional, for external review)

```bash
npm install -g @openai/codex
codex setup
claude mcp add codex -s user -- codex mcp-server
```

## Skills

| Skill | Command | What it does |
|---|---|---|
| **papr-pipeline** | `/papr-pipeline 3 path/to/latex-project` | Multi-round improvement loop. Chains the other 4 skills. |
| **papr-panel** | `/papr-panel path/to/latex-project` | 6-agent parallel review panel (2 waves) |
| **papr-write** | `/papr-write path/to/latex-project` | Implement text changes + remove AI writing patterns |
| **papr-review** | `/papr-review path/to/latex-project` | External blind review via Codex MCP |
| **papr-experiment** | `/papr-experiment path/to/latex-project` | Design + verify experiments from feedback |

## How the Pipeline Works

```
/papr-pipeline chains these per round:

  /papr-panel        6 agents review in parallel, produce score + action list
       |
  /papr-experiment   design experiments if needed (conditional)
       |
  /papr-write        implement text changes + humanize
       |
  /papr-review       blind external review via Codex MCP

  -> score, summary, ask user to continue
```

Each skill is invoked as a separate skill call with its own fresh context. Pipeline stops early at score 9+/10.

## Panel Agents

| Agent | Reads | Built-in Checklists |
|---|---|---|
| Advisor | Full paper | Storyline arc, figure/table quality, captions |
| Expert | Full paper | Technical correctness, baselines (web search), citations, notation |
| Standard | Full paper | Overall quality, structure, novelty assessment |
| Brief | Main only | Self-containedness without appendix |
| Lay | Full paper | Accessibility, jargon, caption clarity |
| Author | Full + thread | Defends paper, produces action list |

## File Structure

```
papr/
├── README.md
├── papr-pipeline/
│   └── SKILL.md
├── papr-panel/
│   ├── SKILL.md
│   └── roles/
│       ├── advisor.md          # storyline + figures + captions
│       ├── reviewer-expert.md  # technical + baselines + citations
│       ├── reviewer-standard.md # quality + structure
│       ├── reviewer-brief.md   # main-text self-containedness
│       ├── reviewer-lay.md     # accessibility + caption clarity
│       └── author.md           # defend + action list
├── papr-write/
│   └── SKILL.md                # text changes + humanize (24 patterns)
├── papr-review/
│   └── SKILL.md                # external blind review via Codex MCP
└── papr-experiment/
    └── SKILL.md                # experiment design + code verification
```

## Requirements

- [Claude Code](https://claude.ai/code)
- [Codex CLI](https://github.com/openai/codex) + MCP server (optional, for /papr-review)
