# Phase 6: Humanize

Invokes `humanizer` on all sections modified in Phase 5.

## Steps

1. Read the list of modified sections from `ROUND_STATE.md` (added by write)
2. Run `humanizer` on each modified section
3. Confirm after each section: `grep "—"` returns zero results

## Scope

Only modified sections need humanizing.
Unmodified sections should not be touched — they were already humanized in a previous round
(or will be in a future round when they are modified).

## Output

Append to `ROUND_STATE.md`:
```
### Humanizer pass — [date]
Sections processed: [list]
Em dashes removed: [N]
Rhetorical questions converted: [N]
```
