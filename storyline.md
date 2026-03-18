
# Paper Storyline and Logic Check

Evaluates whether the paper's argument is coherent, motivated, and correctly sequenced
from the perspective of a first-time reader. Does NOT check formatting — that is `inspect`.


## Section-by-Section Checklist

### Abstract
- [ ] Problem stated in first 1–2 sentences, no jargon
- [ ] Why it matters is clear (motivation, not just description)
- [ ] The approach is distinguishable from the problem statement
- [ ] Key result is quantified — not "significantly better" but "X% on Y benchmark"
- [ ] No method limitations mentioned (those belong in Limitations)

### Introduction
- [ ] Para 1: real-world motivation — why should anyone outside this subfield care?
- [ ] Para 2: what specifically is technically hard (the gap)?
- [ ] Para 3: what prior work tried, and concretely why it falls short
- [ ] Para 4: the proposed approach at high level — no equations yet
- [ ] Contributions list: ≤3 items, each concrete and verifiable, no proposition/section numbers
- [ ] Transition to Related Work is smooth

**Common failure:** Introduction jumps from real-world motivation directly to a specific solution without establishing why prior approaches fail. Flag if the gap paragraph is missing.

**Specific check:** If the method relies on a specific tool, space, or model (e.g. CLIP, contrastive embeddings, barycentric interpolation), is the choice explicitly motivated — does "we use X because Y" appear before the method section?

### Related Work
- [ ] Organized into 2–4 thematic subsections, not a flat citation list
- [ ] Each cluster is differentiated from this paper — not just "X also does Y"
- [ ] Author names not over-used inline; `\cite{}` preferred
- [ ] Length appropriate: ~1 column for short papers
- [ ] Use web search: are there obvious missing related works from the last 1–2 years?

### Method
- [ ] Every concept used is introduced before it is used — no forward dependencies
- [ ] The method follows naturally from the gap established in the introduction
- [ ] Key assumptions are either cited from prior work or supported by experiment
- [ ] Mathematical notation is consistent throughout
- [ ] Every "X works because Y" is backed by theory or experiment, not just asserted
- [ ] Specific technical terms introduced here (symbols, coined names) are defined at first use

**Abruptness flag:** List every concept that appears in the method without prior introduction. Each one is a transition failure.

### Evaluation
- [ ] Each experiment maps to a specific claim from introduction or method
- [ ] Baselines are justified — web search to verify they are current SOTA
- [ ] Metrics are appropriate and justified for the claimed task
- [ ] Every "outperforms" claim is directly supported by the shown numbers
- [ ] Results section interprets numbers — does not just restate table values
- [ ] Ablations cover all key design decisions

### Discussion / Limitations
- [ ] Limitations are specific and quantified, not vague "future work" language
- [ ] Discussion connects back to the introduction's claims


## Output Format

```
## Storyline and Logic Check
### Paper: [filename]
### Date: [date]

