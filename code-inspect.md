
# Code Inspector

Verifies that experiment code faithfully evaluates the paper's claimed contribution,
catches evaluation shortcuts that could misrepresent results, then simplifies.

**Block execution if any CRITICAL issue is found. Fix first, then re-inspect.**


## Checklist

### 1. Claim-to-Code Mapping
- [ ] Identify the exact paper claim this script tests (quote it from the paper)
- [ ] Confirm the output metric corresponds to that claim
- [ ] If claim is about X but metric measures Y → **CRITICAL**

### 2. Data Integrity
- [ ] Train / validation / test splits are disjoint and never cross-contaminate
- [ ] Test data is never used for hyperparameter selection
- [ ] No data augmentation on the test set that is undisclosed
- [ ] If using a pretrained model: confirm it was not trained on the evaluation set
- [ ] Random seed is fixed

### 3. Baseline Fairness
- [ ] Baseline uses identical data preprocessing to our method
- [ ] Same compute budget (or difference is disclosed)
- [ ] Baseline hyperparameters are tuned, not left at defaults — or justify why defaults are fair
- [ ] No unintentional advantage for our method (more context, more inference steps, etc.)

### 4. Metric Implementation
- [ ] Metric matches the paper's definition exactly
- [ ] Library version verified, behavior confirmed on a toy example
- [ ] Edge cases handled: empty inputs, NaN, zero-division
- [ ] Aggregation is correct: mean vs. median, macro vs. micro

### 5. Potential Fallback Check

For each item: **NOT PRESENT** | **PRESENT — BENIGN** | **PRESENT — PROBLEM**

| Fallback | Status |
|---|---|
| Evaluation subset sorted by difficulty rather than randomly sampled | |
| Our method's output post-processed in a way baseline's is not | |
| Metric saturates on easy examples, hiding failure on hard ones | |
| "Self-consistency" metric that rewards stable output regardless of correctness | |
| Human evaluation instructions written in a way that favors our output style | |
| Test prompts seen during training (data contamination) | |

### 6. Simplification (after correctness confirmed)
- [ ] Remove dead code paths
- [ ] Replace magic numbers with named constants + comments
- [ ] Remove redundant intermediate saves
- [ ] Collapse repeated logic into named functions
- [ ] Add top-level docstring: what this measures, expected inputs and outputs


## Output Format

```
## Code Inspection Report
### Script: [filename]
### Paper claim tested: "[exact quote]"
### Date: [date]

