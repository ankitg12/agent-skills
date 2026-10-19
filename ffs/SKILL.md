---
name: ffs
description: "Mine recurring user corrections and workflow friction from agent session transcripts, then convert mechanically detectable patterns into tested rules — use for \"mine rules\", \"what rules should we create\", \"turn this friction into a rule\", repeated corrections, or \"/ffs\"."
---

# FFS — Friction → Fix → Safeguard

Use session friction to improve the harness. The second time the user must repeat a
correction is evidence that prose failed and a mechanical safeguard may be warranted.

## Architecture

Do not build another rule generator. Use the existing layers:

1. **Extract** — a deterministic miner over the agent's session transcripts finds repeated
   user corrections and pairs each with the assistant action before it and the corrected
   action after it. (For AgentsView-backed setups, an extractor like `avmine.py rules`
   does this; any JSONL transcript store can be mined the same way.)
2. **Classify** — decide whether the failure is mechanically detectable.
3. **Generate and validate** — pass one evidence bundle to the harness's existing
   rule-generation path (e.g. an `/omfg`-style command), rather than reimplementing it.
4. **Enforce** — save the accepted output as a project rule (`.omp/rules/<name>.md`) or a
   global rule (`~/.omp/agent/rules/<name>.md`).
5. **Verify** — test positive and negative examples against the rule engine; later re-mine
   the transcripts to confirm the correction stopped recurring.

## When a friction becomes a rule

Create a rule only when all are true:

- The user corrected the same behaviour at least twice, or once with an explicit safety
  consequence.
- The offending action is mechanically recognizable in assistant text or tool arguments.
- A safe replacement can be stated without guessing context.
- No existing rule already covers the pattern.

Otherwise route it elsewhere:

| Failure | Home |
|---|---|
| Detectable command, path, flag, or output phrase | Rule |
| Multi-step workflow requiring judgement | Skill |
| Stable preference or principle | AGENTS.md |
| Tool fact or procedure | Notes / project knowledge |
| One-off historical correction | Journal / source record |

## Mining procedure

Run the deterministic extractor first. If it is unavailable, search raw transcripts for
correction phrases and inspect the surrounding assistant turns; do not infer the offending
action from the correction alone.

For each candidate retain:

- exact user correction and timestamp
- project and stable session id
- assistant command/text immediately before it
- corrected command/text immediately after it
- recurrence count across distinct logical sessions
- recommended scope: project or global
- positive test (must trigger)
- negative test (must not trigger)

Do not use auto-generated session summaries as evidence. They paraphrase and duplicate.
Use raw transcript messages and tool calls.

## Rule-generation handoff

Hand the complaint plus the concrete before/after pair to the existing rule generator,
which already:

- generates a rule candidate
- validates its condition against assistant history
- offers project vs global scope
- permits amendment before saving

Do not duplicate that machinery in the extractor or this skill.

## Verification

For every accepted rule, run both a negative and a positive case through the rule engine
(with OMP's TTSR, for example):

```text
omp ttsr test --rule <absolute-rule-path> --source tool --tool <tool> '<bad input>' -v
omp ttsr test --rule <absolute-rule-path> --source tool --tool <tool> '<good input>' -v
```

The first must trigger; the second must not. A rule that blocks the safe path is worse than
no rule.

After one week, mine the same correction again. Zero recurrences is evidence that the rule
worked. A recurrence means the condition missed the actual form; amend from evidence, not
by broadening blindly.

## Reference example: "puttys pls"

Transcripts showed the same correction on two consecutive days. Before it, sessions ran
opaque one-liner SSH/exec commands against remote testbed hosts with no visible terminal.
After it, they launched visible terminal sessions through named `putty-*` wrapper scripts.

The resulting project rule (`<project>/.omp/rules/visible-putty-operations.md`) catches
the opaque command paths, allows the visible wrappers, and is backed by the historical
correction events — three rounds of evidence, not a guess.
