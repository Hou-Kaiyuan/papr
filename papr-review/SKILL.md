---
name: papr-review
description: |
  External blind review of a paper via Codex MCP. Sends the paper to an
  independent AI model (not the one that edited it) for unbiased review.
  Compares external findings with internal panel results. Invoke as
  /papr-review [paper_dir].
allowed-tools:
  - Read
  - Write
  - Grep
  - Glob
  - mcp__codex__codex
  - mcp__codex__codex-reply
---

# External Blind Review

Sends the finished paper to Codex MCP for independent review.
Run after papr-write, on the final version.

## Prerequisites

Requires Codex MCP. If unavailable:
> Write to `[paper_dir]/.claude/latest/ROUND_STATE.md`: "External review skipped -- Codex MCP not connected."
> Done. Return.

## On invocation

1. Read `[paper_dir]/.claude/latest/ROUND_STATE.md` for internal panel scores.
2. Read the paper's .tex files. Strip author info (\author{} commands).
3. Send to Codex MCP with the review prompt below.
4. Append review to `[paper_dir]/.claude/latest/ROUND_STATE.md`.
5. Compare with internal panel.

## Strict Isolation

The external reviewer receives ONLY:
- Paper text (author info stripped)
- One neutral sentence describing the problem domain

**Never include:** round number, score history, change log, internal discussion,
or any framing suggesting this is a revised version.

## Review Prompt (send verbatim to Codex)

```
You are a reviewer for a top-tier ML/CV conference (NeurIPS, ICLR, CVPR level).
Review the following paper as a formal submission.

Structure your review as:

## Summary
[2-3 sentences]

## Strengths
[Bulleted, specific]

## Weaknesses
[Bulleted, specific]

## Questions for Authors
[Numbered]

## Missing Comparisons or Baselines
[Methods not compared against]

## Presentation Issues
[Figures, tables, writing, structure]

## Overall Score: [1-10]
1-3 = reject | 4-5 = borderline | 6-7 = weak accept | 8-9 = accept | 10 = strong accept

## Confidence: [1-5]
1 = outside my area | 5 = expert
```

## After Receiving Review

Append to `[paper_dir]/.claude/latest/ROUND_STATE.md`:

```
## External Review
Score: [X/10]  Confidence: [Y/5]

[full review text]

## Comparison with Internal Panel
New issues (not raised internally):
- [issue]

Confirmed by both:
- [issue]

Internal-only (not flagged externally):
- [issue]
```

If external reviewer raises a BLOCKER-level finding, note `SIGNAL: ESCALATE` in `[paper_dir]/.claude/latest/ROUND_STATE.md`.
