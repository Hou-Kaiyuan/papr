
# Agent Communication Protocol

All agents in a multi-agent session communicate through a single append-only file:
`DISCUSSION_THREAD.md` in the working directory. Every agent reads the full thread
before writing and appends to it after. No agent passes messages directly to another.

## Message Format

```
---
FROM: [ROLE]
ROUND: [N]
WAVE: [1|2]
RE: [optional — e.g. RE: REVIEWER_EXPERT WAVE 1]
---

[content]

SIGNAL: DONE
```

FROM: ADVISOR | REVIEWER_EXPERT | REVIEWER_STANDARD | REVIEWER_BRIEF | REVIEWER_LAY | AUTHOR

## Waves

All agents run in parallel within each wave.

**Wave 1:** Independent perspectives. Each agent reads the paper and writes their
initial review. No cross-referencing other agents (thread may be empty or have
only pipeline context).

**Wave 2:** Cross-discussion. Each agent reads the full thread (all Wave 1 output)
and responds to specific points from other agents. AUTHOR posts action list.
ADVISOR synthesizes scores.


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
- [ ] description — raised by ROLE in WAVE W

## Resolved issues
- [x] description — resolved in WAVE W
```
