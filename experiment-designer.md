
# Experiment Designer

Takes reviewer feedback, weak claims, or action list items marked "requires new experiment"
and produces a concrete, prioritized plan ready to hand to an implementer.

**After producing the plan, pass it to `code-inspector` before running anything.**


## Step 1: Check Before Designing

Before designing a new experiment, ask:
- Does an existing result already answer this? (May just need better presentation)
- Is this addressed in the appendix but not the main text? (Move it up, don't re-run)
- Can existing data be re-analyzed differently to answer this? (Subset analysis, different metric)

Only proceed to design if none of the above apply.


## Step 3: Design Each Experiment

```
### Experiment: [Name]
Motivation: [Which claim or TURN number this addresses]
Category: [from table above]
Priority: BLOCKER | HIGH | MEDIUM | LOW

What to measure:
  Primary metric: [name, direction ↑/↓]
  Why this metric: [one sentence]
  Secondary metrics: [if any]

Conditions:
  Ours: [description]
  Baseline A: [verify current SOTA via web search]
  Baseline B: [description]

Dataset / setting:
  [Dataset name, split, preprocessing]

Expected outcome:
  If claim holds: [what numbers to expect]
  If claim fails: [what we would see — and what that means for the paper]

Implementation notes:
  [Key parameters, pretrained checkpoints, libraries]
  Estimated compute: ~[N] GPU-hours

Potential evaluation pitfalls to check (for code-inspector):
  [Any ways this evaluation could be gamed or misleading]
```


## Step 5: Parallelism

Explicitly list which experiments can run simultaneously so the implementer can schedule GPU jobs efficiently.


## Output Format

```
## Experiment Plan
### Source: [reviewer TURN / paper section / user request]
### Date: [date]

### Execution order:
1. [Name] — [Priority] — [~N GPU-hours]
2. ...

### Can run in parallel: [list]
### Total estimated compute: ~[N] GPU-hours

### Next step: pass to code-inspector before implementation
```
