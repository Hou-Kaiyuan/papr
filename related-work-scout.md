
# Related Work Scout

Autonomously searches for related papers, current SOTA methods, and missing citations.
Uses web search throughout. Produces a structured report with routed action items.


## Step 2: Search Protocol

### A. Current State of the Art
```
[task name] state of the art 2024 2025
[task name] benchmark leaderboard
best [metric] [dataset] recent
```
Goal: Find top-performing current methods. Flag any not compared against in the paper.

### B. Missing Baselines
```
[method category] alternatives comparison [year]
[paper's core claim] prior approach
```
Goal: Methods that should appear as baselines but don't.

### C. Citation Verification

For each key citation in the paper:
- Confirm the paper exists and the attributed claim matches what it actually says
- Check if a more recent published version exists (many papers update on arXiv)

### D. Novelty Check
```
[core contribution] similar prior work
[technical approach] [year range]
```
Goal: Find papers that could undermine novelty claims. Better to find them now than in review.

### A. Strong baselines we are missing
| Method | Venue / Year | Performance on [benchmark] | URL |
|---|---|---|---|

### B. Missing citations
- [Title] — relevant because [one-line reason] — [URL]

### C. Citation issues
- [Cited paper]: we attribute "[X]" but paper actually says "[Y]"

### D. Novelty concerns
- [Paper]: similar because [description]. Our difference from it: [TBD]

### E. Suggested additions to related work
[2–3 bullets ready to integrate as text]
```

