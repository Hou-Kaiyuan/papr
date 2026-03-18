# Phase 3: Panel

Invokes `discussion-panel` with the Round Briefing as context.

## Steps

1. Pass the Round Briefing (from Phases 1-2) to the discussion-panel as opening context
2. Run `/papr panel` — fully autonomous, 2 waves of parallel agents
3. Collect: session summary, average score, AUTHOR action list

## Passing Context

Open `DISCUSSION_THREAD.md` with a context block before Wave 1:

```
---
FROM: PIPELINE
ROUND: [N]
WAVE: 0
---

Round Briefing:
[paste Phase 1 + Phase 2 output here]

Previous round summary (if Round 2+):
[paste from ROUND_STATE.md]

SIGNAL: DONE
```

Then run `/papr panel` normally.

## Output

Panel writes its own session summary to `DISCUSSION_THREAD.md`.
Pipeline reads the summary and extracts:
- Average score
- AUTHOR action list (text-only vs. experiment items)
