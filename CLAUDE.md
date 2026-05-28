# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

**claude_root location:** `/Users/lauraunterguggenberger/workspace/claude_root`

---

## Agent Workstream — Feature Development Flow

Every new feature follows this pipeline. **Use the right specialist agent at each stage automatically — do not wait to be told.**

```
/jtbd → /cost-estimate → /ux-rfc (for designers) + /rfc (for engineers) → /ui-architecture or /backend-architecture → (mock-designer) → /backend-dev → /ux-review
```

| Stage | Agent / Skill | When to invoke |
|-------|--------------|----------------|
| 1. Define jobs | `/jtbd` | New feature request, before any design |
| 1b. Cost estimate | `/cost-estimate` | After JTBD; before RFC. Answers "how much will this cost?" for planning/stakeholders. |
| 2a. UX design doc | `/ux-rfc` | After JTBD; for sharing with designers (Notion). No code, no file paths. |
| 2b. Technical RFC | `/rfc` | After JTBD; engineer's investigation doc (local only). Architecture, code, test plans. |
| 3a. Frontend design | `/ui-architecture` | RFC exists + feature has UI changes |
| 3b. Backend design | `/backend-architecture` | RFC exists + feature has API/task changes |
| 4. Data contract | `/data-contract` | RFC exists; defines request/response schemas |
| 5. Implement backend | `backend-dev` agent | RFC + data contract exist |
| 6. UX review | `/ux-review` | After any frontend change is visible in the browser |

**Proactive rules:**
- If someone describes a new feature and no JTBD file exists → suggest starting with `/jtbd` before writing any code.
- If a CTO or manager asks about spending, cost, or token usage → run `/cost-estimate explain` automatically.
- If a user's request maps to an L or XL task (see cost table below) → proactively run `/cost-estimate <task>` before starting work and surface the estimate. Do not silently begin an expensive task.
- If an RFC exists but the backend hasn't been implemented → use the `backend-dev` agent, which will read the RFC automatically.
- If a PR or frontend change is described → offer to run `/ux-review` without being asked.
- If a bug fix touches exception handling → check the exception drilling rules in `.claude/agents/backend-dev.md`.
- Agents read their own RFC/JTBD files — you don't need to re-paste context into them.

**Task cost reference — trigger `/cost-estimate` if a task is L or XL:**

| Size | Description | Sessions | Est. tokens | Est. cost (Sonnet 4.6) |
|------|------------|----------|-------------|------------------------|
| XS | Bug fix, config tweak, single-file change | 1–2 | 200k–500k | $0.60–$1.50 |
| S | Small feature, one repo, no RFC | 2–4 | 500k–1.5M | $1.50–$4.50 |
| M | Feature with RFC, one repo | 4–8 | 1.5M–3M | $4.50–$9 |
| **L** | **Feature across multiple repos, RFC + manual test** | **8–15** | **3M–6M** | **$9–$18** |
| **XL** | **Multi-repo, RFC + data contract + iterations + UX review** | **15–30** | **6M–15M** | **$18–$45** |

Multipliers: ×1.5 if Opus 4.6 needed · ×1.2 if browser automation · ×1.3 per additional repo beyond first

**Available agents** (in `.claude/agents/`): `planner`, `mock-designer`, `backend-dev`
**Available skills** (via `/skill-name`): `jtbd`, `cost-estimate`, `ux-rfc`, `rfc`, `ux-review`, `backend-architecture`, `ui-architecture`, `data-contract`, `nb-mypy`, `data-violations`, `lmmm-evaluation`

---

## Search Scope

All code lives under a single parent directory (e.g. `~/workdir/<project>`). Never search outside that path.

---

## MCP Tool Safety — Public vs Private Actions

Several MCP integrations are available (Slack, Notion, Linear, GitHub, etc.). Follow these rules to avoid accidental public visibility:

**Allowed freely (private):**
- **Slack:** DM the project owner only. No new channels, no posting to shared/public channels.
- **Notion:** Create private docs, read existing docs.
- **GitHub:** Read PRs, issues, diffs. Push to feature branches. Comment on existing PRs.
- **Linear:** Read issues and projects.

**Never do without explicit permission:**
- Create or post to Slack channels (public or shared)
- Send Slack messages to anyone not explicitly listed
- Create public GitHub issues, PRs with external visibility, or public gists
- Publish or share Notion pages externally
- Any action that makes content visible outside the team

**When in doubt:** ask before sending. The cost of a false "should I send this?" is zero; the cost of an accidental public post is not.

---

## Transformer Design Philosophy

Transformers in the dataset pipeline are **tiny and declarative**. Each one does exactly one thing. They do not do "magic".

- **Tiny**: a transformer does the minimum operation its name describes. No implicit side effects, no bundled cleanup, no auto-fixing of upstream data problems.
- **Declarative**: the transformer pipeline expresses intent — the sequence of steps is the spec. A transformer that silently fixes bad input hides the intent and makes pipelines harder to reason about.
- **No magic**: if the input isn't in the right shape, the transformer does not try to fix it. Instead it **warns** so the user knows something is wrong and can add the correct upstream step themselves. Silent coercion is worse than a visible warning.
  - Example: a date-range filter transformer does not convert date strings to datetimes. It warns if the column is not already a datetime dtype and directs the user to add a date-parsing transformer first.

