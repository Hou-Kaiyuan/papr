---
name: papr-panel
description: |
  Multi-agent paper review panel. 6 agents (Advisor, Expert, Standard, Brief,
  Lay, Author) review a paper in parallel via 2 waves. Each role has built-in
  audit checklists (format, storyline, baselines, accessibility). Produces
  scores and an action list. Invoke as /papr-panel [paper_dir].
allowed-tools:
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - Agent
  - WebSearch
  - WebFetch
---

# Multi-Agent Paper Review Panel

6 agents review the paper in parallel. Two waves: independent reviews, then cross-discussion.
Each agent writes to its own temp file to avoid race conditions.

## On invocation

IMPORTANT: Do NOT explore the filesystem. Do NOT run `ls` or `bash test`. Do NOT read all .tex files upfront. The agents will read the paper themselves.

1. Read `[paper_dir]/DISCUSSION_THREAD.md` (ignore error if not found).
2. Read `[paper_dir]/ROUND_STATE.md` (ignore error if not found).
3. Immediately start Wave 1 below. Do NOT read the paper .tex files yourself -- each agent reads them in their own context.

## Wave 1: Independent Reviews (all in parallel)

Spawn ALL 6 agents using Agent tool with `run_in_background: true`.
Each agent writes to its own temp file in `[paper_dir]/`:

```
Agent(ADVISOR)           -> wave1_advisor.md
Agent(REVIEWER_EXPERT)   -> wave1_expert.md
Agent(REVIEWER_STANDARD) -> wave1_standard.md
Agent(REVIEWER_BRIEF)    -> wave1_brief.md
Agent(REVIEWER_LAY)      -> wave1_lay.md
Agent(AUTHOR)            -> wave1_author.md
```

Wait for all to complete. Merge all `wave1_*.md` into `DISCUSSION_THREAD.md`. Delete temp files.

## Wave 2: Cross-Discussion (all in parallel)

Spawn ALL 6 agents again. Each reads `DISCUSSION_THREAD.md` and writes to:

```
Agent(ADVISOR)           -> wave2_advisor.md     (synthesize + scores)
Agent(REVIEWER_EXPERT)   -> wave2_expert.md      (challenge defenses)
Agent(REVIEWER_STANDARD) -> wave2_standard.md    (cross-cutting issues)
Agent(REVIEWER_BRIEF)    -> wave2_brief.md       (main-text gaps)
Agent(REVIEWER_LAY)      -> wave2_lay.md         (clarity responses)
Agent(AUTHOR)            -> wave2_author.md      (respond + action list)
```

Wait for all. Merge into `DISCUSSION_THREAD.md`. Delete temp files.

## Agent Prompt Template

```
You are [ROLE] in an academic paper review panel.

## Your Role
[paste from roles/[role].md]

## Paper
[paste paper content, scoped to role's reading scope]

## Discussion So Far
[DISCUSSION_THREAD.md content, or "Empty" for Wave 1]

## Task (Wave [1|2])
Wave 1: Write your initial review including all checklists from your role.
Wave 2: Read others' comments and respond. Address points by name.

## Output
Write to [paper_dir]/wave[N]_[role].md using format:
---
FROM: [ROLE]
WAVE: [1|2]
---
[review]
SIGNAL: DONE
```

## After Wave 2

1. Extract scores from each reviewer
2. Compute average score
3. Write to `[paper_dir]/ROUND_STATE.md`:
```
## Panel Summary
### Score: [avg]/10 (Advisor: X | Expert: Y | Standard: Z | Brief: B | Lay: W)
### Key issues
- [bullet]
### Author action list
[paste from AUTHOR wave 2]
```

## Roles

| Role | File | Reads | Focus |
|---|---|---|---|
| ADVISOR | `roles/advisor.md` | Full paper | Storyline, figures, captions, narrative arc |
| EXPERT | `roles/reviewer-expert.md` | Full paper | Technical, baselines (web search), citations, notation |
| STANDARD | `roles/reviewer-standard.md` | Full paper | Overall quality, structure, novelty |
| BRIEF | `roles/reviewer-brief.md` | Main only | Self-containedness |
| LAY | `roles/reviewer-lay.md` | Full paper | Accessibility, jargon, caption clarity |
| AUTHOR | `roles/author.md` | Full + thread | Defend paper, post action list |
