# Phase 2: Inspect

Runs `paper-quality-inspect` and `paper-storyline` in sequence.

## Steps

1. Run `paper-quality-inspect`: format, figures, tables, citations, terms, structure
2. Run `paper-storyline`: narrative logic, claim-to-experiment map, transition failures

## Output

Append to Round Briefing:

```
## Inspect Report — Round [N]

### Quality issues
BLOCKERS: [N]
- [BLOCKER] [location] — [description]

MAJOR: [N]
- [MAJOR] ...

MINOR: [N]
- [MINOR] ...

### Storyline issues
Verdict: COHERENT | PARTIALLY COHERENT | NEEDS MAJOR REVISION

Claim-to-experiment gaps:
- [claim] → [missing/weak experiment]

Transition failures:
- [section → section]: [description]
```
