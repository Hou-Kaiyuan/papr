# PAPR -- Paper Autonomous Pipeline Review

5 Claude Code skills for autonomous academic paper improvement: multi-agent
review panels with calibrated scoring, iterative refinement, real-bibtex
citation handling, AI-writing removal, and external blind review via the Codex
plugin (latest GPT model).

## Install

### Option A: Symlink (recommended, easy updates)

```bash
git clone https://github.com/Hou-Kaiyuan/papr.git ~/.claude/papr
ln -sfn ~/.claude/papr/papr-* ~/.claude/skills/
# Update anytime: cd ~/.claude/papr && git pull
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

### Codex plugin (required by papr-review and papr-experiment)

papr uses the Codex plugin for external blind review (`/papr-review`) and
adversarial code review (`/papr-experiment`). The Codex MCP path is no longer
supported -- use the plugin only.

```bash
npm install -g @openai/codex
codex login

# Inside Claude Code:
/plugin marketplace add openai/codex-plugin-cc
/plugin install codex@openai-codex
/reload-plugins
/codex:setup
```

Requires a ChatGPT subscription or OpenAI API key, plus Node.js 18.18+. Codex
auto-routes to the latest GPT model -- papr does not hardcode a version.

### Agent Teams (REQUIRED for papr-panel)

papr-panel runs as a 6-teammate Agent Team with live cross-talk. **The feature
flag must be set when Claude Code launches** -- it cannot be enabled from
inside a session. Quit Claude Code, then start with the env var:

```bash
CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1 claude
```

Or persist it in `~/.claude/settings.json`:

```json
{ "env": { "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1" } }
```

If the flag is unset, papr-panel will abort with restart instructions.

### LaTeX (required by papr-write)

```bash
# macOS
brew install --cask mactex && brew install poppler

# Ubuntu/Debian
sudo apt install texlive-full latexmk poppler-utils
```

## Usage

```bash
# Full pipeline (main workflow, auto-invocation disabled -- user must run)
/papr-pipeline 3 path/to/latex-project      # 3 rounds, fully autonomous
/papr-pipeline 2 path/to/latex-project      # 2 rounds

# Individual skills
/papr-panel path/to/latex-project            # 6-agent live-cross-talk panel
/papr-write path/to/latex-project            # edit + humanize + compile
/papr-review path/to/latex-project           # external blind review (Codex)
/papr-experiment path/to/latex-project       # experiment design + TDD
```

## How the pipeline works

```
/papr-pipeline chains per round (fully autonomous):

  /papr-panel        6 calibrated agents in live cross-talk -> score + action list
       |
  /papr-experiment   design + /codex:adversarial-review + TDD + run
       |
  /papr-write        edit -> bibtex download -> /humanizer -> compile PDF
       |
  /papr-review       blind review via Codex plugin (latest GPT)

  -> both scores recorded, next round automatically
  -> stops at 9+/10
```

## Score calibration

LLMs are known to inflate review scores by 1-2 points (mean ~7 vs human mean ~5).
All papr reviewers are calibrated:

- Mean score at top venues is ~5.0/10, acceptance rate ~30%
- "If unsure, score LOWER not higher"
- "Assume most submissions are mediocre unless clearly exceptional"
- A score of 7+ means genuinely strong work
- External review includes the same calibration in its prompt

## Panel: 6 agents, live cross-talk

```
advisor <-> expert <-> standard
   ^                      |
   |    Live SendMessage  v
  lay  <->  author  <->  brief
```

| Agent | Reads | Personality |
|---|---|---|
| Advisor | Full + PDF + figures | Picky wordsmith. Word-by-word on title/abstract/intro/conclusion. Synthesizes final scores. |
| Expert | Full + PDF + figures | Skeptical devil's advocate. Web-searches every claim. Challenges method. |
| Standard | Full + PDF + figures | Balanced gatekeeper. "Would this pass at a top venue?" |
| Brief | Main text only | Busy reviewer. Skips appendix. Flags missing content. |
| Lay | Full + PDF + figures | Curious explorer. Searches to learn, brainstorms ideas. |
| Author | Full + thread | Honest defender. Depth over breadth. Posts the action list. |

## Citation handling

papr-write and papr-experiment download the **real bibtex** for every new
`\cite{}` (arXiv, ACL Anthology, OpenReview, Semantic Scholar API) and save it
to `[paper_dir]/references/<citekey>.bib`. WebSearch alone is not enough --
LLMs hallucinate fields. If a paper cannot be located, the citation is removed
rather than fabricated.

## Depth over shallow statistics

Reviewers, the writer, the experiment designer, and the author all flag and
refuse the following as "deep analysis":

- Spearman/Pearson correlation
- Cohen's d / Cohen's kappa
- Wilson confidence intervals
- Bootstrap CI / bootstrapped error bars
- t-tests, McNemar's test, Fisher's exact test, chi-squared
- p-values and "statistically significant (p<0.05)" framing

Real depth = mechanism explanation, failure-case study, ablation isolating one
component, qualitative analysis, learned-representation visualization.

## Codex integration

| Command | Used by | What it does |
|---|---|---|
| `/codex:adversarial-review` | papr-experiment | Challenge code design decisions |
| `/codex:rescue` | papr-experiment | Debug failing experiments |
| Codex review prompt (latest GPT) | papr-review | Blind external paper review |

## Output

Run artifacts are written to `papr-runs/` in the working directory (NOT
`.claude/`, since writes there are gated by Claude Code permission prompts
even with `--dangerously-skip-permissions`).

```
papr-runs/
├── papr-03-18-16-42-07/
│   ├── round-1/   ROUND_STATE.md, DISCUSSION_THREAD.md
│   ├── round-2/   ...
│   └── latest -> round-2/
└── latest-run -> papr-03-18-16-42-07/
```

Add `papr-runs/` to your paper repo's `.gitignore`.

## File structure

```
papr/
├── README.md
├── papr-pipeline/   SKILL.md (orchestrator; auto-invocation disabled)
├── papr-panel/      SKILL.md + roles/ (6 calibrated reviewers, Agent Teams)
├── papr-write/      SKILL.md (edit + bibtex download + /humanizer + compile)
├── papr-review/     SKILL.md (Codex-plugin blind review)
└── papr-experiment/ SKILL.md (design + /codex:adversarial-review + TDD)
```

## Requirements

| Dependency | Required by | Required? |
|---|---|---|
| [Claude Code](https://claude.ai/code) | all | yes |
| `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` at launch | papr-panel | yes |
| [humanizer](https://github.com/blader/humanizer) skill | papr-write | yes |
| LaTeX + latexmk + poppler | papr-write | yes |
| [Codex plugin](https://github.com/openai/codex-plugin-cc) | papr-review, papr-experiment | yes |
