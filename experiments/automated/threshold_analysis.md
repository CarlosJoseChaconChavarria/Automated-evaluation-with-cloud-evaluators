# Threshold Analysis: Pass/Fail Configuration

## Overview

The `PASS_THRESHOLD` constant in `src/evaluators/evaluate_agent.py` controls what score
counts as a passing evaluation. Scores are on a 1–5 scale produced by GPT-4.1 as judge.

This branch changes the threshold from **3.0 → 4.0** to measure the impact on pass rates.

## Baseline Results (threshold = 3.0)

From the first manual run (eval `eval_bca336dd5d154b15b3ded7c98cb9fa9c`):

| Evaluator         | Pass Rate |
|-------------------|-----------|
| Intent Resolution | 100%      |
| Relevance         | 100%      |
| Groundedness      | 88%       |

## Expected Impact (threshold = 4.0)

A stricter threshold means responses scoring 3.x are now failures.
Pass rates are expected to drop across all metrics — how much depends on
the score distribution. Results will be posted automatically as a PR comment
when this branch's evaluation run completes.

| Evaluator         | Threshold 3.0 | Threshold 4.0 | Delta |
|-------------------|---------------|---------------|-------|
| Intent Resolution | 100%          | TBD           | TBD   |
| Relevance         | 100%          | TBD           | TBD   |
| Groundedness      | 88%           | TBD           | TBD   |

## Threshold Recommendations

| Use Case                        | Recommended Threshold | Rationale |
|---------------------------------|-----------------------|-----------|
| Development / iteration         | 3.0                   | Permissive — catches only clearly bad responses |
| Pre-production / staging gate   | 4.0                   | Stricter — rejects mediocre responses |
| Safety-critical topics          | 4.5                   | Near-perfect required for high-risk advice |

## Risk Tradeoffs

- **Lower threshold (3.0):** More false positives — poor responses pass. Faster iteration.
- **Higher threshold (4.0):** More false negatives — good responses may fail. Safer for production gating.
- **Recommendation:** Use 4.0 as the production CI/CD gate. Keep 3.0 for local dev runs.
