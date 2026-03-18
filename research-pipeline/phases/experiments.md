# Phase 4: Experiments (conditional)

Only runs if the AUTHOR action list contains items marked "requires new experiment".

## Steps

1. Check the AUTHOR action list from `DISCUSSION_THREAD.md`
2. If no experiment items → skip this phase, proceed to Phase 5
3. If experiment items exist:
   a. Pass items to `experiment-designer` → produces prioritized experiment plan
   b. Pass plan to `code-inspector` → verifies correctness before any code runs
   c. Fix any CRITICAL issues flagged by code-inspector
   d. Run approved experiments
   e. Collect results

## Handoff to Write Phase

Experiments results are passed to `paper-writer` in Phase 5 alongside the text-only changes.
Include: metric values, table data, figure data, any new ablation results.

## Skip Condition

If code-inspector finds a CRITICAL issue that cannot be resolved quickly,
flag it in `ROUND_STATE.md` under "Open issues" and skip the experiment.
The paper-writer will note in the text that this experiment is planned for future work.
