---
name: ml-model-evaluation
description: Evaluate the quality of a regression or predictive ML model run: model fit, feature coefficients, output completeness, and stability vs prior runs. MANDATORY TRIGGERS: /ml-model-evaluation <model>, "evaluate the model results", "check model quality", "validate model outputs". DO NOT trigger for: data input validation (use /data-violations), feature design, infra debugging.
user_invocable: true
---

# ml-model-evaluation (`/ml-model-evaluation`)

Evaluates the quality and reasonableness of a regression or predictive model run. Produces a structured evaluation report covering model fit, coefficient sanity, output completeness, and stability vs prior runs.

## Required inputs

- Model or run identifier (e.g. `sales-forecast`, `churn-model-v2`, or a run ID)
- Optional: specific run ID or date range to evaluate
- Optional: `--compare <run-id>` — diff against a previous run

## Evaluation dimensions

| Dimension | What to check |
|-----------|--------------|
| **Model fit** | R², MAPE, residual distribution — within acceptable ranges? |
| **Coefficient sanity** | Are feature coefficients directionally plausible? Any at prior/constraint bounds? |
| **Prediction reasonableness** | Do predictions fall within expected ranges? No implausible outliers? |
| **Output completeness** | Are all expected output files/artifacts present? |
| **Stability** | If comparing runs: did coefficients or predictions shift more than expected? |

## Steps

### 1. Gather context (parallel)

- Fetch the run outputs for the specified model/run from your data store
- Load the model's expected output schema (from config, RFC, or prior runs)
- If `--compare` given: fetch the previous run for diffing

### 2. Evaluate model fit

Check standard regression metrics:
- R² ≥ 0.85 (flag below 0.80 as concerning)
- MAPE ≤ 15% (flag above 20% as concerning)
- Residuals: check for obvious autocorrelation or heteroscedasticity

Adjust thresholds to the model's domain if documented in the project's RFC or config.

### 3. Evaluate coefficient sanity

For each feature coefficient:
- Is the sign (positive/negative) directionally plausible given domain knowledge?
- Is the magnitude within a reasonable range?
- Is any coefficient pegged at a constraint bound? (suggests the data isn't informing that parameter)

### 4. Evaluate prediction reasonableness

- Do predictions fall within the historically observed range (± reasonable margin)?
- Are there any implausibly large or small individual predictions?
- Does the aggregate prediction align with known ground truth where available?

### 5. Check output completeness

Verify all expected artifacts are present per the model's defined output schema:
- Summary/metrics JSON or report
- Key charts or visualisation files
- Export-ready outputs (if applicable)

### 6. Produce the report

```markdown
# Model Evaluation — <model-name> (<run date>)

**Run ID:** [...]
**Overall verdict:** ✅ Pass / ⚠️ Review / ❌ Fail

---

## Model fit

| Metric | Value | Threshold | Status |
|--------|-------|-----------|--------|
| R²     | X.XX  | ≥ 0.85    | ✅ / ⚠️ / ❌ |
| MAPE   | X.X%  | ≤ 15%     | ✅ / ⚠️ / ❌ |

---

## Coefficient sanity

| Feature | Value | Direction | Flag? |
|---------|-------|-----------|-------|

---

## Prediction reasonableness

[Summary of range checks and any outlier flags]

---

## Output completeness

- [x] Metrics summary
- [x] Predictions file
- [ ] ❌ Visualisation output — missing

---

## Comparison to previous run (if --compare)

| Metric | Previous | Current | Delta | Flag? |
|--------|---------|---------|-------|-------|

---

## Recommendations

- [ ] [Specific action for any ⚠️ or ❌ items]
```

## Handling edge cases

- **First run for this model**: skip the comparison section; note "baseline run — no prior to compare"
- **❌ Fail verdict**: surface the specific failing checks prominently before the full report
- **Missing outputs**: flag immediately as a potential pipeline error before evaluating what exists
- **No documented thresholds**: use the defaults in this skill; note they may need tuning per domain

## Quality check

- [ ] Every dimension has an explicit pass/warn/fail
- [ ] All ⚠️ and ❌ items have a recommended action
- [ ] Coefficient flags include the specific value and why it's suspicious
- [ ] Output completeness check lists every expected artifact
