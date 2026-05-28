---
name: data-violations
description: Review a dataset or data source for schema violations after a pipeline run. Categorises violations as major/minor, assigns P0/P1/P2 priority, produces a structured report. MANDATORY TRIGGERS: /data-violations <dataset>, "audit data quality", "check violations for", "review data sources". DO NOT trigger for: pre-run validation, general data debugging, schema design.
user_invocable: true
---

# data-violations (`/data-violations`)

Audits one or more data sources for schema violations after a pipeline run. Produces a structured report with per-source violation tables, priority assignments, and a plain-English stakeholder summary.

## Required inputs

- Dataset or data source identifier (e.g. `sales_data`, `user_events`, `acme_corp`)
- Optional: context about which run triggered the review (e.g. "after March pipeline run")

## Priority rules

| Priority | Condition |
|----------|-----------|
| **P0** | Multiple major violations on one data source — run results likely invalid |
| **P1** | Single major violation OR >10 minor violations — results suspect, needs investigation |
| **P2** | Fewer than 10 minor violations only — results probably valid, low urgency |

**Major violations:** missing required columns, wrong data types, duplicate rows, date gaps >7 days, >10% null rate in a required field.

**Minor violations:** inconsistent casing, unexpected extra columns, minor null rate (<10%) in optional fields, date format inconsistency in non-key columns.

## Steps

### 1. Gather context (parallel)

- Fetch the data source(s) for the specified dataset/run
- Identify which run is being reviewed (latest by default)
- Check if a previous violation report exists for this dataset — if so, note which violations are new vs recurring

### 2. Audit each data source

For each data source, check:
- Required columns present
- Column data types match expected schema
- Date column format (YYYY-MM-DD required)
- Duplicate row count
- Null rates per column
- Date continuity (no unexpected gaps)
- Row count reasonableness (flag extreme outliers vs prior runs)

### 3. Build the violation table

For each data source with violations:

```markdown
### <DataSourceName>

| Violation | Type | Count / Rate | Example |
|-----------|------|-------------|---------|
| [description] | major / minor | [number or %] | [example value] |

**Priority: P0 / P1 / P2**
Reason: [one sentence]
```

### 4. Produce the report

```markdown
# Data Violations — <dataset> (<date>)

**Run context:** [pipeline name / date]
**Overall priority:** P0 / P1 / P2

---

## Stakeholder summary

[One sentence in plain English — no jargon. Suitable for pasting into Slack or a doc.]

---

## Per-source violations

[violation tables per source]

---

## Recurring violations

[List any violations that also appeared in previous reports]

---

## Recommended actions

- [ ] [Specific fix for P0/P1 violations]
- [ ] Log to your issue/doc tracker
```

### 5. Log to tracker (if requested)

If the user asks, open or create a page in their issue/doc tracker (Notion, Linear, GitHub Issues, etc.) and add a new entry for this dataset/run. Follow any MCP tool safety rules in CLAUDE.md before writing.

## Handling edge cases

- **No previous run data**: note "first run for this dataset" and skip the recurring violations section
- **Dataset not found**: ask for the correct identifier before proceeding
- **P0 violation**: flag explicitly before the full report — "⚠️ P0 violation found — results for this dataset may be invalid"

## Quality check

- [ ] Every data source with violations has a priority assigned with a reason
- [ ] Overall priority reflects the worst per-source priority
- [ ] Stakeholder summary is one sentence and jargon-free
- [ ] Recommended actions are specific (not "fix the data")
