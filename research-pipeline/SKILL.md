---
name: research-pipeline
description: Full autonomous paper improvement pipeline. Invoke as `/research-pipeline start` to begin a new pipeline, `/research-pipeline round` to run the next round, `/research-pipeline status` to see current state, or `/research-pipeline <phase>` to run a single phase (scout, inspect, panel, experiments, write, humanize, review, summary). Default 3 rounds. Each round builds on the prior round's summary.
---

# Research Pipeline

End-to-end autonomous paper improvement. Default: 3 rounds.
Each round ends with a score /10 and a summary of what changed.
The human is only consulted between rounds (to approve continuing) and for ESCALATE issues.

**Read `agent-protocol/SKILL.md` before starting.**

---

## Subcommands

| Invocation | Behavior |
|---|---|
| `/research-pipeline start` | Initialize state files, begin Round 1 |
| `/research-pipeline round` | Run the next full round (reads current state) |
| `/research-pipeline status` | Print `ROUND_STATE.md` summary and current score |
| `/research-pipeline scout` | Run Phase 1 only (related-work-scout) |
| `/research-pipeline inspect` | Run Phase 2 only (quality + storyline) |
| `/research-pipeline panel` | Run Phase 3 only (discussion-panel) |
| `/research-pipeline experiments` | Run Phase 4 only (if experiments needed) |
| `/research-pipeline write` | Run Phase 5 only (paper-writer) |
| `/research-pipeline humanize` | Run Phase 6 only (humanizer) |
| `/research-pipeline review` | Run Phase 7 only (external-reviewer) |
| `/research-pipeline summary` | Run Phase 8 only (write round summary) |

---

## Initialization (`start`)

1. Read `agent-protocol/SKILL.md`
2. Confirm paper location and number of rounds (default: 3)
3. Create `ROUND_STATE.md`:

```markdown
# Research Pipeline State
## Paper: [filename]
## Rounds planned: [N]
## Started: [timestamp]
## Rounds completed: 0
## Score history: []
```

4. Create empty `DISCUSSION_THREAD.md`
5. Proceed immediately to Round 1

---

## Per-Round Phases

See detailed instructions in each phase file:

| Phase | File | Skill used |
|---|---|---|
| 1. Scout | `phases/scout.md` | `related-work-scout` |
| 2. Inspect | `phases/inspect.md` | `paper-quality-inspect` + `paper-storyline` |
| 3. Panel | `phases/panel.md` | `discussion-panel` (autonomous) |
| 4. Experiments | `phases/experiments.md` | `experiment-designer` + `code-inspector` |
| 5. Write | `phases/write.md` | `paper-writer` |
| 6. Humanize | `phases/humanize.md` | `humanizer` |
| 7. External Review | `phases/review.md` | `external-reviewer` (Codex MCP) |
| 8. Summary | `phases/summary.md` | writes to `ROUND_STATE.md` |

---

## Round Continuity

**Round 2+ only:** Before Phase 1, prepend the previous round's summary from
`ROUND_STATE.md` to the context. All phases should avoid re-raising issues
already marked resolved in that summary.

---

## Stopping Conditions

Stop early if:
- Score reaches 9+ → paper is ready
- `SIGNAL: ESCALATE` unresolved after external review → surface to human
- User invokes `/research-pipeline status` and declines to continue

---

## Score Reference

| Score | Interpretation |
|---|---|
| 9–10 | Strong accept — ready for top venue |
| 7–8 | Weak accept — minor revisions |
| 5–6 | Borderline — significant work remains |
| 3–4 | Weak reject — major revision needed |
| 1–2 | Reject — fundamental issues |
