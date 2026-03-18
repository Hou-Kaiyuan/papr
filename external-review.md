
# External Reviewer (via Codex MCP)

Sends the finished paper to an external model for independent blind review.
Run once per round, on the final post-humanizer version.


## Prerequisites

Requires Codex MCP to be connected.

If unavailable:
> Note in `ROUND_STATE.md`: "External review skipped — Codex MCP not connected."
> Continue to Phase 8 (summary).


## Review Prompt (send verbatim)

```
You are a reviewer for a top-tier machine learning / computer vision conference (NeurIPS, ICLR, CVPR level). Review the following paper as you would for a formal submission.

Structure your review exactly as follows:

## Summary
[2–3 sentences: what the paper does and claims]

## Strengths
[Bulleted list — specific and concrete]

## Weaknesses
[Bulleted list — specific and concrete; both technical and presentation issues]

## Questions for the Authors
[Numbered list]

## Missing Comparisons or Baselines
[Methods the paper should compare against but does not]

## Presentation Issues
[Figures, tables, writing clarity, structure]

## Overall Score: [1–10]
1–3 = reject | 4–5 = borderline | 6–7 = weak accept | 8–9 = accept | 10 = strong accept

## Confidence: [1–5]
1 = outside my area | 5 = expert


## After Receiving the Review

1. Append full review to `ROUND_STATE.md` under `## External Review — Round [N]`
2. Compare with internal panel summary:

```
## External Review Comparison — Round [N]
Score: [X/10]   Confidence: [Y/5]

New issues not raised internally:
- [issue]

Confirmed by both internal and external:
- [issue]

Internal-only concerns (not flagged externally):
- [issue]  ← may indicate over-concern
```

3. New external issues → add to next round's open issues list
4. If external reviewer raises a BLOCKER-level finding → surface to human before starting next round
