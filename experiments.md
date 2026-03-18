
# Experiment Designer

Takes reviewer feedback, weak claims, or action list items marked "requires new experiment"
and produces a concrete, prioritized plan with built-in code verification.

Every designed experiment is verified before execution. No separate code-inspect step needed.

**Block execution if any CRITICAL issue is found. Fix first, then re-verify.**


## Step 1: Check Before Designing

Before designing a new experiment, ask:
- Does an existing result already answer this? (May just need better presentation)
- Is this addressed in the appendix but not the main text? (Move it up, don't re-run)
- Can existing data be re-analyzed differently to answer this? (Subset analysis, different metric)

Only proceed to design if none of the above apply.


## Step 2: Design Each Experiment

```
### Experiment: [Name]
Motivation: [Which claim or TURN number this addresses]
Category: [from table above]
Priority: BLOCKER | HIGH | MEDIUM | LOW

What to measure:
  Primary metric: [name, direction up/down]
  Why this metric: [one sentence]
  Secondary metrics: [if any]

Conditions:
  Ours: [description]
  Baseline A: [verify current SOTA via web search]
  Baseline B: [description]

Dataset / setting:
  [Dataset name, split, preprocessing]

Expected outcome:
  If claim holds: [what numbers to expect]
  If claim fails: [what we would see — and what that means for the paper]

Implementation notes:
  [Key parameters, pretrained checkpoints, libraries]
  Estimated compute: ~[N] GPU-hours
```


## Step 3: Verify Each Experiment (Code Inspection)

Before running any experiment, verify the code against this checklist:

### 3a. Claim-to-Code Mapping
- [ ] Identify the exact paper claim this script tests (quote it from the paper)
- [ ] Confirm the output metric corresponds to that claim
- [ ] If claim is about X but metric measures Y → **CRITICAL**

### 3b. Data Integrity
- [ ] Train / validation / test splits are disjoint and never cross-contaminate
- [ ] Test data is never used for hyperparameter selection
- [ ] No data augmentation on the test set that is undisclosed
- [ ] If using a pretrained model: confirm it was not trained on the evaluation set
- [ ] Random seed is fixed

### 3c. Baseline Fairness
- [ ] Baseline uses identical data preprocessing to our method
- [ ] Same compute budget (or difference is disclosed)
- [ ] Baseline hyperparameters are tuned, not left at defaults — or justify why defaults are fair
- [ ] No unintentional advantage for our method (more context, more inference steps, etc.)

### 3d. Metric Implementation
- [ ] Metric matches the paper's definition exactly
- [ ] Library version verified, behavior confirmed on a toy example
- [ ] Edge cases handled: empty inputs, NaN, zero-division
- [ ] Aggregation is correct: mean vs. median, macro vs. micro

### 3e. Evaluation Pitfall Check

For each item: **NOT PRESENT** | **PRESENT — BENIGN** | **PRESENT — PROBLEM**

| Pitfall | Status |
|---|---|
| Evaluation subset sorted by difficulty rather than randomly sampled | |
| Our method's output post-processed in a way baseline's is not | |
| Metric saturates on easy examples, hiding failure on hard ones | |
| "Self-consistency" metric that rewards stable output regardless of correctness | |
| Human evaluation instructions written in a way that favors our output style | |
| Test prompts seen during training (data contamination) | |

### 3f. Code Cleanup (after correctness confirmed)
- [ ] Remove dead code paths
- [ ] Replace magic numbers with named constants + comments
- [ ] Remove redundant intermediate saves
- [ ] Collapse repeated logic into named functions
- [ ] Add top-level docstring: what this measures, expected inputs and outputs


## Step 4: Parallelism

Explicitly list which experiments can run simultaneously so the implementer can schedule GPU jobs efficiently.


## Output Format

```
## Experiment Plan
### Source: [reviewer TURN / paper section / user request]
### Date: [date]

### Execution order:
1. [Name] — [Priority] — [~N GPU-hours]
   Verification: PASSED | BLOCKED (reason)
2. ...

### Can run in parallel: [list]
### Total estimated compute: ~[N] GPU-hours

### Verification summary:
- CRITICAL issues: [N] (must fix before running)
- Warnings: [N]
- All checks passed: [list of experiments ready to run]
```
