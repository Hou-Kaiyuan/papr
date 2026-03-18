
# Related Work Scout

Autonomously searches for related papers, current SOTA methods, and missing citations.
Uses web search throughout. Produces a structured report with routed action items.


## Step 1: Extract Paper Context

Before searching, extract from the paper:
- Core task and problem domain
- Method family (e.g. contrastive learning, diffusion, transformer-based)
- Evaluation benchmarks and datasets used
- Claimed contributions (verbatim from intro)
- Key baselines already compared against
- Year range of cited work (to identify gaps in recent coverage)


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


## Step 3: Output Format

```
## Scout Report
### Paper: [filename]
### Date: [date]

### A. Strong baselines we are missing
| Method | Venue / Year | Performance on [benchmark] | URL |
|---|---|---|---|
| [method] | [venue year] | [metric value] | [url] |

### B. Missing citations
- [Title] -- relevant because [one-line reason] -- [URL]

### C. Citation issues
- [Cited paper]: we attribute "[X]" but paper actually says "[Y]"
- [Cited paper]: newer version available ([old year] -> [new year, URL])

### D. Novelty concerns
- [Paper]: similar because [description]. Our difference: [specific distinction]

### E. Suggested additions to related work
- [2-3 sentences ready to integrate as text, with citations]

### Routing
- Baselines to add as experiments: [list] -> pass to experiments
- Text changes (citations, related work): [list] -> pass to write
- Novelty threats to discuss in panel: [list] -> pass to discussion-panel
```
