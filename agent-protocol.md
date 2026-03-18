
# Agent Communication Protocol

All agents in a multi-agent session communicate through a single append-only file:
`DISCUSSION_THREAD.md` in the working directory. Every agent reads the full thread
before writing and appends to it after. No agent passes messages directly to another.

To avoid race conditions, parallel agents write to individual temp files
(`wave[N]_[role].md`). The orchestrator merges them into `DISCUSSION_THREAD.md`
after all agents in a wave complete.

## Message Format

```
---
FROM: [ROLE]
ROUND: [N]
WAVE: [0|1|2]
RE: [optional, e.g. RE: REVIEWER_EXPERT WAVE 1]
---

[content]

SIGNAL: DONE
```

FROM: PIPELINE | ADVISOR | REVIEWER_EXPERT | REVIEWER_STANDARD | REVIEWER_BRIEF | REVIEWER_LAY | AUTHOR

## Waves

**Wave 0 (pipeline only):** Context injection. When the panel runs inside a pipeline,
the orchestrator writes a Wave 0 message with `FROM: PIPELINE` containing the Round
Briefing (scout + inspect output) and previous round summary. Agents should read this
as background context but not respond to it directly.

**Wave 1:** Independent perspectives. All agents spawn in parallel. Each reads the
paper (scoped to their role) and writes their initial review. Each agent writes to
its own temp file (`wave1_[role].md`). Orchestrator merges into `DISCUSSION_THREAD.md`
after all complete.

**Wave 2:** Cross-discussion. All agents spawn in parallel again. Each reads the full
`DISCUSSION_THREAD.md` (now containing all Wave 1 output) and responds to specific
points from other agents. AUTHOR posts action list. ADVISOR synthesizes scores.
Each writes to `wave2_[role].md`. Orchestrator merges after all complete.

## Signals

| Signal | Meaning | When to use |
|---|---|---|
| `SIGNAL: DONE` | Agent has finished their turn | End of every agent message |
| `SIGNAL: ESCALATE` | Issue requires human attention | See escalation triggers below |

## Escalation Triggers

An agent should emit `SIGNAL: ESCALATE` when:

- A claim in the paper appears to be factually incorrect and cannot be verified
- Experimental results seem inconsistent with the method described (possible error)
- Ethical concerns are identified (data usage, attribution, dual-use risks)
- A fundamental flaw is found that cannot be fixed with text changes or new experiments
- Two or more reviewers score below 4/10 with unresolvable disagreements

The orchestrator surfaces ESCALATE issues to the human before continuing to the next phase.

## ROUND_STATE.md Format

```markdown
# Round [N] State
## Paper: [filename]
## Session started: [timestamp]
## Score: [X/10] (Advisor: A | Expert: E | Standard: S | Brief: B | Lay: L)

## Changes this round
- [text changes]
- [new experiments]
- [figures/tables updated]

## Open issues
- [ ] description -- raised by ROLE in WAVE W

## Resolved issues
- [x] description -- resolved in WAVE W
```
