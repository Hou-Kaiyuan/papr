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
  - Edit
  - Grep
  - Glob
  - Bash
  - WebSearch
  - Agent
---

# Experiment Designer + Verifier

Takes "requires new experiment" items from the action list, designs experiments,
and verifies correctness before execution.

**Block execution if any CRITICAL issue is found. Fix first, then re-verify.**

## On invocation

1. Read `.claude/latest-run/latest/ROUND_STATE.md` for the action list.
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

## Step 4: Test Before Running (TDD)

Before running any experiment (especially time-consuming ones):

1. **Write a simple test case first.** Create a minimal input that should produce a known output. Run it to verify the code path works.
2. **Dry run with tiny data.** Run the full pipeline on a small subset (e.g. 10 samples, 1 epoch) to check:
   - Code runs without errors
   - Output format is correct
   - Metrics are computed and logged
   - No OOM or resource issues
3. **Only after dry run passes:** launch the full experiment.

```bash
# Example dry run
python train.py --data subset_10 --epochs 1 --dry-run 2>&1 | tail -20
```

If the dry run fails, fix the issue and re-run. Do NOT launch full experiments without a passing dry run.

## Step 5: Monitor Running Experiments

Do NOT launch an experiment and leave. Monitor it:

```bash
# Watch training progress (check every 60 seconds)
while true; do
    echo "=== $(date) ==="
    tail -5 [log_file]
    sleep 60
done
```

Or use the Bash tool with `run_in_background: true` to launch, then periodically check:
- Is loss decreasing?
- Are metrics being logged?
- Any NaN or error in output?
- GPU utilization normal?

If the experiment stalls, errors, or shows NaN loss, stop it and diagnose before restarting.

## Step 6: Figure & Visualization Design

When experiments produce results that need figures:

### Style Rules
- Use a consistent color palette (e.g. seaborn, matplotlib default)
- NO titles on figures; the caption provides the title
- Legends must NOT overlay on data; place outside or in empty space
- Legend font size should match body text after embedding (~9-10pt)
- Use hatching patterns on bars/areas for B&W printing readability
- Axis labels and tick labels must be readable at column width

### Information Density (conference papers have limited space)
- Do NOT just translate a table into a bar plot; that wastes space and adds no new information
- Prefer visualizations that show relationships, trends, or distributions that tables cannot
- Good figure types: scatter plots with trend lines, heatmaps, t-SNE/UMAP embeddings, qualitative comparisons, ablation curves, attention maps, error analysis grids
- Each figure should deliver information that is hard to convey in a table
- If a result is extraordinarily good, visualize it in a way that makes the impact obvious (e.g. qualitative comparison showing dramatic improvement, not just a number)
- Combine multiple related results into one figure with sub-panels rather than separate figures

### When to use tables vs figures
- Tables: precise numerical comparisons across many methods/metrics
- Figures: trends, distributions, qualitative results, spatial patterns, anything visual
- Do NOT duplicate: if a table shows the numbers, the figure should show something different (e.g. qualitative examples, error cases, ablation curves)

## Step 7: Parallelism

List which experiments can run simultaneously for GPU scheduling.

## Output

Write to `.claude/latest-run/latest/ROUND_STATE.md`:
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
