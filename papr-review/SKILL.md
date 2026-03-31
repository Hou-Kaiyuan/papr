---
name: papr-review
description: |
  Send a paper for external blind review via Codex MCP. Use when the user
  says "external review", "get another opinion", "blind review", or after
  papr-write completes. Compares external findings with internal panel scores.
argument-hint: "[paper_dir]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - Bash
  - mcp__codex__codex
  - mcp__codex__codex-reply
---

# External Blind Review

Sends finished paper to Codex MCP for independent review.

## Prerequisites

If Codex MCP unavailable: write "External review skipped -- Codex MCP not connected." to `ROUND_STATE.md` and return.

## On invocation

1. Read `.claude/latest-run/latest/ROUND_STATE.md` for internal panel scores.
2. Read paper .tex files from `[paper_dir]`. Strip `\author{}` commands.
3. Send to Codex with review prompt below.

## Isolation

External reviewer receives ONLY paper text + one sentence about the domain. Never include: round number, score history, change log, internal discussion, or revision framing.

## Review Prompt

```
You are a reviewer for a top-tier ML/CV conference (NeurIPS, ICLR, CVPR).
Review as a formal submission.

## Summary
[2-3 sentences]
## Strengths
[Bulleted, specific]
## Weaknesses
[Bulleted, specific]
## Questions for Authors
[Numbered]
## Missing Comparisons or Baselines
## Presentation Issues
## Overall Score: [1-10]
1-3 = reject | 4-5 = borderline | 6-7 = weak accept | 8-9 = accept | 10 = strong accept
## Confidence: [1-5]
```

## Output

Append to `.claude/latest-run/latest/ROUND_STATE.md`:
```
## External Review
Score: [X/10]  Confidence: [Y/5]
[full review]

## Comparison
New issues (not raised internally): [list]
Confirmed by both: [list]
Internal-only (not flagged externally): [list]
```

If BLOCKER-level finding: note `SIGNAL: ESCALATE`.
