# Phase 7: External Review

Invokes `external-review` via Codex MCP on the finished paper.

## Strict Isolation Rule

The external reviewer receives ONLY:
- Full paper text (LaTeX source or extracted PDF text, author info stripped)
- One neutral sentence describing the problem domain

**Never include:** round number, score history, change log, internal discussion,
author identity, or any framing suggesting this is a revised version.

## Steps

1. Prepare isolated prompt (see `external-review` skill for exact template)
2. Send via Codex MCP
3. Receive review
4. Append full review to `ROUND_STATE.md` under `## External Review — Round [N]`
5. Compare with internal panel summary:

```
## External Review Comparison — Round [N]

New issues (not raised internally):
- [issue]

Confirmed by both internal and external:
- [issue]

Internal-only concerns (not raised externally):
- [issue]  ← may indicate over-concern
```

6. New external issues → add to next round's action list

## If Codex MCP Unavailable

Note in `ROUND_STATE.md`: "External review skipped — Codex MCP not connected."
Pipeline continues to Phase 8.
