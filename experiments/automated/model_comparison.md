# Model Comparison: GPT-4.1 vs GPT-4.1-mini

## Overview

This PR runs the same 89-item evaluation dataset using **GPT-4.1-mini** as the judge model
instead of **GPT-4.1**, to quantify the quality-cost tradeoff.

The model is controlled by a single line in `src/evaluators/evaluate_agent.py`:

```python
model_deployment_name = os.environ.get("MODEL_NAME", "gpt-4.1-mini")
```

Or via `MODEL_NAME` in `.env` / GitHub Actions variable `vars.MODEL_NAME`.

---

## Quality Score Comparison

| Evaluator         | GPT-4.1 (baseline) | GPT-4.1-mini (this run) | Delta |
|-------------------|--------------------|-------------------------|-------|
| Intent Resolution | 100%               | TBD                     | TBD   |
| Relevance         | 100%               | TBD                     | TBD   |
| Groundedness      | 88%                | TBD                     | TBD   |
| Elapsed time      | ~50 min            | TBD                     | TBD   |

> TBD values will be posted automatically as a PR comment when this run completes.

---

## Cost Analysis (estimated)

Azure OpenAI pricing as of 2025 (per 1M tokens):

| Model         | Input     | Output    | Relative cost |
|---------------|-----------|-----------|---------------|
| GPT-4.1       | $2.00     | $8.00     | 1×            |
| GPT-4.1-mini  | $0.40     | $1.60     | ~0.20×        |

For 89 evaluation items (estimated ~500 input + ~300 output tokens each):
- **GPT-4.1:** ~$0.09 per run
- **GPT-4.1-mini:** ~$0.02 per run

At scale (1,000 items/day): GPT-4.1 ≈ $1.00/day vs GPT-4.1-mini ≈ $0.20/day.

---

## Recommendation

| Use Case                        | Recommended Model | Rationale |
|---------------------------------|--------------------|-----------|
| Production CI/CD gate           | GPT-4.1            | Higher accuracy, cost acceptable per run |
| High-frequency monitoring       | GPT-4.1-mini       | ~5× cheaper, acceptable if scores align |
| Development iteration (fast)    | GPT-4.1-mini       | Speed and cost savings during dev cycles |
| Safety-critical evaluation      | GPT-4.1            | More reliable reasoning on edge cases |

**Final recommendation:** Use GPT-4.1-mini if quality delta is < 5% across all metrics.
Update this table once the PR comment scores are available.
