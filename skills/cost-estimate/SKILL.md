---
name: cost-estimate
description: Estimate token usage and cost for a task before starting. MANDATORY TRIGGERS: L or XL tasks, CTO/manager asks about spending/cost/token usage, /cost-estimate <task>, "how much will this cost?". DO NOT trigger for: XS/S tasks, informational questions, ongoing work already in progress.
user_invocable: true
---

# cost-estimate (`/cost-estimate`)

Surfaces a cost and session estimate before beginning expensive work. Prevents surprise spend on L/XL tasks.

## Required inputs

- Task description (feature name, RFC name, or plain description)
- Optional: `explain` flag — break down assumptions in detail

## Size reference table

| Size | Description | Sessions | Est. tokens | Est. cost (Sonnet 4.6) |
|------|------------|----------|-------------|------------------------|
| XS | Bug fix, config tweak, single-file change | 1–2 | 200k–500k | $0.60–$1.50 |
| S | Small feature, one repo, no RFC | 2–4 | 500k–1.5M | $1.50–$4.50 |
| M | Feature with RFC, one repo | 4–8 | 1.5M–3M | $4.50–$9 |
| L | Feature across multiple repos, RFC + manual test | 8–15 | 3M–6M | $9–$18 |
| XL | Multi-repo, RFC + data contract + iterations + UX review | 15–30 | 6M–15M | $18–$45 |

**Multipliers:** ×1.5 if Opus 4.6 needed · ×1.2 if browser automation · ×1.3 per additional repo beyond first

## Steps

### 1. Classify the task

Determine size by asking:
- Does it span multiple repos? (+1 size)
- Does it require an RFC? (+1 size)
- Does it require a data contract? (+1 size)
- Does it require manual testing or UX review? (+1 size)
- Is the scope well-defined or exploratory? (exploratory = upper end of range)

### 2. Apply multipliers

- Count how many repos are involved
- Check if Opus is needed (complex reasoning, architecture review)
- Check if browser automation is needed (UX review, screenshot capture)

### 3. Output the estimate

```
## Cost Estimate — <task name>

Size: <XS / S / M / L / XL>
Sessions: <range>
Tokens: <range>
Est. cost: <range>

Assumptions:
- <key assumption 1>
- <key assumption 2>

Multipliers applied: <none / list>

Adjusted estimate: <final range if multipliers applied>
```

### 4. Recommend proceed or pause

- XS/S/M: "This is within normal range — safe to proceed."
- L: "This is an L task. Confirm before starting?"
- XL: "This is an XL task (~$18–$45). Recommend breaking into phases. Confirm?"

## Handling edge cases

- **`/cost-estimate explain`**: break down every assumption in detail, show the math
- **Unclear scope**: give a range for best-case (well-scoped) and worst-case (exploratory)
- **Already in progress**: report remaining cost based on what's left, not total

## Quality check

- [ ] Size classification matches the criteria table
- [ ] All applicable multipliers applied
- [ ] Recommendation to proceed/pause is explicit
