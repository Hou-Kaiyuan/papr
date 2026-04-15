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

CRITICAL: Do NOT read ROUND_STATE.md before spawning agents. Reading it contaminates
your context with previous scores, which leaks into agent prompts even if you try
not to pass them. Only read ROUND_STATE.md AFTER wave 2 to write results.

1. Write fresh empty `.claude/latest-run/latest/DISCUSSION_THREAD.md`.
2. Check mode: run `echo $CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS`
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

### 0. Clean up any existing team FIRST

BEFORE creating the team, always try to delete any existing "papr-panel" team:

```
try: TeamDelete(team_name: "papr-panel")
catch: ignore (no existing team)
```

This prevents the "Already leading team" error when running multiple rounds.

### 1. Spawn all 6 teammates

Use a unique team name per run with timestamp to avoid conflicts:

```
team_name = "papr-panel-$(date +%s)"

Agent(team_name: [team_name], name: "advisor",  prompt: [advisor role + paper])
Agent(team_name: [team_name], name: "expert",   prompt: [expert role + paper])
Agent(team_name: [team_name], name: "standard", prompt: [standard role + paper])
Agent(team_name: [team_name], name: "brief",    prompt: [brief role + paper])
Agent(team_name: [team_name], name: "lay",      prompt: [lay role + paper])
Agent(team_name: [team_name], name: "author",   prompt: [author role + paper])
```

### 2. Teammate instructions

Each teammate's prompt includes:

```
You are [ROLE] reviewing a paper submitted to a top ML/CV venue.
This is a FRESH review. You have never seen this paper before.

[paste roles/[role].md]

## MANDATORY TOOL USE
You MUST use tools to complete this review. A review with 0 tool uses is invalid.

Required reads (use the Read tool):
1. [paper_dir]/main.tex (or the main .tex file)
2. Any \input{} files referenced from main.tex (method.tex, evaluation.tex, etc.)
3. [paper_dir]/main.pdf (for visual layout)
4. Individual figures in [paper_dir]/figures/ (at least 3 figure files)

You MUST read at least 5 files before writing your review.
Do NOT just respond from prompt text -- actually Read the paper files.

EXPERT and LAY roles: you MUST also use WebSearch at least 2 times.

Paper location: [paper_dir]
- Do NOT read ROUND_STATE.md or any pipeline state files
- Do NOT look at git history or diffs

## Communication
You can directly message other panelists using SendMessage:
  SendMessage(target_name: "expert", content: "Your response to their point")
  SendMessage(type: "broadcast", content: "Message to all panelists")

## Workflow
1. READ the paper files (tool use required)
2. Post your initial review (broadcast to all)
3. Read others' reviews as they arrive and respond directly
4. Challenge points you disagree with, acknowledge good points
5. When discussion converges, post your final score

## Scoring Calibration
At top venues, mean score is ~5.0/10. Only ~30% are accepted.
If you are unsure, score LOWER not higher. A 7+ means genuinely strong.
Assume most submissions are mediocre unless clearly exceptional.

## Rules
- This is a BLIND review. You know NOTHING about previous versions or revisions.
- Address other reviewers by name when responding
- Keep exchanges focused and specific
- AUTHOR: post your final action list when discussion winds down
- ADVISOR: post synthesis + final scores when all reviews are in

Use the Write tool to save your final review to:
  .claude/latest-run/latest/[role]_review.md
```

### 3. Lead monitors and collects

The lead (this agent) monitors the team:
- Wait for all teammates to post final reviews
- If discussion stalls, broadcast: "Please post final scores."
- Collect all `[role]_review.md` files
- Merge into `DISCUSSION_THREAD.md`
- Write panel summary to `ROUND_STATE.md`
- **MANDATORY cleanup:** `TeamDelete(team_name: [team_name])` — do NOT skip this

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
You are [ROLE] reviewing a paper submitted to a top ML/CV venue.
This is a FRESH review. You have never seen this paper before.

[paste roles/[role].md]

## MANDATORY TOOL USE
You MUST use tools to complete this review. A review with 0 tool uses is invalid.

Required reads (use the Read tool):
1. [paper_dir]/main.tex (or the main .tex file)
2. Any \input{} files referenced from main.tex
3. [paper_dir]/main.pdf (for visual layout)
4. Individual figures in [paper_dir]/figures/ (at least 3 figure files)

You MUST read at least 5 files before writing your review.
Do NOT just respond from prompt text -- actually Read the paper files.
EXPERT and LAY: you MUST also use WebSearch at least 2 times.

Paper location: [paper_dir]
Do NOT read ROUND_STATE.md or any pipeline state files.
Do NOT look at git history or diffs.
Do NOT explore the .claude/ directory or any subdirectories within it.
Do NOT run `ls .claude/`, `find .claude/`, or any filesystem exploration there.
Only read files explicitly listed in this prompt (paper .tex, main.pdf, figures/).
You only have access to the current paper -- no other run history exists for you.

Discussion: [DISCUSSION_THREAD.md content, or "Empty" for Wave 1]

## Scoring Calibration
At top venues, mean score is ~5.0/10 and only ~30% are accepted.
If you are unsure about quality, score LOWER not higher.
A score of 7+ means the paper is genuinely strong. Assume most
submissions are mediocre unless clearly exceptional.
This is a BLIND review. You know NOTHING about previous versions.

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
