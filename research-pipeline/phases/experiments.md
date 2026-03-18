# Phase 4: Experiments (conditional)

Only runs if the AUTHOR action list contains items marked "requires new experiment".

## Steps

1. Check the AUTHOR action list from `DISCUSSION_THREAD.md`
2. If no experiment items → skip this phase, proceed to Phase 5
3. If experiment items exist:
   a. Pass items to `experiments` → produces prioritized experiment plan
   b. Pass plan to `code-inspect` → verifies correctness before any code runs
   c. Fix any CRITICAL issues flagged by code-inspect
   d. Run approved experiments
   e. Collect results

## Handoff to Write Phase

Experiments results are passed to `write` in Phase 5 alongside the text-only changes.
Include: metric values, table data, figure data, any new ablation results.

## Skip Condition

If code-inspect finds a CRITICAL issue that cannot be resolved quickly,
flag it in `ROUND_STATE.md` under "Open issues" and skip the experiment.
The write will note in the text that this experiment is planned for future work.
