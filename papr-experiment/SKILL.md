---
name: papr-experiment
description: |
  Designs, verifies, runs, and monitors experiments from reviewer feedback,
  with TDD dry-runs and figure generation. Use when the user says "design
  experiments", "run ablation", "need new results", or when a papr-panel action
  list contains "requires new experiment" items.
argument-hint: "[paper_dir]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - Bash
  - WebSearch
  - WebFetch
  - Agent
---

# Experiment Designer + Verifier + Runner

IMPORTANT: Top venue reviewers can tell the difference between "we reframed the
text" and "we actually ran new experiments." Text-only improvements plateau fast.
New experimental evidence is what moves a paper from borderline to accept.

Do NOT default to "existing data can answer this" unless it genuinely can with
a concrete analysis you can point to. Avoid the lazy path of reframing text
when the real fix is a new experiment.

## DO NOT add these as "deep analysis" (they are SHALLOW)

Do NOT add the following statistical tests as a substitute for real analysis:
- Spearman/Pearson correlation
- Cohen's d, Cohen's kappa
- Wilson confidence intervals
- Bootstrap CI / bootstrapped error bars
- t-tests, paired/unpaired
- McNemar's test
- Fisher's exact test
- Chi-squared test
- p-values and "statistically significant (p<0.05)" framing
- ANY "we computed X coefficient/test" without it being central to the claim
- ANY sentence whose main point is "the difference is significant" rather than
  "the difference is X percentage points and is caused by Y mechanism"

These are the SHALLOW analyses reviewers complain about. Adding them ruins paper
structure and signals the authors don't have a real story. Top-venue reviewers
read "we ran McNemar's test, p=0.03" as a substitute for actual insight, not as
evidence of rigor. They belong in appendices ONLY if absolutely necessary for
one specific claim, and even then they should not be the primary contribution.

## What "deep analysis" actually means

Deep analysis means understanding WHY the method works:
- Failure case studies: when does the method fail? Why?
- Ablations that isolate which component contributes what
- Qualitative analysis of model behavior (attention, embeddings, mistakes)
- Comparison of HOW (not just IF) different methods solve the same problem
- Discovery of unexpected behavior and explanation of the mechanism
- Visualization of the learned representation/decision boundary

Example: "Our method outperforms X by 5%" is a number. Deep analysis is "X fails
on long-tail cases because Y; we visualize this in Figure 3 and confirm with
ablation Z that removing the long-tail component drops performance to baseline."

## On invocation

1. Read `papr-runs/latest-run/latest/ROUND_STATE.md` for action list.
2. Read `papr-runs/latest-run/latest/DISCUSSION_THREAD.md` for full reviewer feedback.
3. Extract ALL items where reviewers raised:
   - Explicitly marked "requires new experiment"
   - Weak experimental evidence for a claim
   - Missing ablations
   - Unsupported assumptions
   - Requests for additional baselines or comparisons
   - "Why does this work?" without a satisfying answer in the paper
4. If genuinely nothing needs experiments: return "No experiments needed."
   But be honest -- most papers reviewed at top venues DO need more experiments.

## Step 1: Triage

For each item, classify honestly:
- **MUST experiment:** The claim has no supporting evidence without this
- **SHOULD experiment:** Evidence exists but is weak; a new experiment would be convincing
- **CAN skip:** Existing data genuinely answers this (cite the specific table/figure)

Default to MUST or SHOULD. Only classify as CAN skip if you can point to a
specific existing result that directly addresses the reviewer's concern.

## Step 2: Design

```
### Experiment: [Name]
Motivation: [claim addressed]
Priority: BLOCKER | HIGH | MEDIUM | LOW
Impact: [what accepting this paper depends on this experiment?]
Metric: [name, direction]
Conditions: Ours vs Baseline A [verify SOTA via WebSearch] vs Baseline B
Dataset: [name, split, preprocessing]
Expected: if claim holds [numbers] / if fails [what we'd see, and what
  that means for the paper -- be honest about this]
Compute: ~[N] GPU-hours
```

## Step 3: Verify (Code Inspection)

Run manual checklist AND Codex adversarial review if available:

```
# If Codex plugin installed, use it for adversarial code review:
/codex:adversarial-review
# This challenges design decisions and catches subtle bugs
```

### Manual checklist:
- [ ] Script tests the exact paper claim (quote it)
- [ ] Metric corresponds to the claim (X!=Y -> CRITICAL)
- [ ] Train/val/test splits disjoint, no test leakage
- [ ] Pretrained model not trained on eval set
- [ ] Random seed fixed
- [ ] Baseline preprocessing identical to ours
- [ ] Compute budgets comparable
- [ ] Metric matches paper definition, edge cases handled
- [ ] No data contamination

| Pitfall | Status |
|---|---|
| Eval subset sorted by difficulty | |
| Our output post-processed, baseline's not | |
| Metric saturates on easy examples | |
| Data contamination | |

## Step 4: Test Before Running (TDD)

1. Write a simple test case with known expected output. Run it.
2. Dry-run full pipeline on tiny data (10 samples, 1 epoch):
```bash
python train.py --data subset_10 --epochs 1 --dry-run 2>&1 | tail -20
```
3. Only after dry run passes: launch full experiment.

## Step 5: Run and Monitor

Do NOT launch and leave. Monitor:
```bash
while true; do echo "=== $(date) ==="; tail -5 [log_file]; sleep 60; done
```
Check: loss decreasing? Metrics logged? NaN? GPU utilization?
If stalled or NaN: stop, diagnose, fix, restart.

If debugging is difficult, delegate to Codex:
```
/codex:rescue "experiment [name] is producing NaN loss after epoch 3, diagnose and fix"
```

Long-running experiments are expected and acceptable. A 2-hour training run that
produces a convincing ablation is worth more than 10 text-only edits.

## Step 6: Figures

When experiments produce figures:
- Consistent color palette, no titles, legends outside data
- Hatching for B&W, font sizes matching body text
- Do NOT make bar plots restating table numbers
- Show trends, distributions, qualitative comparisons, attention maps
- Maximize info density; combine related results into multi-panel figures
- Tables for precise numbers, figures for what tables cannot show

## Step 7: Parallelism

List which experiments can run simultaneously.

## Step 8: New citations from this round

If experiments introduce new baselines, datasets, or methods that the paper now
cites, follow the same bibtex-download workflow as papr-write: WebSearch the
paper, download the actual `.bib` file (arXiv `/bibtex/<id>`, ACL Anthology
BibTeX export, OpenReview, or Semantic Scholar API), and save it to
`[paper_dir]/references/<citekey>.bib`. Never fabricate a bibtex entry from
memory -- if you cannot locate the paper, drop the comparison or the claim.

## Output

Write to `papr-runs/latest-run/latest/ROUND_STATE.md`:
```
## Experiment Results
### Completed:
1. [Name] -- [metric result] -- [supports/contradicts claim]
### Skipped (with justification):
1. [Name] -- [why: specific existing result that answers this]
### Figures generated: [list]
### New tables: [list]
```
