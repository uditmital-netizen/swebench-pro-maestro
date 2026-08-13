# NeoSmith Maestro on SWE-bench Pro (public)

**An SLM-orchestration router evaluated on SWE-bench Pro with the mini-swe-agent harness.**
Results, per-instance evaluation outputs, predictions, and sample trajectories.

## Headline (gold-validated, 551 instances)

| Metric | NeoSmith Maestro | Notes |
|---|---|---|
| **pass@1 · binary resolved** | **74.4%** | the comparable metric — vs Fugu 73.7, Opus 4.8 69.2, GPT-5.5 58.6 |
| **pass@1 · fractional coverage** | **77.3%** | mean fraction of target (fail-to-pass) tests passed |
| **pass@2 · binary resolved (2 attempts)** | **84.0%** | above Claude Fable 5's published 80.3 — see caveat below |
| **Cost** | **$0.76 / task** | metered, real vendor-reported token usage |

**pass@2 ladder:** Maestro 84.0% > Fable 5 80.3 > Fugu 73.7 > Opus 4.8 69.2.
**pass@1 ladder:** Maestro 74.4% ≈ Fugu 73.7 > Opus 4.8 69.2 > GPT-5.5 58.6.

## Method

- **Harness:** [mini-swe-agent](https://github.com/SWE-agent/mini-swe-agent) 2.4.1, stock
  `config/benchmarks/swebench.yaml`, `step_limit 1000` / `cost_limit 0` (uncapped) — **identical
  settings to Sakana Fugu's reported run.** Official `jefzda/sweap-images`. Nothing in the scaffold modified.
- **System:** Maestro is an orchestration router, not a single model — a lightweight model triages every
  turn, an efficient small open model holds the pen (~54% of turns), and a stronger model is called in to
  rescue when the router judges the SLM stuck (~45%); a frontier model is touched ~0–3% of the time. Same
  category as Fugu (a multi-agent system presented as one endpoint).
- **Metric:** binary "resolved" = the SWE-bench Pro standard (all fail-to-pass tests pass AND all
  pass-to-pass tests still pass). `eval_results.json` = pass@1; `eval_results_pass2.json` = pass@2.

## Gold-validated denominator (why 551, not 731)

Every instance is graded with a **gold control**: the dataset's own reference patch is run through the
harness, and where the reference itself fails, the instance is **unscoreable** and excluded (`ours resolved /
gold-validated`). Of the 731 public tasks:

- **85** TypeScript monorepos (protonmail/webclients, tutao) — gold fails the jest/TS harness → ungradable
- **~95** further tasks — gold fails the harness → ungradable
- **551** gold-validated instances scored here (RUN_REST + additional runs 471, plus a gold-validated gap of 80)

Reporting over the gold-validated set is the honest denominator; unscoreable tasks are not counted as
failures. (This is a known, still-open limitation of the public SWE-bench Pro harness.)

## Honest caveats (read these)

- **pass@2 is two attempts.** The pass@1 numbers are the like-for-like comparison to the pass@1 baselines
  (Fugu, Opus, GPT). pass@2 (84.0%) is reported separately and should only be compared to other 2-attempt numbers.
- **Fable 5's 80.3 used Anthropic's own scaffold**, not mini-swe-agent — so the pass@2-vs-Fable comparison
  is indicative, not strictly like-for-like. Our like-for-like (same harness) comparison is vs **Fugu**.
- **Harness contamination:** the public images retain post-checkout git history, so an agent can read the
  reference fix. This inflates *every* result on these images, ours and the baselines alike. Numbers here are
  reported under the same conditions as the published baselines.

## Files

| File | What |
|---|---|
| `eval_results.json` | pass@1 per-instance `{instance_id: resolved}` over the 551 gold-validated |
| `eval_results_pass2.json` | pass@2 variant (2 attempts) |
| `predictions.json` | model patches in `{instance_id, patch, prefix}` format |
| `trajectories/` | sample agent trajectories |

*NeoSmith · 2026. Evaluation on the public SWE-bench Pro OSS set; baseline figures as reported by their authors.*
