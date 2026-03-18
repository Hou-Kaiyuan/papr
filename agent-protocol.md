
# Agent Communication Protocol

All agents in a multi-agent session communicate through a single append-only file:
`DISCUSSION_THREAD.md` in the working directory. Every agent reads the full thread
before each turn and appends to it after. No agent passes messages directly to another.

FROM: [ROLE]         # ADVISOR | REVIEWER_EXPERT | REVIEWER_STANDARD | REVIEWER_LAY | AUTHOR
ROUND: [N]
TURN: [T]
RE: [optional]       # e.g. RE: TURN 3

## Default Turn Order

1. ADVISOR
2. REVIEWER_EXPERT
3. REVIEWER_STANDARD
4. REVIEWER_LAY
5. AUTHOR (responds to all, posts action list)
6. REVIEWER_EXPERT (rebuttal round)
7. ADVISOR (closes, gives score)


## ROUND_STATE.md Format

```markdown
# Round [N] State
## Paper: [filename]
## Session started: [timestamp]
## Anticipated score: [X/10]

## Changes this round
- [text changes]
- [new experiments]
- [figures/tables updated]

## Open issues
- [ ] description — raised by ROLE at TURN T

## Resolved issues
- [x] description — resolved at TURN T
```

