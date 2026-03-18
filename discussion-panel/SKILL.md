---
name: discussion-panel
description: Fully autonomous multi-agent paper review panel. Invoke as `/discussion-panel` to run a full session, `/discussion-panel score` to get scores only, or `/discussion-panel role <ROLE>` to run a single agent turn. All roles (Advisor, Expert Reviewer, Standard Reviewer, Lay Reviewer, Author) are agents — no human input during the panel. Outputs DISCUSSION_THREAD.md and a session summary with scores and action list.
---

# Discussion Panel

Runs a fully autonomous multi-agent discussion on a paper. All roles are agents.
The human sees only the final summary.

**Read `agent-protocol/SKILL.md` first.**

---

## Subcommands

| Invocation | Behavior |
|---|---|
| `/discussion-panel` | Full session — all 7 turns in order |
| `/discussion-panel score` | Skip discussion; each agent gives a score + top-3 concerns only |
| `/discussion-panel role ADVISOR` | Run only the ADVISOR turn (use any role name) |
| `/discussion-panel rebuttal` | Run only turns 6–7 (Expert rebuttal + Advisor close) |
| `/discussion-panel focus <topic>` | Full session with all agents focused on a specific topic |

---

## Setup (full session)

1. Read `agent-protocol/SKILL.md`
2. Read role definitions from `roles/` (see below)
3. Locate paper file (PDF or LaTeX source)
4. Create or append to `DISCUSSION_THREAD.md` and `ROUND_STATE.md`
5. If Round 2+, read previous round summary — agents must not re-raise resolved issues
6. Begin with ADVISOR's opening turn immediately

---

## Role Files

Load the appropriate file before running each agent's turn:

| Role | File | Reads |
|---|---|---|
| ADVISOR | `roles/advisor.md` | Main + appendix |
| REVIEWER_EXPERT | `roles/reviewer-expert.md` | Main + appendix |
| REVIEWER_STANDARD | `roles/reviewer-standard.md` | **Main text only** |
| REVIEWER_LAY | `roles/reviewer-lay.md` | Abstract + intro + figures |
| AUTHOR | `roles/author.md` | Full thread + paper |

---

## Autonomous Turn Sequence

Run all turns without stopping for human input:

1. **ADVISOR** — `SIGNAL: DONE`
2. **REVIEWER_EXPERT** — `SIGNAL: DONE`
3. **REVIEWER_STANDARD** — `SIGNAL: DONE`
4. **REVIEWER_LAY** — `SIGNAL: DONE`
5. **AUTHOR** — responds point by point, posts action list — `SIGNAL: DONE`
6. **REVIEWER_EXPERT** rebuttal — only if AUTHOR has unconvincing responses — `SIGNAL: DONE`
7. **ADVISOR** — synthesis + score — `SIGNAL: DONE`

---

## AUTHOR Action List Format

AUTHOR posts this at the end of their turn:

```
## Author Action List (Turn [T])

### Will fix — text only
- [ ] [specific change] — from [ROLE] TURN [T]

### Will fix — requires new experiment
- [ ] [description] → pass to experiment-designer

### Will clarify in text (not a real weakness)
- [ ] [description + why]

### Will not fix
- [ ] [description] — reason: [one sentence]
```

---

## Session End

After ADVISOR's closing turn:
1. Each agent posts score (1–10) + top-3 remaining concerns
2. Compute average → write to `ROUND_STATE.md`
3. Write session summary:

```
## Panel Summary — Round [N]
### Score: [avg]/10  (Advisor: X | Expert: Y | Standard: Z | Lay: W)

### Key issues raised
- [bullet]

### Author action list
[paste final action list]

### Passed to next phase
- Experiments needed: [N items]
- Text-only changes: [N items]
```

---

## Escalation

| Condition | Action |
|---|---|
| `SIGNAL: ESCALATE` | Surface to human in round summary before continuing |
| Experiment items in action list | Pass to `experiment-designer` |
| Text-only items | Pass to `paper-writer` |
