---
name: ux-review
description: Review a frontend feature or PR against UX best practices and the RFC spec. MANDATORY TRIGGERS: after any frontend change is visible in browser, /ux-review <feature or PR>, "review the UX", "check UX gaps", "does this follow our patterns?". DO NOT trigger for: backend-only changes, pre-implementation design reviews (use /ux-rfc instead).
user_invocable: true
---

# ux-review (`/ux-review`)

Reviews a feature implementation against the RFC spec, JTBD, and your project's UX best practices. Produces a structured report with passes, issues (prioritised), and JTBD alignment.

## Required inputs

- Feature name, PR number, worktree name, or localhost URL
- Optional: `--chrome` flag for live browser review

## Steps

### 1. Gather context (parallel)

- Read `rfcs/<feature-name>.md` — extract screen specs, flows, and acceptance criteria
- Read `jtbd/<feature-name>.md` — extract user jobs to verify against
- Read `ux-ui/best-practices.md` if it exists in the project — load the shared UX pattern vocabulary
- If a worktree/directory is given: read the key frontend component files
- If a URL is given and `--chrome` is available: take a screenshot of each relevant screen state

### 2. Identify what to review

Build a checklist of screen states from the RFC:
- List every named state (empty, loading, error, filled, success)
- List every user flow step that has a visual component

### 3. Review each element

For each screen state or flow step, check against:

| Pattern | What to look for |
|---------|-----------------|
| Affordance & Signposting | Are interactive elements obvious? Are errors clearly communicated? |
| Progressive Disclosure | Is complexity hidden until needed? |
| Validation timing | Does validation fire at the right moment (not too early, not too late)? |
| Empty states | Are empty/loading/error states handled gracefully? |
| JTBD alignment | Does this screen help the user accomplish their stated job? |
| RFC conformance | Does the implementation match the RFC wireframes and spec? |

### 4. Produce the review report

Save to `reviews/<feature-name>-ux-review.md`:

```markdown
# UX Review — [Feature Name]

**Date:** YYYY-MM-DD
**Feature:** [name]
**RFC:** `rfcs/<feature-name>.md`
**Worktree / URL:** [...]
**Review type:** [Implementation vs RFC / Live browser review]

---

## Passes

- **[Pattern]** — [what's working and why]

---

## Issues

| # | Pattern | Current behaviour | RFC spec | Priority |
|---|---------|------------------|----------|----------|
| 1 | [pattern] | [what it does] | [what RFC says] | Must fix / Should fix / Nice to have |

---

## JTBD Alignment

| Job | Addressed? | Notes |
|-----|-----------|-------|
| [job statement] | ✅ / ✅ Partial / ❌ | [note] |

---

## Escalation to Designer

### 🔴 Issue #N — [title]
[Detail for designer: what changed, why it matters, what RFC says]
```

### 5. Offer next steps

For each "Must fix" issue, suggest the specific code change needed.

## Handling edge cases

- **No RFC**: review against JTBD only and general best-practices patterns; note the gap
- **No `ux-ui/best-practices.md`**: use the patterns in this skill as defaults
- **No chrome / can't see UI**: review source code for the patterns; note which checks require visual confirmation
- **PR not yet merged**: review against the diff, flag any patterns that need visual verification before merge

## Quality check

- [ ] Every RFC screen state has a corresponding pass or issue
- [ ] Every issue has a priority assigned
- [ ] JTBD alignment table covers all jobs from the JTBD file
- [ ] "Must fix" issues have a concrete suggested fix
- [ ] Review saved to `reviews/` directory
