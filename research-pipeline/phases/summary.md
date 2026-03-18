# Phase 8: Round Summary

Writes the round summary to `ROUND_STATE.md` and presents it to the human.

## Steps

1. Collect from this round:
   - Internal panel average score (from `DISCUSSION_THREAD.md`)
   - External reviewer score (from Phase 7, if available)
   - All changes logged by `write`
   - Open issues not yet resolved
   - AUTHOR action items carried forward

2. Write to `ROUND_STATE.md`:

```markdown
## Round [N] Summary

### Scores
| Source | Score |
|---|---|
| Internal panel average | X/10 |
| External reviewer | Y/10 |
| Combined estimate | Z/10 |

### What was discussed
- [bullet per major discussion point from DISCUSSION_THREAD.md]

### What changed this round
**Text changes:**
- [list from write log]

**New experiments:**
- [list]

**Figures / tables updated:**
- [list]

**Content removed:**
- [list]

### Open issues → Round [N+1]
- [ ] [issue — source]

### Top remaining concerns
1. [concern] — [BLOCKER | MAJOR | MINOR]
2. [concern]
3. [concern]
```

3. Present summary to human
4. If rounds remain: ask "Ready for Round [N+1]? Any specific focus?"
5. If final round: "Pipeline complete. Final estimated score: [Z/10]."
