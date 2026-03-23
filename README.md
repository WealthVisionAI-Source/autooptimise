# autooptimise

![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)
![OpenClaw Compatible](https://img.shields.io/badge/OpenClaw-compatible-blue)
![Version](https://img.shields.io/badge/version-v0.1.0-green)

> Benchmark-driven skill improvement for OpenClaw. Measure quality objectively, propose targeted changes, validate with live testing — no guesswork.

---

## The Problem

Skills are written once and rarely improved. There's no feedback loop. You don't know if a skill is "good" until it produces a bad output in a real session — and even then, the fix is guesswork.

**autooptimise brings measurement to skill quality:**

- Score your skill against a fixed benchmark (0–10 across 4 dimensions)
- Identify exactly which tasks score low and why
- Propose a targeted, minimal change
- Re-score — keep the change if it improves the mean by ≥0.5, discard if not
- Validate with real API calls to confirm no regressions
- Log everything — what changed, why, and by how much

---

## What It Does

autooptimise runs a closed improvement loop on any OpenClaw AgentSkill:

```
Load skill
    ↓
Run benchmark (10 tasks)
    ↓
Score each output: accuracy · tool_usage · conciseness · formatting
    ↓
Identify lowest-scoring pattern
    ↓
Propose targeted modification to SKILL.md
    ↓
You approve the change (nothing is ever auto-applied)
    ↓
Re-run benchmark
    ↓
Score improved ≥0.5? → KEEP and log it
Score unchanged or worse? → DISCARD and log it
    ↓
Regression test: verify all query types still work
```

---

## Proven Results

Two skills optimised, both validated with live API calls:

| Skill | Baseline | After | Δ | Status |
|-------|----------|-------|---|--------|
| weather | 6.65/10 | 8.04/10 | +1.39 | ✅ Kept |
| github | 8.99/10 | 9.91/10 | +0.92 | ✅ Kept |

**Weather skill:** Added Response Guidelines — agent now returns exactly the field(s) asked for, nothing more. Before: ask for humidity, get humidity + temperature + conditions. After: ask for humidity, get humidity only.

**GitHub skill:** Added `--jq` output formatting guidance. Before: `--jq '.author'` returned raw `{"login": "someone"}`. After: `--jq '"Author: " + .author.login'` returns `Author: someone`. Also added two-step pattern for finding the latest failed CI run.

Both improvements confirmed with live API calls (wttr.in for weather). Zero regressions across all query types.

---

## Requirements

- OpenClaw installed (any version)
- At least one model configured — works with Anthropic, OpenAI, or any NIM model
- No external accounts, no network registration, no API keys beyond what you already have
- No Mission Control or custom agent team required

---

## Installation

```bash
# Via ClawHub CLI
npx clawhub@latest install autooptimise

# Or manual
git clone https://github.com/fg-openclaw/autooptimise \
  ~/.openclaw/workspace/skills/autooptimise
```

---

## Usage

Just ask your OpenClaw agent in natural language:

```
optimise my weather skill
run autooptimise on the github skill
benchmark my news skill and suggest improvements
```

The agent will:
1. Confirm which skill to target
2. Run the benchmark (10 tasks, ~2–5 minutes)
3. Show you the scores and identify the lowest-scoring pattern
4. Propose a specific change with a diff
5. Wait for your approval — **nothing is ever changed without your say-so**
6. Re-run benchmark and show before/after scores
7. Run regression tests to confirm no regressions introduced
8. Log the full run to `runner/experiment_log.md`

---

## Scheduling

**On demand (v0.1):** Ask your agent to run it whenever you want.

**Heartbeat (easy, no cron):** Add a line to your `HEARTBEAT.md`:
```
Check if any skill hasn't been optimised in 30 days. If so, run autooptimise on the oldest one.
```
Your agent will pick this up automatically during its periodic checks.

**Cron (v0.2):** Scheduled overnight runs coming in the next version.

---

## Scoring Rubric

```
aggregate_score = (accuracy × 0.4) + (tool_usage × 0.25) + (conciseness × 0.2) + (formatting × 0.15)
```

| Dimension | Weight | What it measures |
|-----------|--------|-----------------|
| accuracy | 40% | Correct and complete for what was asked |
| tool_usage | 25% | Right command, flags, and parameters |
| conciseness | 20% | No unrequested output, no padding |
| formatting | 15% | Clear, labelled, easy to read |

Score bands: `0–2` poor · `3–5` average · `6–8` good · `9–10` excellent

**Keep threshold:** Mean score must improve by ≥0.5 to keep the change.

---

## How Validation Works

The benchmark alone tells you scores went up. But did the skill actually get better in real usage? That's what validation confirms.

**Benchmark improvement** — did scores go up?
**Regression test** — did we break anything? (All query types re-run against the live API)
**Integration check** — does the skill work end-to-end in a real OpenClaw session?

> *The improvement that matters is observable agent behaviour — not just a higher score. Run the modified skill in a real session. It either works better or it doesn't.*

**v0.1 note:** Benchmark outputs are generated by simulating how an agent following the skill would respond. v0.2 will invoke the skill live and capture real outputs automatically.

---

## Safety

**No external dependencies.** autooptimise makes no network calls beyond your existing OpenClaw model provider. No registration, no telemetry, no external APIs.

**No auto-apply.** Every proposed change requires explicit approval. The agent shows you the exact diff and waits. You can reject, modify, or ask for an alternative.

**Tool availability check (v0.2):** Before running benchmarks, autooptimise will verify that the skill's required tools are installed (e.g. `gh` CLI for the GitHub skill). Running benchmarks for a tool that isn't installed produces meaningless scores — this pre-flight check prevents that.

**Skill safety auditing (v0.2):** A planned audit mode will scan skills for suspicious patterns before optimising them — unexpected external URLs, instructions that attempt to override system prompts, or tool requests inconsistent with the skill's stated purpose. This is particularly important when optimising skills downloaded from unknown sources.

---

## What Makes This Different

There are other self-improvement skills on ClawHub. Here's how autooptimise compares:

| Feature | Others | autooptimise |
|---------|--------|--------------|
| Benchmark scoring (0–10) | ❌ | ✅ |
| Skill-specific optimisation | ❌ | ✅ |
| Agent separation (no self-marking) | ❌ | ✅ |
| No external network required | ❌ some | ✅ |
| Measurable before/after | ❌ | ✅ |
| Regression testing | ❌ | ✅ |
| Human approval gate | ❌ some | ✅ always |

Other approaches either log errors reactively (no measurement), audit config/cost (not skill quality), or run autonomous code changes with no scoring and no approval gate. autooptimise is the only one that asks: *how good is this skill, measured objectively, before and after a change?*

---

## Honest Limitations (v0.1)

1. **Benchmark outputs are simulated.** The agent imagines what a skill would produce — it doesn't invoke the actual tool. v0.2 will use live invocation.
2. **LLM-as-judge has known biases.** Jon scores against a strict rubric, but LLMs can favour confident-sounding outputs. Live validation is the reality check.
3. **10 tasks per skill.** Enough to find patterns, not enough for statistical significance. v0.2 expands to 20–50 tasks.
4. **Tool availability not enforced yet.** If the skill's required tool isn't installed (e.g. `gh` CLI), benchmark scores are meaningless. Pre-flight checks coming in v0.2.

Being upfront about this is what makes the tool trustworthy. The improvements are real — the methodology will tighten.

---

## Roadmap

### v0.1.0 (current)
- [x] Single skill optimisation per run
- [x] 10-task benchmark suite
- [x] LLM judge scoring (4 dimensions)
- [x] Human approval gate — nothing auto-applied
- [x] Regression testing with live API calls
- [x] Experiment log
- [x] Works with any model, no Mission Control required

### v0.2.0 (planned)
- [ ] Live skill invocation (real outputs, not simulated)
- [ ] Tool availability pre-flight check
- [ ] Skill safety auditing (scan for suspicious patterns)
- [ ] Scheduled overnight runs via cron
- [ ] Expanded benchmark suites (20–50 tasks per skill)
- [ ] Multi-skill batch runs
- [ ] Score history and trend charts

---

## File Structure

```
autooptimise/
├── SKILL.md                  ← OpenClaw entry point
├── README.md                 ← This file
├── LICENSE                   ← MIT
├── CHANGELOG.md              ← Version history
├── .gitignore                ← Excludes personal run logs
├── benchmark/
│   ├── tasks.json            ← Benchmark task suite
│   └── scorer.md             ← LLM judge scoring rubric
└── runner/
    ├── run_experiment.md     ← Experiment loop instructions
    └── experiment_log.md     ← Your run history (gitignored)
```

---

## Live Benchmark Results

The following results were produced by running autooptimise against the **real built-in OpenClaw GitHub skill** — all 10 tasks used live `gh` CLI calls against an actual GitHub repo (`WealthVisionAI-Source/autooptimise`). Zero simulation.

### GitHub Skill — 2026-03-23

| | Score |
|-|-------|
| **Baseline** | 7.83 / 10 |
| **After 1 iteration** | 9.49 / 10 |
| **Improvement** | **+1.66** ✅ |

**What was found:** 3 of 10 tasks (list issues, list releases, list PRs) returned completely blank output when `gh` found nothing — users couldn't tell if the command succeeded or failed. Jon (Nemotron/Kimi judge) scored these 4.55 each.

**Single change applied:**
```diff
+- When gh commands return empty output (no issues, PRs, releases, etc.),
+  ALWAYS output a clear confirmation message like "No open issues found"
+  or "No releases available". Never return blank/empty responses.
```

**Per-task before/after:**

| Task | Before | After | Δ |
|------|--------|-------|---|
| Auth check | 8.65 | 8.85 | +0.20 |
| Repo metadata | 9.85 | 9.85 | — |
| **List issues** | **4.55** | **9.70** | **+5.15** |
| Last 5 commits | 9.25 | 9.25 | — |
| **List releases** | **4.55** | **9.70** | **+5.15** |
| File contents | 9.65 | 9.40 | -0.25 |
| Contributors | 8.85 | 8.85 | — |
| Topics/tags | 9.85 | 9.85 | — |
| **Open PRs** | **4.55** | **9.70** | **+5.15** |
| Clone traffic | 8.50 | 9.75 | +1.25 |

One line added to the skill. One iteration. +1.66 improvement — proven with live API calls.

---

## Contributing

Contributions welcome — especially new benchmark task suites for specific skills.

1. Fork the repo
2. Add benchmark tasks for your skill in `benchmark/`
3. Run the loop, share your results in the PR
4. Open a pull request

---

## Support the Project

If autooptimise improved your skills, a small donation keeps development going.

☕ [Buy me a coffee on Ko-fi](https://ko-fi.com/fgili)
💛 [GitHub Sponsors](https://github.com/sponsors/WealthVisionAI-Source)

No pressure — the skill is MIT licensed and always will be.

---

## License

MIT © 2026 — free to use, modify, and redistribute.
