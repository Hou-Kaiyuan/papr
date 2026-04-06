# PAPR -- Paper Autonomous Pipeline Review

5 Claude Code skills for autonomous academic paper improvement. Multi-agent review panels with calibrated scoring, iterative refinement, AI writing removal, and external blind review via GPT-5.4.

## Install

### Option A: Symlink (recommended, easy updates)

```bash
# Clone once
git clone https://github.com/Hou-Kaiyuan/papr.git ~/papr

# Symlink skills into Claude Code
ln -sfn ~/papr/papr-* ~/.claude/skills/

# Update anytime with just:
cd ~/papr && git pull
```

### Option B: Copy

```bash
git clone https://github.com/Hou-Kaiyuan/papr.git /tmp/papr
cp -r /tmp/papr/papr-* ~/.claude/skills/
```

### Humanizer (required by papr-write)

```bash
git clone https://github.com/blader/humanizer.git ~/.claude/skills/humanizer
```

### Codex Plugin (for external review + code inspection)

Uses GPT-5.4 via Codex for blind external review (`/papr-review`) and adversarial
code review (`/papr-experiment`). Also provides `/codex:rescue` for debugging.

```bash
# Install Codex CLI
npm install -g @openai/codex
codex login

# Install Codex plugin for Claude Code
/plugin marketplace add openai/codex-plugin-cc
/plugin install codex@openai-codex
/reload-plugins
/codex:setup
```

Requires ChatGPT subscription or OpenAI API key. Node.js 18.18+.

### Optional: Agent Teams (for live panel cross-talk)

```bash
# Environment variable
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1

# Or persistent (add to ~/.claude/settings.json):
{ "env": { "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1" } }
```

### LaTeX (required by papr-write)

```bash
# macOS
brew install --cask mactex && brew install poppler

# Ubuntu/Debian
sudo apt install texlive-full latexmk poppler-utils
```

## Usage

```bash
# Full pipeline (main workflow)
/papr-pipeline 3 path/to/latex-project      # 3 rounds, fully autonomous
/papr-pipeline 2 path/to/latex-project      # 2 rounds

# Individual skills
/papr-panel path/to/latex-project            # 6-agent review panel
/papr-write path/to/latex-project            # edit + humanize + compile
/papr-review path/to/latex-project           # external review (GPT-5.4)
/papr-experiment path/to/latex-project       # experiment design + TDD
```

## How the Pipeline Works

```
/papr-pipeline chains per round (fully autonomous):

  /papr-panel        6 calibrated agents -> score + action list
       |
  /papr-experiment   design + /codex:adversarial-review + TDD + run
       |
  /papr-write        edit -> /humanizer -> compile PDF
       |
  /papr-review       blind review via GPT-5.4 (Codex plugin)

  -> both scores recorded, next round automatically
  -> stops at 9+/10
```

## Score Calibration

LLMs are known to inflate review scores by 1-2 points (mean ~7 vs human mean ~5).
All PAPR reviewers are calibrated:

- Mean score at top venues is ~5.0/10, acceptance rate ~30%
- "If unsure, score LOWER not higher"
- "Assume most submissions are mediocre unless clearly exceptional"
- A score of 7+ means genuinely strong work
- External review (GPT-5.4) includes calibration instructions in the prompt

## Panel Modes

### Wave Mode (default)

```
Wave 1: 6 agents review independently (parallel)
Wave 2: 6 agents respond to each other (parallel)
```

### Agent Teams Mode (`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`)

```
advisor <-> expert <-> standard
   ^                      |
   |    Live SendMessage   v
  lay  <->  author  <->  brief
```

## Panel Agents

| Agent | Reads | Personality |
|---|---|---|
| Advisor | Full + PDF + figures | Picky wordsmith. Word-by-word on title/abstract/intro/conclusion. |
| Expert | Full + PDF + figures | Skeptical devil's advocate. Web-searches every claim. Challenges method. |
| Standard | Full + PDF + figures | Balanced gatekeeper. "Would this pass at a top venue?" |
| Brief | Main text only | Busy reviewer. Skips appendix. Flags missing content. |
| Lay | Full + PDF + figures | Curious explorer. Searches to learn, brainstorms ideas. |
| Author | Full + thread | Honest defender. Depth over breadth. Action list. |

## Codex Integration

| Command | Used by | What it does |
|---|---|---|
| `/codex:adversarial-review` | papr-experiment | Challenge code design decisions |
| `/codex:rescue` | papr-experiment | Debug failing experiments |
| Codex review prompt (GPT-5.4) | papr-review | Blind external paper review |

## Output

```
.claude/
├── papr-03-18-16/
│   ├── round-1/   ROUND_STATE.md, DISCUSSION_THREAD.md
│   ├── round-2/   ...
│   └── latest -> round-2/
└── latest-run -> papr-03-18-16/
```

## File Structure

```
papr/
├── README.md
├── papr-pipeline/   SKILL.md (orchestrator)
├── papr-panel/      SKILL.md + roles/ (6 calibrated reviewers)
├── papr-write/      SKILL.md (edit + /humanizer + compile)
├── papr-review/     SKILL.md (GPT-5.4 blind review via Codex)
└── papr-experiment/ SKILL.md (design + /codex:adversarial-review + TDD)
```

## Requirements

| Dependency | Required by | Required? |
|---|---|---|
| [Claude Code](https://claude.ai/code) | all | yes |
| [humanizer](https://github.com/blader/humanizer) skill | papr-write | yes |
| LaTeX + latexmk + poppler | papr-write | yes |
| [Codex plugin](https://github.com/openai/codex-plugin-cc) | papr-review, papr-experiment | recommended |
| Agent Teams | papr-panel | optional |
