---
name: autooptimise
description: Autonomously optimise OpenClaw skill files using a Karpathy-style benchmark-driven experiment loop. The agent runs a skill against a fixed set of test tasks, scores each output using an LLM judge, proposes a targeted modification, re-tests, and keeps or discards the change — iterating up to 3 times per run. Use when asked to "optimise my [skill-name] skill", "run autooptimise on [skill]", "improve my skill overnight", or "benchmark my skill". Designed for overnight autonomous runs with human approval before any changes are applied.
---

# autooptimise

Autonomous benchmark-driven skill optimisation for OpenClaw. Inspired by Andrej Karpathy's [autoresearch](https://github.com/karpathy/autoresearch) — the same modify → test → score → keep/discard loop, applied to agent skill quality instead of GPU training.

## Trigger Phrases

- `"optimise my weather skill"`
- `"run autooptimise on [skill-name]"`
- `"benchmark my [skill-name] skill"`
- `"improve my skill overnight"`

## Key Files

| File | Purpose |
|------|---------|
| `benchmark/tasks.json` | Test task suite (prompts + expected qualities) |
| `benchmark/scorer.md` | LLM judge scoring rubric |
| `runner/run_experiment.md` | Autonomous loop instructions (load this next) |
| `runner/experiment_log.md` | Auto-created run log (gitignored) |

## How to Run

1. Read `runner/run_experiment.md` — it contains the full loop instructions
2. Confirm the target skill with the user if not specified
3. Execute the loop (max 3 iterations)
4. Present proposed changes for human approval — **never auto-apply**

## Scoring

Use the best available LLM judge model (prefer a strong reasoning model). Score each task 0–10 on:
- **Accuracy** — correct answer / correct tool called
- **Conciseness** — no padding, no unnecessary text
- **Tool usage** — right tool, right parameters
- **Formatting** — output matches expected format

Full rubric: `benchmark/scorer.md`

## Safety Rules

- **Never auto-apply changes.** Always present a diff and wait for explicit human approval.
- **Never modify** `benchmark/tasks.json` or `benchmark/scorer.md` during a run.
- **Never exceed 3 iterations** per run in v0.1.
- Log every action to `runner/experiment_log.md`.
