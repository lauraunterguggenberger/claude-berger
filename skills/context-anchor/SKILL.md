---
name: context-anchor
description: Session context manager. Loads ticket context to start a session, verifies correctness before pushing, switches focus between tickets, or re-anchors after an interruption. Trigger with /ca, or natural language like "what was I working on?" or "where was I?".
user_invocable: true
---

# context-anchor (`/ca`)

Manages session context across four modes. Detect the mode from the arguments or natural language.

## Mode detection

| Invocation | Mode |
|-----------|------|
| `/ca ISSUE-123 [ISSUE-456 ...]` | **init** — start a session on 1–3 tickets |
| `/ca verify` | **verify** — check correctness before pushing |
| `/ca switch ISSUE-123` | **switch** — change active ticket focus |
| `/ca anchor` | **anchor** — re-entry after interruption |
| "What was I working on?" | **anchor** |
| "Start my session — I'm on ISSUE-XXX" | **init** |
| "Is my code correct?" | **verify** |
| "Switch me to the X ticket" | **switch** |
| "I got pulled into a meeting / lost the thread" | **anchor** |

---

## Mode: init

Load full context for 1–3 tickets and orient the session.

### Steps

1. **For each ticket ID**, gather context in parallel:
   - Fetch the issue from your issue tracker (title, status, description, assignee, priority)
   - Search `rfcs/` for an RFC matching the ticket ID
   - Search `jtbd/` for a JTBD file matching the ticket ID
   - Search `reviews/` for any review notes
   - Identify active worktrees/branches containing the ticket ID (`git -C <wt> branch --show-current`)

2. **Surface a session brief** for each ticket:
   ```
   ## ISSUE-XXX — <title> [<status>]
   RFC: <path or "none">
   JTBD: <path or "none">
   Worktrees: backend-<name> (branch: ...) | frontend-<name> (branch: ...)
   Open PRs: #NNN (<status>) | none
   Last action: <inferred from git log or RFC notes>
   ```

3. **If multiple tickets**, identify the recommended starting point:
   - Prefer tickets that are BLOCKED or In Review over In Progress
   - Note dependency order if tickets are related

4. **Proactively surface the next atomic action** for the primary ticket:
   > "Recommended first action: ..."

---

## Mode: verify

Check correctness of the current state before pushing. Run appropriate checks based on what's active.

### Steps

1. **Identify what's active** — infer from context or ask:
   - Which worktrees/directories are relevant
   - What ticket/feature is being verified

2. **For each active worktree, run the appropriate checks:**

   **Backend worktree:**
   ```bash
   uv run pytest tests/ -x -q   # or your project's test command
   uv run mypy .                 # or your project's type check command
   ```

   **Frontend worktree:**
   ```bash
   pnpm tsc --noEmit   # or npx tsc --noEmit
   pnpm lint
   ```

   **ML worktree:** No typed checks — verify notebooks/scripts run without errors if applicable.

3. **Check for correctness criteria** — look in the RFC or JTBD for explicit acceptance criteria. If found, state them and assess whether the current code satisfies them.

4. **Report verdict per worktree:**
   - ✅ READY — all checks pass
   - ❌ BLOCKED — specific failure with file:line
   - ⚠️ WARNINGS — non-blocking issues

5. **If anything is BLOCKED**, suggest the exact next fix action (not a vague "fix the error").

---

## Mode: switch

Re-focus the session on a different ticket without losing state.

### Steps

1. **Summarize the current ticket state** before switching (one line per open item):
   > "Leaving ISSUE-XXX: [uncommitted changes in backend-X | PR #N awaiting review | all clear]"

2. **Load context for the target ticket** using the same steps as **init** (single ticket).

3. **Recommend the next action** for the target ticket.

---

## Mode: anchor

Re-orient after an interruption. The goal is to surface exactly where the user left off in one glance.

### Steps

1. **Infer active tickets** from:
   - Recent conversation context (ticket IDs mentioned)
   - Running processes (`docker ps` or equivalent)
   - Git worktrees/branches with uncommitted changes (`git -C <wt> status --short`)
   - Most recently modified files across active worktrees

2. **For each active ticket/worktree, report:**
   ```
   ## Where you left off — ISSUE-XXX

   Worktree: backend-<name> | frontend-<name>
   Branch: <user>/<branch>
   Uncommitted: <N files changed / clean>
   Last commit: "<message>" (<time>)
   PR: #NNN — <CI status> | no PR yet

   Last action: <inferred>
   Next action: <specific recommendation>
   ```

3. **If there are open blockers** (failing CI, unresolved review comments), surface them immediately — don't bury them.

4. **Suggest a single next atomic action** to resume.

---

## General rules

- Always use `git -C <absolute-path>` — never `cd` into worktrees.
- When checking PRs, use `gh pr view <number> --repo <owner>/<repo> --json state,reviews,statusCheckRollup`.
- If an issue tracker MCP is available (Linear, GitHub, Jira), fetch issue details directly. If not, ask the user for status.
- Keep the output dense and scannable — this is a quick-orient tool, not a report.
