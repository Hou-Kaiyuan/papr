---
name: papr-panel
description: |
  Run a 6-agent parallel paper review panel. Use when the user says "review
  my paper", "run panel", "get feedback", or wants multi-perspective critique.
  Agents: Advisor (storyline), Expert (technical+baselines), Standard (quality),
  Brief (main-text only), Lay (accessibility), Author (defense+action list).
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

Two waves, all agents in parallel. Each agent writes to its own temp file (no race conditions).

**Scoring rule:** Each round scored INDEPENDENTLY. Do NOT show agents previous scores.

## On invocation

1. Read `.claude/latest-run/latest/ROUND_STATE.md` (ignore error if missing). Do NOT pass previous scores to agents.
2. Write fresh empty `.claude/latest-run/latest/DISCUSSION_THREAD.md`.
3. Start Wave 1.

## Wave 1: Independent Reviews

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

## Wave 2: Cross-Discussion

Same agents, same pattern with `wave2_*.md`. Each reads full `DISCUSSION_THREAD.md` first. AUTHOR posts action list. ADVISOR synthesizes. Merge and delete.

## Agent Prompt

```
You are [ROLE] reviewing an academic paper.

[paste roles/[role].md]

Paper location: [paper_dir]
- Read .tex source files for text
- Read main.pdf for visual layout
- Read figures/ directory for individual figure inspection
You MUST look at actual figures, not just LaTeX code.

Discussion: [DISCUSSION_THREAD.md content, or "Empty" for Wave 1]

Score independently. Do NOT reference previous rounds. Evaluate as a fresh submission.

Wave [1|2]: [Wave 1: initial review with checklists | Wave 2: respond to others by name]

Output to: .claude/latest-run/latest/wave[N]_[role].md
Format: FROM: [ROLE] / WAVE: [N] / [review] / SIGNAL: DONE
```

## After Wave 2

Write to `.claude/latest-run/latest/ROUND_STATE.md`:
```
## Panel Summary
### Score: [avg]/10 (Advisor: X | Expert: Y | Standard: Z | Brief: B | Lay: W)
### Key issues
- [bullet]
### Author action list
[from AUTHOR wave 2]
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
