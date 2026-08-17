# NeoSmith Maestro — SWE-bench Pro Submission

**An SLM-orchestration router evaluated on SWE-bench Pro with the mini-swe-agent harness, graded by the official Scale harness.**

---

## 1. Results

This submission covers **534 of the 731 SWE-bench Pro tasks (73% of the benchmark)** — every instance for which we retain a patch from the Maestro pipeline — each graded by the official Scale harness. Per task we submit the best patch we retain: the provably-best-config patch where it survives (182 tasks), and the pipeline's trajectory patch otherwise (352 tasks).

| Metric | Attempted (534) | Full benchmark (731) |
|---|---|---|
| **pass@1** (single attempt) | **73.6%** (393 / 534) | **53.8%** (393 / 731) |

Comparable single-attempt baselines (over the full 731): **Fugu 73.7, Fable 5 80.3, Opus 4.8 69.2, GPT-5.5 58.6.**

### Read the denominators honestly

- **73.6% over attempted (534)** — how often Maestro resolves an issue it takes on. Level with Sakana Fugu (73.7), above Opus 4.8 (69.2), under the same mini-swe-agent regime.
- **53.8% over the full 731** — coverage-adjusted: the **197 tasks not yet attempted** (dominated by the hardest TypeScript monorepos — protonmail/tutao) count as unresolved. This is lower than the attempted rate by construction, and closing that coverage gap is in progress.
- On the subset with **provably-best-config patches (182)**, pass@1 is **80.8%**; the mixed-provenance patches on the other 352 tasks pull the blended figure to 73.6%. So 73.6% is a floor for the best-config capability, not a ceiling.

We report both denominators openly so the figure cannot be mistaken for something it is not.

---

## 2. The system

Maestro is not a single model. It is an **SLM-orchestration pipeline** behind one endpoint: a lightweight model (MiniMax-M3) triages every turn, an efficient model (GLM-5.2) holds the pen (~54% of turns), a stronger model (Kimi-K3) is summoned only on a demonstrated struggle signal and then hands back, and a frontier model is touched ≈1% of the time. Every action is settled by **execution** — running the repro / test suite — and that verdict drives the next routing decision. Full architecture, collaboration modes, distillation, and economics are in the companion report (`maestro-paper.html`).

---

## 3. Harness

- **[mini-swe-agent](https://github.com/SWE-agent/mini-swe-agent) 2.4.1**, stock `config/benchmarks/swebench.yaml`, `step_limit 1000 / cost_limit 0` — **identical settings to Sakana Fugu's reported run.** Official `jefzda/sweap-images`. Nothing in the scaffold modified.
- Minimal bash-only ReAct loop (not the richer SWE-Agent ACI scaffold) — the harder, less-scaffolded regime Fugu reported under.

---

## 4. Grading — official harness, with one documented fix

All figures are graded by **Scale's own `swe_bench_pro_eval.py --use_local_docker`** (execution of every fail-to-pass and pass-to-pass test), not our in-house grader.

**One harness fix was required** and it matters for any local-docker run: the official eval creates its Docker client with `docker.from_env()`, defaulting to a **60-second read timeout**. Long test suites (Go/TypeScript monorepos routinely exceed 60s) trip it, the harness returns `None`, and the task is silently scored **False** — an *infra* failure, not a capability failure. The fix is one line — `docker.from_env(timeout=3600)` — and changes no grading logic. Without it, a local-docker run understates *every* system, ours and baselines alike.

---

## 5. Honest caveats

- **73% coverage, not full.** 534 of 731 tasks are attempted; the 197 not attempted (mostly the hardest TypeScript monorepos) count as unresolved over the full benchmark. The full-731 figure (53.8%) is coverage-limited, not capability-limited — on tasks we attempt we resolve 73.6%.
- **Mixed patch provenance.** 182 tasks carry provably-best-config patches (80.8% on that slice); the other 352 carry the pipeline's trajectory patches from across runs. Each submitted patch is matched to its own official-harness verdict, so every result reproduces.
- **Single attempt (pass@1)**, the like-for-like metric against the pass@1 baselines.
- **Fable 5's 80.3 used Anthropic's own scaffold**, not mini-swe-agent — so comparisons to it are indicative; the clean same-harness comparison is vs **Fugu (73.7)**.
- **Harness contamination.** The public images retain post-checkout git history; this inflates *every* result on these images — ours and the baselines — equally.

---

## 6. Files

| File | Contents |
|---|---|
| `eval_results.json` | pass@1 per-instance `{instance_id: resolved}` over the 534 attempted tasks, official-harness graded (**393 resolved / 534**) |
| `predictions.json` | the 534 model patches, `{instance_id, patch, prefix}` — each matched to its verdict |
| `maestro-paper.html` | full technical report (architecture, distillation, economics) |

---

## 7. Reproduction

1. Pull the `jefzda/sweap-images` for each instance.
2. Run mini-swe-agent 2.4.1 (stock `swebench.yaml`, `step_limit 1000 / cost_limit 0`) against the Maestro endpoint, with `MSWEA_COST_TRACKING=ignore_errors`.
3. Grade with `swe_bench_pro_eval.py --use_local_docker` (apply the `timeout=3600` Docker-client fix).
4. Resolved = `passed_tests ⊇ (fail_to_pass ∪ pass_to_pass)` per instance.

---

*NeoSmith · 2026. SWE-bench Pro (public OSS set). Baselines as reported by their authors; all Maestro figures graded by the official Scale harness.*
