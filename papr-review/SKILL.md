---
name: papr-review
description: |
  Send a paper for external blind review via Codex plugin (GPT-5.4). Use when
  the user says "external review", "get another opinion", "blind review", or
  after papr-write completes. Compares external findings with internal panel.
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

# External Blind Review (via Codex / GPT-5.4)

Sends finished paper to Codex for independent review using GPT-5.4.

## Prerequisites

Codex plugin must be installed:
```
/plugin marketplace add openai/codex-plugin-cc
/plugin install codex@openai-codex
/codex:setup
```

If Codex unavailable: write "External review skipped -- Codex not available." to `ROUND_STATE.md` and return.

## On invocation

1. Read `.claude/latest-run/latest/ROUND_STATE.md` for internal panel scores.
2. Read paper .tex files from `[paper_dir]`. Strip `\author{}` commands.
3. Send to Codex with review prompt below.

## Isolation

External reviewer receives ONLY paper text + one sentence about the domain. Never include: round number, score history, change log, internal discussion, or revision framing.

## Review Prompt

```
You are a reviewer for a top-tier ML/CV conference (NeurIPS, ICLR, CVPR level).
Review this paper as a formal submission.

IMPORTANT CALIBRATION: At these venues, the mean review score is approximately
5.0/10 and only about 30% of submissions are accepted. Be critical and cautious.
If the paper is weak or you are unsure, give it low scores. Only clearly
exceptional work deserves 7+. Assume most submissions are mediocre unless the
evidence is compelling.

Structure your review as:

## Summary
[2-3 sentences: what the paper does and claims]

## Strengths
[Bulleted, specific, with section references]

## Weaknesses
[Bulleted, specific, with section references. Be thorough -- most papers have
significant weaknesses. Do not soften criticism.]

## Questions for Authors
[Numbered. Include questions that would change your score if answered well.]

## Missing Comparisons or Baselines
[Methods the paper should compare against. Search your knowledge for recent
SOTA methods in this area.]

## Presentation Issues
[Figures, tables, writing clarity, structure]

## Depth of Analysis
[Does the paper dive deep enough? Or is it shallow/incremental?
Top venues want deeply studied phenomena, not many surface observations.]

## Overall Score: [1-10]
1-2 = strong reject | 3-4 = reject | 5 = borderline | 6 = weak accept |
7-8 = accept | 9-10 = strong accept
Most papers should fall in the 4-6 range.

## Confidence: [1-5]
1 = outside my area | 3 = somewhat familiar | 5 = expert

## Decision: [Accept / Borderline / Reject]
```

## Output

Append to `.claude/latest-run/latest/ROUND_STATE.md`:
```
## External Review (GPT-5.4 via Codex)
Score: [X/10]  Confidence: [Y/5]  Decision: [Accept/Borderline/Reject]
[full review]

## Comparison
New issues (not raised internally): [list]
Confirmed by both: [list]
Internal-only (not flagged externally): [list]
```

If BLOCKER-level finding: note `SIGNAL: ESCALATE`.
