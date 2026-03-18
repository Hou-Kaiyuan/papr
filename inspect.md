
# Paper Quality Inspection

Systematic format and consistency audit. Does NOT evaluate scientific merit --
that is `storyline` and `discussion-panel`.

Severity: **BLOCKER** (fix before submission) | **MAJOR** | **MINOR**


## 1. Figure Audit

For each figure:
- [ ] All sub-figures within a panel are the same size
- [ ] Legend does not overlap data curves or bars
- [ ] Font sizes readable at column width (~9-10pt after embedding)
- [ ] Axis labels present and legible
- [ ] Color palette distinguishable in grayscale (B&W safety)
- [ ] Figure is cited in the body text with `\ref{fig:}`
- [ ] No figure overflows column or page margins


## 2. Table Audit

For each table:
- [ ] Mixed-direction metrics have up/down arrows in column headers
  - Omit arrows if all metrics go the same direction
  - Omit arrows if there is only one metric
- [ ] Table spans appropriate width -- major result tables should use `\begin{table*}` in two-column papers, not be crammed into one column
- [ ] Best result per column is bolded
- [ ] Delta column present when comparing method vs. baseline
- [ ] Table is cited in the body text
- [ ] Table does not duplicate a nearby figure -- flag one for removal
- [ ] No table overflows column or page margins


## 3. Structure Audit

- [ ] Paper fits within page limit (check venue requirements)
- [ ] Abstract is within word limit
- [ ] All sections present: Abstract, Introduction, Related Work, Method, Experiments, Conclusion
- [ ] Appendix/supplementary is properly separated
- [ ] No orphaned headings (heading at bottom of page, content on next)


## 4. Citation Audit

- [ ] Every `\ref{fig:}` and `\ref{tab:}` points to an existing float
- [ ] Every figure and table has at least one `\ref` in the body text
- [ ] No unresolved `?` placeholder references (compilation failures)
- [ ] Citation style is consistent -- no mixing of inline "Author et al." prose and `\cite{}`
- [ ] Inline author name over-use: flag any section with more than 2-3 "X et al." mentions


## 5. Notation Audit

- [ ] All symbols defined at first use
- [ ] No symbol used with two different meanings
- [ ] Acronyms defined at first use in main text (even if defined in abstract)
- [ ] Consistent notation throughout (same symbol for same concept)


## 6. Caption Quality Audit

For each figure/table:
- [ ] Caption is shorter than the body paragraph that references it
- [ ] Caption describes *what is shown*, not *what it means* -- interpretation belongs in the body
- [ ] Caption does not reproduce the body text verbatim


## Output Format

```
## Paper Quality Inspection Report
### Paper: [filename]
### Date: [date]

### Summary: [N] total -- [B] blockers, [M] major, [m] minor

### BLOCKER
- [Section/Figure/Table]: [description of issue]

### MAJOR
- [Section/Figure/Table]: [description of issue]

### MINOR
- [Section/Figure/Table]: [description of issue]

### Passed checks
- [list of audit categories with no issues]
```
