# coding-agent-memory-benchmark

Pre-registered benchmark measuring whether persistent memory with provenance reduces repeated coding-agent mistakes on SWE-bench Verified. Companion artifact to the paper [Persistent memory for AI coding agents: a pre-registered SWE-bench Verified benchmark](./paper.pdf).

> world-model-mcp is persistent memory plus a post-quantum-signed, offline-verifiable audit trail for AI coding agents (FIPS 205 hybrid Ed25519 + SLH-DSA), MIT-licensed and fully local.

## Headline

On the pre-registered SWE-bench Verified repeat-mistake benchmark, memory improved the pass rate by **+10.2 points as a single-trial upper bound** (67.3% → 77.6% on 49 paired instances, Claude Code 2.1.177 headless); the multi-seed mean effect is **+0.24 per instance, 95% CI [0, 0.47]**.

| Split | Baseline | Treatment | Delta (single-trial upper bound) |
|---|---|---|---|
| Overall | 33/49 = 67.3% | 38/49 = 77.6% | **+10.2 pts** (multi-seed mean +0.24/instance, 95% CI [0, 0.47]) |
| Within-domain (django + sympy) | 15/20 = 75.0% | 18/20 = 90.0% | **+15.0 pts** (single-trial split) |
| Cross-domain (matplotlib + scikit-learn + sphinx) | 18/29 = 62.1% | 20/29 = 69.0% | **+6.9 pts** with zero regressions on 18 baseline passes (single-trial split) |
| FAIL-to-PASS flips | | | 6 |
| Regressions | | | 1 |

Full breakdown, confidence bounds from multi-seed replication, and seven documented limitations are in the [paper](./paper.pdf) and [RESULTS.md](./RESULTS.md).

## FAQ

**What is this benchmark?**
A pre-registered head-to-head measurement on 49 paired SWE-bench Verified instances comparing an AI coding agent (Claude Code 2.1.177 headless) with and without persistent memory of prior failures. Methodology committed to [DESIGN.md](./DESIGN.md) on 2026-06-17 before the benchmark ran on 2026-06-24, so any adjustment would be visible in the git history.

**What was the effect size?**
On the pre-registered SWE-bench Verified repeat-mistake benchmark, memory improved the pass rate by **+10.2 points as a single-trial upper bound** (67.3% → 77.6% on 49 paired instances); the multi-seed mean effect is **+0.24 per instance, 95% CI [0, 0.47]**. Both numbers are reported so a reader can pick the framing matched to their decision.

**Is this a peer-reviewed paper?**
No. The [paper](./paper.pdf) is a self-published pre-registered report following the SWE-bench Pro 7-category failure taxonomy (arxiv 2509.16941). Single-author, pre-registered, and every raw artifact (predictions, classifications, constraints, scores) is checked into this repo for independent replay.

