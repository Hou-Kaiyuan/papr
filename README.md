# PAPR -- Paper Autonomous Pipeline Review

5 Claude Code skills for autonomous academic paper improvement. Each skill runs independently with its own context.

## Install

```bash
# Install PAPR skills
git clone https://github.com/Hou-Kaiyuan/papr.git /tmp/papr
cp -r /tmp/papr/papr-* ~/.claude/skills/

# Install humanizer (required by papr-write)
git clone https://github.com/blader/humanizer.git ~/.claude/skills/humanizer
```

### Codex MCP (optional, for external review)

```bash
npm install -g @openai/codex
codex setup
claude mcp add codex -s user -- codex mcp-server
```

## Usage

### Full pipeline (the main workflow)

```bash
# Run 3 rounds of autonomous improvement on a LaTeX project
/papr-pipeline 3 path/to/latex-project

# Run 2 rounds
/papr-pipeline 2 path/to/latex-project

# Default (3 rounds, will ask for paper path)
/papr-pipeline
```

### Review panel only

```bash
# Run the 6-agent parallel review panel
/papr-panel path/to/latex-project

# Scores only, no discussion
/papr-panel path/to/latex-project score

# Focus the panel on a specific topic
/papr-panel path/to/latex-project focus "related work"
```

### Write + humanize

```bash
# Implement text changes from the panel's action list, then remove AI writing patterns
/papr-write path/to/latex-project
```

### External review

```bash
# Send paper for blind review via Codex MCP (requires Codex MCP setup)
/papr-review path/to/latex-project
```

### Experiment design

```bash
# Design and verify experiments from reviewer feedback
/papr-experiment path/to/latex-project
```

## How the Pipeline Works

```
/papr-pipeline chains these per round:

  /papr-panel        6 agents review in parallel, produce score + action list
       |
  /papr-experiment   design experiments if needed (conditional)
       |
  /papr-write        implement text changes, then /humanizer on modified sections
       |
  /papr-review       blind external review via Codex MCP

  -> score, summary, ask user to continue
```

Each skill is invoked as a separate skill call with its own fresh context. Pipeline stops early at score 9+/10.

## Output

All generated files are saved in the working directory under `.claude/`:

```
.claude/
├── 03-18-16-round-1/
│   ├── ROUND_STATE.md
│   └── DISCUSSION_THREAD.md
├── 03-18-17-round-2/
│   └── ...
└── latest -> 03-18-17-round-2/
```

Paper source files are never polluted with pipeline artifacts.

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
│       ├── advisor.md
│       ├── reviewer-expert.md
│       ├── reviewer-standard.md
│       ├── reviewer-brief.md
│       ├── reviewer-lay.md
│       └── author.md
├── papr-write/
│   └── SKILL.md                # text changes, then invokes /humanizer
├── papr-review/
│   └── SKILL.md                # external blind review via Codex MCP
└── papr-experiment/
    └── SKILL.md                # experiment design + code verification
```

## Requirements

- [Claude Code](https://claude.ai/code)
- [humanizer](https://github.com/blader/humanizer) skill (required by papr-write)
- [Codex CLI](https://github.com/openai/codex) + MCP server (optional, for /papr-review)
