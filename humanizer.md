
# Humanizer

Removes AI writing patterns and produces natural, human academic prose.

**Run after all content changes are complete. Never mid-edit.**


## What to Remove

### 1. Em Dashes (—) — Highest Priority

The single clearest AI writing signal. Every em dash must be replaced.

| Pattern | Replace with |
|---|---|
| `X — Y` (contrast) | `X; Y` or `X, whereas Y` |
| `X — Y — Z` (parenthetical) | `X (Y) Z` or split into two sentences |
| `X — Y` (elaboration) | `X: Y` or `X, which Y` |
| `X — that is, Y` | `X, specifically Y` or `X (i.e., Y)` |

After processing each section, run: `grep "—"` → must return zero results.

### 2. Rhetorical Questions

Convert all rhetorical questions in body text to declarative statements.

| Before | After |
|---|---|
| "Why does this work? Because..." | "This works because..." |
| "What is the key insight? The model..." | "The key insight is that the model..." |
| "How do we address this? We propose..." | "We address this by proposing..." |

### 3. Hedging Stacks

One hedge word per claim maximum.

| Before | After |
|---|---|
| "we believe it may possibly suggest" | "this suggests" |
| "it seems that perhaps" | "this indicates" |
| "could potentially be useful for" | "is useful for" |

### 4. Overused AI Transition Phrases

| Replace | With |
|---|---|
| "It is worth noting that" | State the fact directly |
| "Furthermore," (used >3× in a section) | Vary: "In addition," / "Also," / restructure |
| "This allows us to" | Describe what it does directly |
| "Importantly," (used >1× in the paper) | Use at most once; cut the rest |
| "In conclusion," (in body text) | Reserve for the actual Conclusion section only |
| "As mentioned earlier," | Restructure or add a specific `\ref` |

### 5. Passive Voice Chains

Flag chains of 3+ consecutive passive constructions and rewrite at least one active.

"It can be seen that X is shown to be..." → "X is..." or "We show that X..."


## Self-Test Before Finishing Each Section

- [ ] `grep "—"` → zero results
- [ ] `grep "?"` in body → zero or near-zero
- [ ] No "a reviewer might ask" anywhere
- [ ] First paragraph reads naturally when read aloud

