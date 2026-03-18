---
name: papr-experiment
description: |
  Design and verify experiments from reviewer feedback. Takes action list items
  marked "requires new experiment", checks if existing data can answer them,
  designs experiments with verification checklists, and produces a prioritized
  plan. Invoke as /papr-experiment [paper_dir].
allowed-tools:
  - Read
  - Write
  - Grep
  - Glob
  - WebSearch
  - Bash
---

# Experiment Designer + Verifier

Takes "requires new experiment" items from the action list, designs experiments,
and verifies correctness before execution.

**Block execution if any CRITICAL issue is found. Fix first, then re-verify.**

## On invocation

1. Read `[paper_dir]/.claude/latest/ROUND_STATE.md` for the action list.
2. Extract items marked "requires new experiment".
3. If none: output "No experiments needed." and return.
4. For each item, run through the steps below.

## Step 1: Check Before Designing

- Does an existing result already answer this? (May just need better presentation)
- Is this in the appendix but not main text? (Move it up, don't re-run)
- Can existing data be re-analyzed? (Subset analysis, different metric)

Only design if none apply.

## Step 2: Design Each Experiment

```
### Experiment: [Name]
Motivation: [which claim this addresses]
Priority: BLOCKER | HIGH | MEDIUM | LOW

What to measure:
  Primary metric: [name, direction]
  Secondary metrics: [if any]

Conditions:
  Ours: [description]
  Baseline A: [verify SOTA via web search]
  Baseline B: [description]

Dataset: [name, split, preprocessing]

Expected outcome:
  If claim holds: [expected numbers]
  If claim fails: [what we'd see, what it means]

Compute: ~[N] GPU-hours
```

## Step 3: Verify (Code Inspection)

### Claim-to-Code Mapping
- [ ] Script tests the exact paper claim (quote it)
- [ ] Output metric corresponds to that claim
- [ ] Claim about X but metric measures Y -> **CRITICAL**

### Data Integrity
- [ ] Train/val/test splits disjoint
- [ ] Test data not used for hyperparameter selection
- [ ] Pretrained model not trained on eval set
- [ ] Random seed fixed

### Baseline Fairness
- [ ] Identical preprocessing for baseline and ours
- [ ] Same compute budget (or disclosed)
- [ ] Baseline hyperparameters tuned or justified
- [ ] No unintentional advantage for our method

### Metric Implementation
- [ ] Metric matches paper definition exactly
- [ ] Edge cases handled (empty, NaN, zero-division)
- [ ] Aggregation correct (mean/median, macro/micro)

### Evaluation Pitfalls
| Pitfall | Status |
|---|---|
| Eval subset sorted by difficulty (not random) | |
| Our output post-processed, baseline's not | |
| Metric saturates on easy examples | |
| Data contamination (test seen in training) | |

## Step 4: Parallelism

List which experiments can run simultaneously for GPU scheduling.

## Output

Write to `[paper_dir]/.claude/latest/ROUND_STATE.md`:
```
## Experiment Plan
### Execution order:
1. [Name] -- [Priority] -- [~N GPU-hours] -- Verification: PASSED|BLOCKED
2. ...

### Parallel: [list]
### Total compute: ~[N] GPU-hours
### CRITICAL issues: [N]
### Ready to run: [list]
```
