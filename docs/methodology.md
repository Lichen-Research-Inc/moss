# Moss Benchmark Methodology

*Last updated: 2026-04-17*

This document describes the reproducibility protocol used for all Moss benchmark results published in this repository.

## Principles

1. **Seed-pinned inference** — All stochastic LLM calls pass an explicit random seed. Re-running the same benchmark with the same configuration and seed produces byte-identical outputs.
2. **Dated judge snapshots** — LLM-as-judge evaluations pin to dated model snapshots (e.g., `gpt-4o-2024-11-20`), not floating aliases. Judges can change underneath floating aliases.
3. **Configuration-as-executed capture** — Every benchmark run logs the full argparse namespace, git commit hash, and hostname, so reported configurations cannot silently diverge from actual execution.
4. **Exclusive-resource eval** — Benchmarks run with no concurrent load on the inference server. BF16 GPU non-determinism is sensitive to batch-size fluctuations caused by concurrent requests.
5. **Per-category N reporting** — Every per-category accuracy is reported with its N. Small-N categories (e.g., `common-sense` at N=7 on conv5) exhibit larger per-category volatility than large-N categories, and the distinction matters when interpreting deltas.

## Variance Characterization

We ran N=10 identical-configuration benchmarks on conversation 4 to measure the noise floor:

| Category | σ (per-run) | ±2σ band (pp) |
|---|---|---|
| adversarial | 0.008 | ±1.6 |
| multi-hop | 0.011 | ±2.1 |
| temporal | 0.024 | ±4.8 |
| single-hop | 0.052 | ±10.4 |
| common-sense | 0.055 | ±11.1 |
| **overall** | **0.006** | **±1.2** |

**Significance gate**: Changes to the overall score under 1.2 percentage points on a single conversation should not be interpreted as signal. Single-hop and common-sense single-conversation deltas below 5pp are within noise.

## Judge-Confound Control

LLM-as-judge evaluations exhibit non-negligible disagreement on identical predictions. To avoid inflation from permissive judges, we:

- Pin to `gpt-4o-2024-11-20` as our primary judge (strict baseline).
- Report a secondary judge analysis (`gpt-4.1-mini`) for cross-paper comparison.
- Per-category judge-delta analysis available on request pending calibration work.

**Finding**: Judge-model bias is not a uniform additive shift. It is category-specific (largest on common-sense) and answer-style-sensitive (direction of bias depends on the answering model).

## Multi-Seed Protocol (in progress, VER-1342)

For publication-grade reporting, we run N=5 seeds per configuration across two conversations (conv4 and conv5), producing 10 data points per configuration. Final σ figures update this file as data completes.

## Reproducing a Result

Each `benchmarks/locomo_*_results.json` file contains:
- Exact configuration (retrieval channels, category routing, cross-encoder top-k)
- Seed used (default: 42)
- Judge model and date
- Per-category accuracies and Ns
- Full per-conversation breakdown

To reproduce:
1. Install Moss at the commit referenced in the result file's config
2. Apply the listed configuration
3. Run with `--seed 42` (or the listed seed)
4. Judge output with the exact judge model snapshot listed

Any observed delta >2σ from the reported number is a reproducibility bug — please file an issue.

## Related Work

This methodology follows the precedent set by "Quantifying Variance in Evaluation Benchmarks" (Madaan et al., ICLR 2025; arXiv:2406.10229) which studied seed variance across 13 NLP benchmarks. To our knowledge, this is the first application of that protocol to a conversational long-term memory benchmark.

For judge-confound precedent in single-turn RAG, see "Same Input, Different Scores" (arXiv:2603.04417). Our work extends judge-confound analysis to multi-turn conversational memory with per-category analysis.
