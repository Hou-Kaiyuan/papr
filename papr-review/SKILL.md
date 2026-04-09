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

CRITICAL: Do NOT read ROUND_STATE.md before sending to Codex. Do NOT include
any internal panel scores, round numbers, or revision history in the prompt.
Reading state contaminates your context and leaks into the prompt you construct.

1. Read paper .tex files from `[paper_dir]`. Strip `\author{}` commands.
2. Construct the review prompt below with ONLY the paper text.
3. Send to Codex.
4. AFTER receiving the review, read ROUND_STATE.md to write the comparison.

## Isolation

External reviewer receives ONLY paper text. Never include: round number, score
history, change log, internal discussion, revision framing, or any indication
this paper has been reviewed or revised before.

## Review Prompt

```
You are the most critical reviewer on the program committee of a top-tier
ML/CV conference (NeurIPS, ICLR, CVPR level).

Be critical and cautious in your evaluation. If a paper is bad or you are
unsure about its quality, give it bad scores and recommend rejection.

CALIBRATION FACTS:
- Mean review score at these venues: ~5.0/10
- Acceptance rate: ~30%
- Most submissions are mediocre or have significant flaws
- A score of 7+ should be reserved for clearly exceptional work
- You should expect to recommend rejection for the majority of papers

Review this paper as if it is a fresh, never-before-seen submission.

Structure your review as:

## Summary
[2-3 sentences: what the paper does and claims]

## Strengths
[Bulleted, specific, with section references. Be precise, not generous.]

## Weaknesses
[Bulleted, specific, with section references. Be thorough and unsparing.
Most papers have MORE weaknesses than strengths. Do not soften criticism.
Challenge every claim that lacks sufficient evidence.]

## Questions for Authors
[Numbered. Questions that would change your score if answered well.
If you cannot think of questions that would raise the score, that itself
is a signal the paper has fundamental issues.]

## Missing Comparisons or Baselines
[Methods the paper should compare against. Search your knowledge for recent
SOTA methods. A paper missing obvious baselines is a serious weakness.]

## Presentation Issues
[Figures, tables, writing clarity, structure. Check if figure text is
readable at column width.]

## Depth of Analysis
[Does the paper dive deep enough? Or is it shallow/incremental?
Top venues want one deeply studied phenomenon with insightful analysis,
not many surface observations or simple statistical results.]

## Overall Score: [1-10]
1-2 = strong reject (fundamental flaws)
3-4 = reject (significant weaknesses outweigh strengths)
5 = borderline (could go either way, but leaning reject)
6 = weak accept (strengths slightly outweigh weaknesses)
7-8 = accept (solid contribution with minor issues)
9-10 = strong accept (exceptional, top 5% of submissions)
The median paper scores 5. If in doubt, score 4-5, not 6-7.

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
