---
name: papr-panel
description: |
  Runs a 6-agent parallel paper review panel via Agent Teams (live cross-talk).
  Agents: Advisor (storyline), Expert (technical+baselines), Standard (quality),
  Brief (main-text only), Lay (accessibility), Author (defense+action list).
  Use when the user says "review my paper", "run panel", "get feedback", or
  asks for multi-perspective critique. Requires CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1.
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
scored INDEPENDENTLY -- do NOT show agents previous scores.

## Hard prerequisite: Agent Teams must be enabled

This skill requires the Claude Code Agent Teams feature. It cannot be turned on
from inside a running session -- the env var is read at Claude Code startup.

```bash
if [ "$CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS" != "1" ]; then
  echo "ERROR: papr-panel requires CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1"
  echo "Quit and relaunch Claude Code with:"
  echo "  CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1 claude"
  exit 1
fi
```

If unset: STOP, report the error, do not attempt any fallback. Do not spawn
plain Agents as a workaround -- losing live cross-talk silently degrades panel
quality and the user should know.

## On invocation

CRITICAL: Do NOT read ROUND_STATE.md before spawning teammates. Reading it
contaminates your context with previous scores, which leaks into prompts even
if you try not to pass them. Only read ROUND_STATE.md AFTER discussion ends.

1. Verify env var (above). Abort with instructions if unset.
2. Write fresh empty `papr-runs/latest-run/latest/DISCUSSION_THREAD.md`.
3. Pick a unique team name: `team_name = "papr-panel-$(date +%s)"`.
4. Best-effort `TeamDelete(team_name: "papr-panel")` to clean up any stale
   default-named team from earlier sessions; ignore errors.

## Spawn the team

```
Agent(team_name: [team_name], name: "advisor",  prompt: [advisor role + paper])
Agent(team_name: [team_name], name: "expert",   prompt: [expert role + paper])
Agent(team_name: [team_name], name: "standard", prompt: [standard role + paper])
Agent(team_name: [team_name], name: "brief",    prompt: [brief role + paper])
Agent(team_name: [team_name], name: "lay",      prompt: [lay role + paper])
Agent(team_name: [team_name], name: "author",   prompt: [author role + paper])
```

## Teammate prompt template

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
Do NOT read ROUND_STATE.md or any pipeline state files.
Do NOT look at git history or diffs.
Do NOT explore the .claude/ or papr-runs/ directories or any subdirectories within them.
Do NOT run `ls .claude/`, `ls papr-runs/`, `find .claude/`, `find papr-runs/`,
or any filesystem exploration there.
Only read files explicitly listed in this prompt (paper .tex, main.pdf, figures/).
You only have access to the current paper -- no other run history exists for you.

## Communication (Agent Teams)
Message other panelists directly using SendMessage:
  SendMessage(target_name: "expert", content: "Your response to their point")
  SendMessage(type: "broadcast", content: "Message to all panelists")

## Workflow
1. READ the paper files (tool use required)
2. Broadcast your initial review to the whole team
3. Read others' reviews as they arrive; respond by name
4. Challenge disagreements, acknowledge strong points
5. When discussion converges, post your final score
6. AUTHOR: post the final action list when discussion winds down
7. ADVISOR: post synthesis + final scores when all reviews are in

## Scoring Calibration
At top venues, mean score is ~5.0/10. Only ~30% are accepted.
If you are unsure, score LOWER not higher. A 7+ means genuinely strong.
Assume most submissions are mediocre unless clearly exceptional.
This is a BLIND review. You know NOTHING about previous versions.

Use the Write tool to save your final review to:
  papr-runs/latest-run/latest/[role]_review.md
```

## Lead monitors and collects

The lead (this agent) monitors the team:
- Wait for all teammates to post final reviews
- If discussion stalls, broadcast: "Please post final scores."
- Collect all `[role]_review.md` files
- Merge into `DISCUSSION_THREAD.md`
- Write panel summary to `ROUND_STATE.md`
- **MANDATORY cleanup:** `TeamDelete(team_name: [team_name])` -- never skip this,
  even on error paths. Stale teams cause "Already leading team" failures next round.

## After Panel

Write to `papr-runs/latest-run/latest/ROUND_STATE.md`:
```
## Panel Summary
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
