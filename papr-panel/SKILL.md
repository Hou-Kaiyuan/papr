---
name: papr-panel
description: |
  Run a 6-agent parallel paper review panel. Use when the user says "review
  my paper", "run panel", "get feedback", or wants multi-perspective critique.
  Agents: Advisor (storyline), Expert (technical+baselines), Standard (quality),
  Brief (main-text only), Lay (accessibility), Author (defense+action list).
  Supports Agent Teams for live cross-talk when enabled.
argument-hint: "[paper_dir]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - Bash
  - Agent
  - WebSearch
  - WebFetch
---

# Multi-Agent Paper Review Panel

**Scoring calibration:** At top venues (NeurIPS, ICLR, CVPR), the mean review score
is ~5.0/10 and only ~30% of papers are accepted. LLMs are known to inflate scores
by 1-2 points. All agents MUST be calibrated: if unsure about quality, score LOWER
not higher. A score of 7+ should mean the paper is genuinely strong. Each round
scored INDEPENDENTLY — do NOT show agents previous scores.

## On invocation

1. Read `.claude/latest-run/latest/ROUND_STATE.md` (ignore error if missing). Do NOT pass previous scores to agents.
2. Write fresh empty `.claude/latest-run/latest/DISCUSSION_THREAD.md`.
3. Check mode: run `echo $CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS`
   - If `1`: use **Agent Teams mode** (live cross-talk)
   - Otherwise: use **Wave mode** (2-wave subagents)

---

## Agent Teams Mode (live cross-talk)

When `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`, agents run as teammates with direct messaging.

### Setup

```bash
# Check if enabled
if [ "$CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS" = "1" ]; then echo "TEAMS"; else echo "WAVES"; fi
```

### 1. Spawn all 6 teammates

```
Agent(team_name: "papr-panel", name: "advisor",  prompt: [advisor role + paper])
Agent(team_name: "papr-panel", name: "expert",   prompt: [expert role + paper])
Agent(team_name: "papr-panel", name: "standard", prompt: [standard role + paper])
Agent(team_name: "papr-panel", name: "brief",    prompt: [brief role + paper])
Agent(team_name: "papr-panel", name: "lay",      prompt: [lay role + paper])
Agent(team_name: "papr-panel", name: "author",   prompt: [author role + paper])
```

### 2. Teammate instructions

Each teammate's prompt includes:

```
You are [ROLE] in a live paper review panel (Agent Teams mode).

[paste roles/[role].md]

Paper location: [paper_dir]
- Read .tex files, main.pdf, and figures/ directory
- You MUST look at actual figures

You can directly message other panelists using SendMessage:
  SendMessage(target_name: "expert", content: "Your response to their point")
  SendMessage(type: "broadcast", content: "Message to all panelists")

## Workflow
1. Read the paper and post your initial review (broadcast to all)
2. Read others' reviews as they arrive and respond directly
3. Challenge points you disagree with, acknowledge good points
4. When discussion converges, post your final score

## Scoring Calibration
At top venues, mean score is ~5.0/10. Only ~30% are accepted.
If you are unsure, score LOWER not higher. A 7+ means genuinely strong.
Assume most submissions are mediocre unless clearly exceptional.

## Rules
- Score independently. Do NOT reference previous rounds.
- Address other reviewers by name when responding
- Keep exchanges focused and specific
- AUTHOR: post your final action list when discussion winds down
- ADVISOR: post synthesis + final scores when all reviews are in

Write your final review to .claude/latest-run/latest/[role]_review.md
```

### 3. Lead monitors and collects

The lead (this agent) monitors the team:
- Wait for all teammates to post final reviews
- If discussion stalls, broadcast: "Please post final scores."
- Collect all `[role]_review.md` files
- Merge into `DISCUSSION_THREAD.md`
- Write panel summary to `ROUND_STATE.md`
- Clean up: delete team

---

## Wave Mode (2-wave subagents, default)

When Agent Teams is not enabled, use file-based coordination with 2 waves.

### Wave 1: Independent Reviews

Spawn ALL 6 agents with `run_in_background: true`:

| Agent | Output file | Task |
|---|---|---|
| ADVISOR | `wave1_advisor.md` | Storyline + figures + word choice |
| EXPERT | `wave1_expert.md` | Technical + baselines (web search) |
| STANDARD | `wave1_standard.md` | Quality + structure |
| BRIEF | `wave1_brief.md` | Main-text self-containedness |
| LAY | `wave1_lay.md` | Accessibility + ideas (web search) |
| AUTHOR | `wave1_author.md` | Preemptive defense |

All files in `.claude/latest-run/latest/`. Wait for all. Merge into `DISCUSSION_THREAD.md`. Delete temp files.

### Wave 2: Cross-Discussion

Same agents, same pattern with `wave2_*.md`. Each reads full `DISCUSSION_THREAD.md` first. AUTHOR posts action list. ADVISOR synthesizes. Merge and delete.

### Agent Prompt (Wave Mode)

```
You are [ROLE] reviewing an academic paper.

[paste roles/[role].md]

Paper location: [paper_dir]
- Read .tex source files for text
- Read main.pdf for visual layout
- Read figures/ directory for individual figure inspection
You MUST look at actual figures, not just LaTeX code.

Discussion: [DISCUSSION_THREAD.md content, or "Empty" for Wave 1]

## Scoring Calibration
At top venues, mean score is ~5.0/10 and only ~30% are accepted.
If you are unsure about quality, score LOWER not higher.
A score of 7+ means the paper is genuinely strong. Assume most
submissions are mediocre unless clearly exceptional.
Score independently. Do NOT reference previous rounds.

Wave [1|2]: [Wave 1: initial review with checklists | Wave 2: respond to others by name]

Output to: .claude/latest-run/latest/wave[N]_[role].md
Format: FROM: [ROLE] / WAVE: [N] / [review] / SIGNAL: DONE
```

---

## After Panel (both modes)

Write to `.claude/latest-run/latest/ROUND_STATE.md`:
```
## Panel Summary
### Mode: [Agent Teams | Wave]
### Score: [avg]/10 (Advisor: X | Expert: Y | Standard: Z | Brief: B | Lay: W)
### Key issues
- [bullet]
### Author action list
[from AUTHOR]
```

## Roles

| Role | Reads | Personality |
|---|---|---|
| ADVISOR | Full + PDF + figures | Picky wordsmith. Title/abstract/intro/conclusion word-by-word. |
| EXPERT | Full + PDF + figures | Skeptical challenger. Web-searches every claim. |
| STANDARD | Full + PDF + figures | Balanced gatekeeper. "Would this be accepted?" |
| BRIEF | Main text + main figures | Busy reviewer. Skips appendix. |
| LAY | Full + PDF + figures | Curious explorer. Searches to learn, brainstorms ideas. |
| AUTHOR | Full + thread | Honest defender. Depth over breadth in action list. |