**How do I reproduce it?**
See the [Reproduce](#reproduce) section below. Reproducibility hinges on Claude Code 2.1.177 headless, SWE-bench Verified snapshot from OpenAI's Hugging Face release, and per-task 1800s timeout. Any material divergence should open an issue with the seed + judge model.

**What is memory doing in this benchmark?**
Extracting one short directive per baseline Wrong-Solution failure (classified via the SWE-bench Pro 7-category taxonomy) and injecting those directives into the treatment arm's prompt through `world-model-mcp`'s provenance-aware storage (`asserted_by`, `confirmer`, `confirmation_state`, `evidence_type`, `last_decay_at`). Constraints, extraction prompt, and per-arm scripts are all in this repo.

## What is measured

- **Corpus:** 50 SWE-bench Verified tasks across django, sympy, matplotlib, scikit-learn, sphinx. One task dropped due to upstream setup-script issue; 49 paired instances scored.
- **Treatment arm:** constraints extracted from prior baseline failures via the SWE-bench Pro 7-category failure taxonomy, replayed through world-model-mcp's provenance-aware storage (`asserted_by`, `confirmer`, `confirmation_state`, `evidence_type`, `last_decay_at`) with evidence-typed decay (test 180d, bug_fix 365d, user_correction 730d, source_code 365d, session 14d).
- **Judge:** identical to baseline. Judge-model self-reference risk is documented in Section 6 of the paper.
- **Pre-registration:** methodology committed to [DESIGN.md](./DESIGN.md) on 2026-06-17, benchmark ran 2026-06-24. The design predates the result; adjustments would be visible in the git history.

## Reproduce

```bash
git clone https://github.com/SaravananJaichandaran/coding-agent-memory-benchmark.git
cd coding-agent-memory-benchmark

pip install "world-model-mcp>=0.12.14"
export ANTHROPIC_API_KEY=sk-ant-...

# Download SWE-bench Verified separately from OpenAI:
#   https://huggingface.co/datasets/princeton-nlp/SWE-bench_Verified
# (Not redistributed here per OpenAI's data terms.)
python scripts/task_setup.py --verified /path/to/verified.parquet --out tasks.jsonl

python scripts/orchestrator.py --tasks tasks.jsonl --arm baseline --out baseline_results.jsonl
python scripts/failure_classifier.py --results baseline_results.jsonl --out baseline_classified.jsonl
python scripts/learning_hook.py --classified baseline_classified.jsonl --out constraints.json
python scripts/orchestrator.py --tasks tasks.jsonl --arm treatment --constraints constraints.json --out treatment_results.jsonl
python scripts/score.py --baseline baseline_results.jsonl --treatment treatment_results.jsonl
```

Expected: within a few points of the headline. Any material divergence should file an issue with the seed and judge model.

## What is here

- [paper.md](./paper.md), [paper.pdf](./paper.pdf) — full write-up. Zenodo DOI: [10.5281/zenodo.21076824](https://doi.org/10.5281/zenodo.21076824) (v0.9.2) or concept DOI [10.5281/zenodo.20834508](https://doi.org/10.5281/zenodo.20834508) (resolves to latest).
- [DESIGN.md](./DESIGN.md) — pre-registration document (2026-06-17).
- [SEED_PLAN.md](./SEED_PLAN.md) — multi-seed replication methodology.
- [RESULTS.md](./RESULTS.md) — headline table and per-repository breakdown.
- [scripts/](./scripts/) — pipeline: task setup, orchestrator, failure classifier, learning hook, scorer, multi-seed aggregation.
- [baseline_results.jsonl](./baseline_results.jsonl), [baseline_classified.jsonl](./baseline_classified.jsonl), [constraints.json](./constraints.json), [multi_seed_summary_seed2.json](./multi_seed_summary_seed2.json) — reference data. Treatment results, retry data, and per-task progress logs are not shipped in this repo; regenerate with the orchestrator.

## Related work

The paper cites the standard corpus:

- SWE-bench and SWE-bench Verified (Princeton NLP, OpenAI curated subset)
- SWE-bench Pro failure taxonomy — used verbatim for the 7-category classifier
- LongMemEval and LoCoMo — conversation-memory benchmarks; complementary, not comparable (they measure single-agent long-context conversation, not coding-task recurrence)

## Provenance

- **Paper:** [CC-BY 4.0](./LICENSE-PAPER).
- **Code and data:** [MIT](./LICENSE).
- **Author:** Saravanan Jaichandaran (independent, [world-model-mcp](https://github.com/SaravananJaichandar/world-model-mcp) maintainer).

## Cite

```
Jaichandaran, S. (2026). Persistent memory for AI coding agents: a pre-registered
SWE-bench Verified benchmark (v0.9.2). Zenodo. https://doi.org/10.5281/zenodo.21076824
```

## Related repositories

- [world-model-mcp](https://github.com/SaravananJaichandar/world-model-mcp) — the temporal knowledge graph MCP server whose provenance layer this benchmark tests.
- [world-model-mcp-typescript-sdk](https://github.com/SaravananJaichandar/world-model-mcp-typescript-sdk) — TypeScript client for the same server.
