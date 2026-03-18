
# Humanizer

Removes AI writing patterns and produces natural, human academic prose.

**Run after all content changes are complete. Never mid-edit.**

## How It Works

This subcommand delegates to the `/humanizer` skill, which covers 24 AI writing
patterns based on Wikipedia's "Signs of AI writing" guide. Invoke it on each
modified section of the paper.

## Workflow

1. Identify all sections modified in the current round (check `ROUND_STATE.md`
   or the author action list).
2. For each modified section, extract the LaTeX text.
3. Run `/humanizer` on the extracted text.
4. Replace the original section text with the humanized output.
5. Run the self-test checklist below before moving on.

## Academic-Specific Additions

On top of the base humanizer patterns, also check for these academic tells:

### Em Dashes (—)

The single clearest AI writing signal in academic text. Every em dash must go.

| Pattern | Replace with |
|---|---|
| `X — Y` (contrast) | `X; Y` or `X, whereas Y` |
| `X — Y — Z` (parenthetical) | `X (Y) Z` or split into two sentences |
| `X — Y` (elaboration) | `X: Y` or `X, which Y` |
| `X — that is, Y` | `X, specifically Y` or `X (i.e., Y)` |

### Rhetorical Questions

Convert all rhetorical questions in body text to declarative statements.

| Before | After |
|---|---|
| "Why does this work? Because..." | "This works because..." |
| "What is the key insight? The model..." | "The key insight is that the model..." |
| "How do we address this? We propose..." | "We address this by proposing..." |

### Hedging Stacks

One hedge word per claim maximum.

| Before | After |
|---|---|
| "we believe it may possibly suggest" | "this suggests" |
| "it seems that perhaps" | "this indicates" |
| "could potentially be useful for" | "is useful for" |

### Overused Academic AI Transitions

| Replace | With |
|---|---|
| "It is worth noting that" | State the fact directly |
| "Furthermore," (used >3x in a section) | Vary: "In addition," / "Also," / restructure |
| "This allows us to" | Describe what it does directly |
| "Importantly," (used >1x in the paper) | Use at most once; cut the rest |
| "In conclusion," (in body text) | Reserve for the actual Conclusion section only |
| "As mentioned earlier," | Restructure or add a specific `\ref` |

### Passive Voice Chains

Flag chains of 3+ consecutive passive constructions and rewrite at least one active.

"It can be seen that X is shown to be..." → "X is..." or "We show that X..."

## Self-Test Before Finishing Each Section

- [ ] `grep "—"` → zero results
- [ ] `grep "?"` in body → zero or near-zero
- [ ] No "a reviewer might ask" anywhere
- [ ] First paragraph reads naturally when read aloud