**Rule of thumb**: if you find yourself writing `pd.to_datetime(df[col])` inside a transformer that isn't the date-parsing transformer, that logic belongs in a separate transformer — not as a side-effect of this one.

---

## Git Commands in Worktrees

Always use `git -C <absolute-path>` — never `cd <worktree> && git ...`. The worktrees are not subdirectories of the shell's cwd; `cd` into them breaks the shell context.

```bash
# Correct
git -C /absolute/path/to/worktrees/nb-feature push

# Wrong — will fail with "cannot change to" error
cd nb-feature && git push
```

---

## tmux Workflow

Full reference at `TMUX.md`. Standard layout: window 0: backend, 1: frontend, 2: git, 3: claude.

Rename windows to reflect what's running (≤10 chars). Add `set-option -g allow-rename off` to `~/.tmux.conf` to prevent overwrites.

---

## Notion

- **Before writing to any Notion page, fetch it first and verify it was created by you.** If the page was created by someone else, treat it as read-only — never modify.
- Creating new pages is always fine.
- This applies to all Notion tools: `notion-update-page`, `notion-create-comment`, etc.

---

## Demo Rules (trial run or live recording)

- **Never make code changes during a demo.** No file edits, no commits, no pushes. Fix in a separate session first.
- **Never rebase during a demo.** Rebasing mid-trial risks merge conflicts and broken worktrees.
- If a bug surfaces during a trial run, note it and address it after the trial.

---

## Running the Backend Service Locally (Docker)

Each backend worktree needs **two local-only changes** — neither is committed:
1. Add `env_file: .env` to both `web:` and `worker:` services in `docker-compose.yml`
2. Symlink `.env`: `ln -sf <canonical-env-path> <worktree>/.env`

Without (1): server crashes on missing secrets and async workers hang.

---

## Running the Jupyter/ML Service Locally (Docker)

No `.env` symlink or `docker-compose.yml` edits needed — uses cloud credential volume mounts. Jupyter on **port 8000** (no token).

**Always use `--recursive` for submodule init** — `--init` alone leaves nested submodules empty:
```bash
git -C /path/to/worktrees/<name> submodule update --init --recursive
```

---

## Running the Frontend Locally

Run `./setup-fe-worktree.sh <name> <port>` before starting — it patches `.env.local` with the correct auth redirect URI. Port 5175 is reserved for the main branch.

### Port reference

| Port | Use |
|------|-----|
| **5175** | Main (reserved) |
| **5176** | Primary worktree |
| 5178–5179 | Parallel worktrees |
| 5173 | Available (no auth redirect registered) |

---

## Manual Testing — Pre-stage Checklist

Run `./stage-info.sh <backend-wt> <frontend-wt>` (or `./stage-info.sh --fe-only <frontend-wt>`) first. Expected uncommitted files: `docker-compose.yml` (backend) and `.env.local` (frontend).

⚠️ **`generate_schema.py` has a hardcoded `modules` list** — every new transformer must be added manually or it won't appear in the frontend picker. Most common reason a transformer seems missing.

After adding a transformer, regenerate and copy schema:
```bash
PYTHONPATH=. uv run python tasks/dataset_generator/generate_schema.py
cp tasks/dataset_generator/schema.json ../<frontend-wt>/packages/transformers/schema.json
git -C ../<frontend-wt> commit -am "chore: sync schema.json from <backend-branch>"
```

Note: copy macOS screenshots with `find … -print0 | while IFS= read -r -d '' f` — plain `cp` fails on screenshot filenames.

---

## Running Tests & Common Commands

### Backend service (any backend worktree)

```bash
# Tests
uv run pytest tests/
uv run pytest tests/path/to/test_specific.py -v
uv run pytest tests/ --cov=. --cov-report=term-missing

# Lint / format
ruff format .
ruff check --select I --fix .

# Generate transformer schema.json
PYTHONPATH=. uv run python tasks/dataset_generator/generate_schema.py
cp tasks/dataset_generator/schema.json ../frontend/packages/transformers/schema.json
```

Via Docker — **always use the worker container, not web**:
```bash
docker exec <project>-worker-1 uv run pytest /app/tests/
docker exec <project>-worker-1 uv run python /app/tasks/dataset_generator/generate_schema.py
```

---

## Worktree Layout

Feature worktrees live at `../worktrees/`, **not** inside `claude_root/`. This keeps `claude_root` small so Claude Code's internal worktrees (`.claude/worktrees/*`) stay lightweight.

```
project-root/
  claude_root/   ← scripts, docs, rfcs, skills + main-branch refs only
  worktrees/     ← all feature worktrees: backend-*, frontend-*, ml-*
```

**Always create new worktrees into `../worktrees/`:**

