---
name: papr-experiment
description: |
  Design, verify, and run experiments from reviewer feedback. Use when the
  user says "design experiments", "run ablation", "need new results", or
  when papr-panel action list has "requires new experiment" items. Includes
  TDD verification and experiment monitoring.
argument-hint: "[paper_dir]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - Bash
  - WebSearch
  - Agent
---

# Experiment Designer + Verifier + Runner

## On invocation

1. Read `.claude/latest-run/latest/ROUND_STATE.md` for action list.
2. Extract "requires new experiment" items. If none: return "No experiments needed."

## Step 1: Check Before Designing

For each item, ask:
- Existing result already answers this? (better presentation, not new experiment)
- In appendix but not main text? (move up, don't re-run)
- Existing data re-analyzable? (subset, different metric)

Only design if none apply.

## Step 2: Design

```
### Experiment: [Name]
Motivation: [claim addressed]
Priority: BLOCKER | HIGH | MEDIUM | LOW
Metric: [name, direction]
Conditions: Ours vs Baseline A [verify SOTA via WebSearch] vs Baseline B
Dataset: [name, split, preprocessing]
Expected: if claim holds [numbers] / if fails [what we'd see]
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

## Step 5: Monitor

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

## Output

Write to `.claude/latest-run/latest/ROUND_STATE.md`:
```
## Experiment Plan
1. [Name] -- [Priority] -- [~N GPU-hours] -- Verification: PASSED|BLOCKED
Parallel: [list]
Total compute: ~[N] GPU-hours
CRITICAL issues: [N]
```
