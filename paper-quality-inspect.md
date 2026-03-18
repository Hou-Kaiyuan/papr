
# Paper Quality Inspection

Systematic format and consistency audit. Does NOT evaluate scientific merit —
that is `paper-storyline` and `discussion-panel`.

Severity: **BLOCKER** (fix before submission) | **MAJOR** | **MINOR**


## 2. Table Audit

For each table:
- [ ] Mixed-direction metrics have ↑/↓ arrows in column headers
  - Omit arrows if all metrics go the same direction
  - Omit arrows if there is only one metric
- [ ] Table spans appropriate width — major result tables should use `\begin{table*}` in two-column papers, not be crammed into one column
- [ ] Best result per column is bolded
- [ ] Δ column present when comparing method vs. baseline
- [ ] Table is cited in the body text
- [ ] Table does not duplicate a nearby figure → flag one for removal
- [ ] No table overflows column or page margins


## 4. Citation Audit

- [ ] Every `\ref{fig:}` and `\ref{tab:}` points to an existing float
- [ ] Every figure and table has at least one `\ref` in the body text
- [ ] No unresolved `?` placeholder references (compilation failures)
- [ ] Citation style is consistent — no mixing of inline "Author et al." prose and `\cite{}`
- [ ] Inline author name over-use: flag any section with more than 2–3 "X et al." mentions


## 6. Caption Quality Audit

For each figure/table:
- [ ] Caption is shorter than the body paragraph that references it
- [ ] Caption describes *what is shown*, not *what it means* — interpretation belongs in the body
- [ ] Caption does not reproduce the body text verbatim


## Output Format

```
## Paper Quality Inspection Report
### Paper: [filename]
### Date: [date]

### Summary: [N] total — [B] blockers, [M] major, [m] minor
```
