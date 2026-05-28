---
name: rfc
description: Write a technical RFC (design document) for a feature. Run after JTBD. Saves to rfcs/<feature-name>.md. MANDATORY TRIGGERS: /rfc <name>, "write an RFC", "draft a design doc", "start the RFC for". DO NOT trigger for: bug fixes without design decisions, JTBD-only tasks, pure UX work (use /ux-rfc instead).
user_invocable: true
---

# rfc (`/rfc`)

Produces a technical RFC at `rfcs/<feature-name>.md`. Reads the JTBD file first, then investigates the codebase to produce an architecture-grounded design doc.

## Required inputs

- Feature name or ticket ID
- Optional: `ux` flag — write a UX/UI-focused RFC instead of technical

## Steps

### 1. Gather context (parallel)

- Read `jtbd/<feature-name>.md` if it exists — extract jobs and scope
- Read `rfcs/<feature-name>.md` if it exists — offer to refine rather than overwrite
- Fetch the issue from your issue tracker if a ticket ID is given
- Identify relevant worktrees or branches (`backend-*`, `frontend-*`, `ml-*`) and read key files

### 2. Investigate the architecture

Map the relevant code layers:

| Layer | What to look for |
|-------|-----------------|
| Endpoints/handlers | Existing endpoints with similar patterns |
| Models/schemas | Data models, TypeScript types |
| Async tasks | Background jobs that may need extension |
| Database | Queries/mutations relevant to this feature |
| Tests | Existing test patterns to follow |

Build an architecture table: file path, role, what changes.

### 3. Draft candidate approaches

Write 2–3 approaches with tradeoffs. Be concrete — include file paths and code sketches. Pick a recommended approach and state why.

### 4. Write the RFC file

Save to `rfcs/<feature-name>.md` using this structure:

```markdown
# RFC: [Feature Name]

**Author:**
**Date:** YYYY-MM-DD
**Status:** Draft
**Ticket / PR:**
**JTBD doc:** `../jtbd/<feature-name>.md`

---

## Goal
[One or two sentences — what and why now]

---

## Architecture Analysis

| Layer | File | Role |
|-------|------|------|

### Data flow
1. …

---

## Candidate Approaches

### Approach A: [Name]
### Approach B: [Name]

**Recommended:** Approach X — because …

---

## Data Contract
[Endpoint, request/response shape — or note "see /data-contract"]

---

## Test Plan
- [ ] …

---

## Out of Scope
- …

---

## Open Questions
- [ ] …

---

## Decisions Log

| Decision | Rationale | Date |
|----------|-----------|------|
```

### 5. Offer next steps

> "RFC saved at `rfcs/<feature-name>.md`. Suggested next: `/data-contract <feature-name>` then `/backend-architecture <feature-name>`."

## Handling edge cases

- **No JTBD file**: proceed but note the gap — write a one-line "inferred jobs" section at the top
- **UX flag**: omit code/file-path sections; focus on user flows, wireframe descriptions, and component states — suitable for sharing with designers
- **File already exists**: read it, summarise current state, ask whether to refine specific sections or start fresh
- **Very large codebase**: read handler and model files only — skip test files until Test Plan section

## Quality check

- [ ] Architecture table has real file paths (not placeholders)
- [ ] At least two approaches with genuine tradeoffs
- [ ] Recommended approach stated with reasoning
- [ ] Test plan has at least happy path + primary failure mode
- [ ] File saved at correct path
