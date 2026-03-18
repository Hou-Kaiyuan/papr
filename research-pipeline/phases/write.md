# Phase 5: Write

Invokes `write` with the AUTHOR action list and any experiment results.

## Steps

1. Collect inputs:
   - AUTHOR action list (text-only items) from `DISCUSSION_THREAD.md`
   - Experiment results from Phase 4 (if any)
   - Quality issues from `inspect` (BLOCKER + MAJOR items)

2. Run `write` — implements all changes in priority order:
   a. Structural changes first (section reorders, merges)
   b. Content changes (rewrites, additions, removals)
   c. Local fixes (terms, citations, captions)
   d. Style fixes (em dashes, author names) — last, before humanizer

3. Every change logged to `ROUND_STATE.md` under `## Changes this round`

## Do Not Run Humanizer Here

Humanizer runs in Phase 6, after all edits are complete.
write should flag sections for humanizer but not run it mid-edit.
