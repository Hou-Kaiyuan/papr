# Phase 1: Scout

Uses `related-work-scout` skill.

## Steps

1. Extract from the paper: core task, method family, evaluation benchmarks, claimed contributions
2. Run web searches in three categories:

**A. State of the art**
- `[task] state of the art 2024 2025`
- `[task] benchmark leaderboard`

**B. Missing baselines**
- `[method category] alternatives comparison`
- `[core claim] prior approach`

**C. Novelty check**
- `[contribution] similar prior work`

3. For each key citation in the paper, verify it exists and the attributed claim is accurate

## Output

Append to Round Briefing:

```
## Scout Report — Round [N]

### Missing baselines
| Method | Venue/Year | Performance | URL |

### Missing citations
- [title] — relevant because [reason]

### Novelty concerns
- [paper]: similar because [description]

### Actions
- [finding] → [route to: experiment-designer | paper-writer | discussion-panel]
```
