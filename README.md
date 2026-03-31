# PAPR -- Paper Autonomous Pipeline Review

5 Claude Code skills for autonomous academic paper improvement. Multi-agent review panels, iterative refinement, AI writing removal, and external blind review.

## Install

```bash
# Install PAPR skills
git clone https://github.com/Hou-Kaiyuan/papr.git /tmp/papr
cp -r /tmp/papr/papr-* ~/.claude/skills/

# Install humanizer (required by papr-write)
git clone https://github.com/blader/humanizer.git ~/.claude/skills/humanizer
```

### Optional: Codex MCP (for external review)

```bash
npm install -g @openai/codex
codex setup
claude mcp add codex -s user -- codex mcp-server
```

### Optional: Agent Teams (for live panel cross-talk)

By default, panel agents communicate via 2-wave file exchange. For live real-time
discussion between agents, enable the experimental Agent Teams feature:

```bash
# Option 1: Environment variable
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1

# Option 2: Claude Code settings (persistent)
# Add to ~/.claude/settings.json:
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

Requires Claude Code v2.1.32+. When enabled, panel agents use `SendMessage` for
direct peer-to-peer communication instead of wave files.

### LaTeX (required by papr-write)

```bash
# macOS
brew install --cask mactex && brew install poppler

# Ubuntu/Debian
sudo apt install texlive-full latexmk poppler-utils

# Verify
latexmk --version && pdfinfo -v
```

## Usage

### Full pipeline

```bash
/papr-pipeline 3 path/to/latex-project      # 3 rounds
/papr-pipeline 2 path/to/latex-project      # 2 rounds
/papr-pipeline                               # asks for rounds + path
```

### Review panel only

```bash
/papr-panel path/to/latex-project            # full review
/papr-panel path/to/latex-project score      # scores only
/papr-panel path/to/latex-project focus "related work"
```

### Write + humanize + compile

```bash
/papr-write path/to/latex-project
```

### External review

```bash
/papr-review path/to/latex-project
```

### Experiment design

```bash
/papr-experiment path/to/latex-project
```

## How the Pipeline Works

```
/papr-pipeline chains per round (fully autonomous):

  /papr-panel        6 agents review in parallel -> score + action list
       |
  /papr-experiment   design + verify + run experiments (if needed)
       |
  /papr-write        edit text -> /humanizer -> compile PDF
       |
  /papr-review       blind external review via Codex MCP

  -> both scores recorded, next round starts automatically
  -> stops early at 9+/10
```

## Panel Modes

### Wave Mode (default)

Two rounds of parallel execution. All agents write independently (Wave 1),
results are merged, then all agents respond to each other (Wave 2).

```
Wave 1: All 6 agents review independently (parallel)
           |
        merge into DISCUSSION_THREAD.md
           |
Wave 2: All 6 agents respond to each other (parallel)
           |
        merge -> scores + action list
```

### Agent Teams Mode (experimental)

When `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`, agents communicate in real time
via direct messaging. More natural discussion flow.

```
advisor <-> expert <-> standard
   ^                      |
   |    Live SendMessage   v
  lay  <->  author  <->  brief
```

Agents challenge, respond, and converge organically. The lead collects
final reviews when discussion settles.

## Panel Agents

| Agent | Reads | Personality |
|---|---|---|
| Advisor | Full + PDF + figures | Picky wordsmith. Reads title/abstract/intro/conclusion word-by-word. |
| Expert | Full + PDF + figures | Skeptical challenger. Web-searches every claim and baseline. |
| Standard | Full + PDF + figures | Balanced gatekeeper. "Would this be accepted at a top venue?" |
| Brief | Main text only | Busy reviewer. Skips appendix. Flags missing main-text content. |
| Lay | Full + PDF + figures | Curious explorer. Searches to learn, brainstorms cross-domain ideas. |
| Author | Full + thread | Honest defender. Depth over breadth. Produces structured action list. |

## Output

All generated files saved under `.claude/` in the working directory:

```
.claude/
├── papr-03-18-16/
│   ├── round-1/
│   │   ├── ROUND_STATE.md       # scores, action list, changes
│   │   └── DISCUSSION_THREAD.md # full panel discussion
│   ├── round-2/
│   │   └── ...
│   └── latest -> round-2/
└── latest-run -> papr-03-18-16/
```

Paper source files are never polluted with pipeline artifacts.

## File Structure

```
papr/
├── README.md
├── papr-pipeline/
│   └── SKILL.md                     # orchestrator
├── papr-panel/
│   ├── SKILL.md                     # panel coordinator (wave + teams modes)
│   └── roles/
│       ├── advisor.md               # storyline + figures + word choice
│       ├── reviewer-expert.md       # technical + baselines + citations
│       ├── reviewer-standard.md     # quality + structure + novelty
│       ├── reviewer-brief.md        # main-text self-containedness
│       ├── reviewer-lay.md          # accessibility + curiosity + ideas
│       └── author.md               # defense + action list
├── papr-write/
│   └── SKILL.md                     # edit + /humanizer + compile PDF
├── papr-review/
│   └── SKILL.md                     # external blind review via Codex MCP
└── papr-experiment/
    └── SKILL.md                     # design + TDD + monitor + figures
```

## Requirements

| Dependency | Required by | Required? |
|---|---|---|
| [Claude Code](https://claude.ai/code) | all | yes |
| [humanizer](https://github.com/blader/humanizer) skill | papr-write | yes |
| LaTeX + latexmk + poppler | papr-write | yes |
| [Codex CLI](https://github.com/openai/codex) + MCP | papr-review | optional |
| Agent Teams (`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`) | papr-panel | optional |
