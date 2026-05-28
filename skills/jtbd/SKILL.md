---
name: jtbd
description: Write a Jobs-To-Be-Done file for a feature. Run first, before any RFC. MANDATORY TRIGGERS: new feature request, "write JTBD", "define user jobs", /jtbd <name>. DO NOT trigger for: bug fixes, RFCs already in progress, pure refactors.
user_invocable: true
---

# jtbd (`/jtbd`)

Produces a JTBD file at `jtbd/<feature-name>.md`. Run before any RFC — the JTBD anchors what problem is being solved and for whom.

## Required inputs

- Feature name or ticket ID (e.g. `user-notifications`, `ISSUE-123`, or a plain description)
- Optional: existing JTBD file to refine

## Steps

### 1. Gather context

Run in parallel:
- Check if `jtbd/<feature-name>.md` already exists — if so, read it and offer to refine instead of overwrite
- If a ticket ID is given, fetch the issue from your issue tracker (title, description, acceptance criteria)
- Search `rfcs/` for any existing RFC matching the feature name

### 2. Draft JTBD statements

Write 2–5 statements in the format:
> "When [situation], I want to [motivation], so I can [outcome]."

Rules:
- The situation is the user's context, not ours
- The motivation is what they want to accomplish, not a feature request
- The outcome is the value they get — measurable if possible
- Avoid implementation details in any field

### 3. Write the file

Save to `jtbd/<feature-name>.md` using this structure:

```markdown
# JTBD: [Feature Name]

**Date:** YYYY-MM-DD
**Author:**
**Ticket / PR:**

---

## Jobs to Be Done

1. When …, I want to …, so I can ….
2. …

---

## User Validation

- [ ] Reviewed JTBD statements with a real user
- Notes:

---

## What We Decided to Build

_Given the jobs above, we are building:_

- …

_What we are explicitly not building (and why):_

- …

---

## Did We Solve the Job?

_Fill in after shipping._

- [ ] The job is solved
- Evidence / user feedback:
```

### 4. Offer next step

After writing, prompt:
> "JTBD written at `jtbd/<feature-name>.md`. Ready to run `/rfc <feature-name>` next?"

## Handling edge cases

- **No ticket ID, vague description**: ask one clarifying question — "Who is the primary user for this feature?" — then proceed
- **File already exists**: read it, summarise current jobs, ask whether to refine or replace
- **Multiple personas**: write separate job groups labelled by persona

## Quality check

Before finishing, verify:
- [ ] Every statement uses the "When / I want / so I can" format
- [ ] No statement mentions a UI element or implementation detail
- [ ] The outcomes are distinct (not restating the same job in different words)
- [ ] File saved at the correct path

## Example interaction

```
/jtbd user-notifications
```

Expected output shape:
```
# JTBD: User Notifications

## Jobs to Be Done

1. When an important event occurs in the system, I want to be alerted immediately,
   so I can take action before the situation worsens.
2. When I return after time away, I want to see a summary of what I missed,
   so I can quickly get back up to speed without reading every update.
...
```
