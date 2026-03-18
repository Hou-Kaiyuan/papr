---
name: discussion-panel
description: |
  Fully autonomous multi-agent paper review panel. All roles run in parallel
  via the Agent tool, communicating through DISCUSSION_THREAD.md. Two waves:
  first all agents give independent perspectives, then all respond to each other.
  Invoke as /papr panel, /papr panel score, or /papr panel focus <topic>.
allowed-tools:
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - Agent
  - WebSearch
  - WebFetch
---

# Discussion Panel

Fully autonomous multi-agent paper review. All roles run as separate Agent
subagents in parallel — like reviewers sitting in the same room discussing.

**Read `agent-protocol.md` first.**

---

## Subcommands

| Invocation | Behavior |
|---|---|
| `/papr panel` | Full session — two waves of parallel discussion |
| `/papr panel score` | Skip discussion; each agent gives a score + top-3 concerns only |
| `/papr panel focus <topic>` | Full session with all agents focused on a specific topic |

---

## Agents

| Role | File | Reads | Persona |
|---|---|---|---|
| ADVISOR | `roles/advisor.md` | Main + appendix | Senior professor, big-picture |
| REVIEWER_EXPERT | `roles/reviewer-expert.md` | Main + appendix | Domain expert, technically rigorous |
| REVIEWER_STANDARD | `roles/reviewer-standard.md` | Main + appendix | Area chair, thorough review |
| REVIEWER_BRIEF | `roles/reviewer-brief.md` | **Main text only** | Busy reviewer, skips supplementary |
| REVIEWER_LAY | `roles/reviewer-lay.md` | Entire paper | Non-expert from neighboring field |
| AUTHOR | `roles/author.md` | Full thread + paper | PhD student who wrote the paper |

---

## Orchestration

All agents run as **separate Agent subagents** using the Agent tool. This gives
each agent isolated context — they only see what their role should see.

### Setup

1. Read `agent-protocol.md`
2. Locate paper file (PDF or LaTeX source)
3. Create or open `DISCUSSION_THREAD.md` and `ROUND_STATE.md`
4. If Round 2+, prepend previous round summary to `DISCUSSION_THREAD.md`

### Wave 1: Independent Perspectives (all in parallel)

Spawn ALL 6 agents simultaneously using the Agent tool with `run_in_background: true`.

Each agent's prompt must include:
- Their role instructions (from `roles/<role>.md`)
- The paper content (scoped to what their role reads)
- Path to `DISCUSSION_THREAD.md` (append their output)
- Instruction: "Write your initial review/perspective. Append to DISCUSSION_THREAD.md using the agent-protocol format."

```
Spawn in parallel:
  Agent(ADVISOR,          run_in_background: true)
  Agent(REVIEWER_EXPERT,  run_in_background: true)
  Agent(REVIEWER_STANDARD,run_in_background: true)
  Agent(REVIEWER_BRIEF,   run_in_background: true)
  Agent(REVIEWER_LAY,     run_in_background: true)
  Agent(AUTHOR,           run_in_background: true)
```

Wait for ALL agents to complete before proceeding to Wave 2.

In Wave 1, the AUTHOR writes a preemptive defense: acknowledges known weaknesses,
flags areas where reviewers are likely to push back, and outlines what can be fixed.

### Wave 2: Cross-Discussion (all in parallel)

Spawn ALL 6 agents again simultaneously with `run_in_background: true`.

Each agent now reads the full `DISCUSSION_THREAD.md` (containing all Wave 1 output)
and responds to specific points raised by others.

```
Spawn in parallel:
  Agent(ADVISOR,          run_in_background: true)  -> synthesize, assign scores
  Agent(REVIEWER_EXPERT,  run_in_background: true)  -> challenge author defenses
  Agent(REVIEWER_STANDARD,run_in_background: true)  -> respond to cross-cutting issues
  Agent(REVIEWER_BRIEF,   run_in_background: true)  -> flag main-text gaps others missed
  Agent(REVIEWER_LAY,     run_in_background: true)  -> respond to clarity issues
  Agent(AUTHOR,           run_in_background: true)  -> address all concerns, post action list
```

Wait for ALL agents to complete.

### Agent Prompt Template

When spawning each agent, construct the prompt like this:

```
You are [ROLE] in an academic paper review panel.

## Your Role Instructions
[paste content from roles/<role>.md]

## Paper
[paste paper content, scoped to what this role reads]

## Discussion Thread So Far
[paste current DISCUSSION_THREAD.md content, or "Empty — you are writing first" for Wave 1]

## Your Task (Wave [1|2])
Wave 1: Write your initial review from your role's perspective.
Wave 2: Read all other agents' comments and respond. Address specific points by name.

## Output
Append your response to DISCUSSION_THREAD.md at path [absolute path].
Use this format:

---
FROM: [ROLE]
ROUND: [N]
WAVE: [1|2]
---

[your review/response]

SIGNAL: DONE
```

---

## AUTHOR Action List Format

AUTHOR posts this at the end of Wave 2:

```
## Author Action List

### Will fix — text only
- [ ] [specific change] — from [ROLE]

### Will fix — requires new experiment
- [ ] [description] -> pass to experiments

### Will clarify in text (not a real weakness)
- [ ] [description + why]

### Will not fix
- [ ] [description] — reason: [one sentence]
```

---

## Session End

After Wave 2 completes:
1. Read all agents' Wave 2 output from `DISCUSSION_THREAD.md`
2. Extract scores from each reviewer and ADVISOR
3. Compute average -> write to `ROUND_STATE.md`
4. Write session summary:

```
## Panel Summary — Round [N]
### Score: [avg]/10  (Advisor: X | Expert: Y | Standard: Z | Brief: B | Lay: W)

### Key issues raised
- [bullet]

### Author action list
[paste final action list]

### Passed to next phase
- Experiments needed: [N items]
- Text-only changes: [N items]
```

---

## Escalation

| Condition | Action |
|---|---|
| `SIGNAL: ESCALATE` | Surface to human in round summary before continuing |
| Experiment items in action list | Pass to `experiments` |
| Text-only items | Pass to `write` |