```bash
# backend feature branch
git -C /path/to/claude_root worktree add -b <user>/<branch> ../worktrees/nb-<name> backend/main

# frontend feature branch
git -C /path/to/claude_root worktree add -b <user>/<branch> ../worktrees/fe-<name> frontend/main

# ml feature branch
git -C /path/to/claude_root worktree add -b <user>/<branch> ../worktrees/ml-<name> ml/main
```

**After creating a backend worktree:**
```bash
ln -sf /path/to/canonical/.env /path/to/worktrees/<name>/.env
# then add env_file: .env to docker-compose.yml (see Running the Backend section)
```

**After creating a frontend worktree:**
```bash
./setup-fe-worktree.sh <name> 5176   # primary port; use 5178/5179 for parallel worktrees
```

All scripts (`setup-fe-worktree.sh`, `stage-info.sh`, `teardown-worktree.sh`, `start-stage.sh`, `stop-stage.sh`) resolve worktree names by checking `claude_root/` first, then `../worktrees/` — no special flags needed.

---

## Git Conventions

- **Never commit directly to `main`** in any repo. All changes go through branches and PRs.
- **Never commit `docker-compose.yml` changes.** The `env_file: .env` lines and port remappings are local-only hacks required for Docker to start in worktrees. They must never be committed or pushed.
- **Never delete branches or worktrees** once created without explicit user instruction. If a deletion is made, record it below under "Deleted Branches/Worktrees".

### Deleted Branches / Worktrees

| Date | Repo | Branch | Worktree | Reason |
|------|------|--------|----------|--------|

---

## Code Review / PR Philosophy

- **Minimal changes per PR.** Only include changes that directly serve the feature or fix.
- **Address review comments minimally.** Make only the change needed to resolve the comment.
- **A PR is too big if** it mixes unrelated changes. Flag it and suggest a split.
- **Always run tests** after any change, even small ones.

### Comment Style

- **Comments explain _why_, not _what_.** Don't repeat what the code is doing — explain the reasoning, the constraint, or the non-obvious tradeoff.
- **Use `# Why:` as a tag** when the comment exists solely to explain a design decision or workaround.
- If the logic is self-evident, don't add a comment at all.

### Bug Severity Triage

- **Prioritise data-corrupting bugs** above all others — silent data mutation, duplicate records, incorrect aggregations. These are the hardest for users to detect.
- **Performance / N+1 concerns** should be assessed against actual data scale before flagging.
- **Edge case bugs** are low priority unless they touch data correctness.

### Testing Philosophy

- **Test core features first.** Happy path + most likely failure mode before edge cases.
- **Don't prioritise edge case tests over core coverage.** An untested happy path is a worse gap.
- **Tests should reflect real usage** — not hypothetical worst cases.

---

## Patterns to Follow

### Handlers
- Inherit from the telemetry base handler (adds OTel spans automatically).
- `auth_type` set appropriately for internal vs. external endpoints.
- `initialize()` pulls the async DB/storage client from `self.settings`.
- Use `self.get_json_body()` to parse request body; `self.write({...})` to respond.
- Run blocking I/O (pandas, S3) in `asyncio.get_event_loop().run_in_executor(None, fn, arg)`.

### Async DB/Storage Client
- One unified async HTTP client class for all backend data access.
- Key patterns: fetch by ID, fetch latest record by foreign key, resolve S3/storage paths from records.

### Telemetry
- Import the telemetry service at module level, not inside functions.
- Use span context propagation helpers when threading spans across executor or task boundaries.
- Use `.error` for handled failures; `.exception` only for unexpected exceptions (attaches stack trace, triggers alerts).

### Transformer pipeline
- Each transformer: `def name(df: pd.DataFrame, params: SomeModel) -> tuple[pd.DataFrame, Metadata]`
- Discovered dynamically via a transformer registry and `__all__` on each module.
- `ProcessTransformerResult.success=False` means handled failure; error message is in `.error_msg`.

### Chart Data Convention

Chart components receive **typed row arrays**, never raw columnar data.

- Columnar → row conversion happens **once** in the parent view, not inside each chart component.
- Field references use typed constants, not string literals.
- Row types are derived from the generated API client types.

Anti-patterns (do not introduce):
- `Record<string, Array<...>>` as a chart data prop — use typed row arrays
- String-typed field name props — fields are known at compile time
- Columnar-to-row conversion inside a chart component — convert once upstream

### API Boundary Types (Frontend)

When a generated client provides response types, derive component props from those types — don't re-declare `Record<string, unknown>` or untyped equivalents. Centralize any shared transformation in the parent and pass typed results down.

---

## Progress Tracking

**Local reference:** `docs/progress-status.md`

This file logs all shipped PRs, in-progress work, local design/planning artifacts, and investigations with timestamps. It is the single source of truth for "what did I accomplish this week."

**Proactive rules for Claude:**
- When finishing a task (PR merged, RFC written, manual test completed, JTBD drafted), offer to update the progress tracker.
- At natural stopping points (switching tickets, end of a session), nudge: "Want me to log this to the progress tracker?"
- If no commit or push has happened in a while during active coding, gently remind — uncommitted work is invisible work.
- When starting a new week, offer to add a new "Week of YYYY-MM-DD" section to the tracker.
