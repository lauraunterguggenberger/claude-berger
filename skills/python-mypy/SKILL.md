---
name: python-mypy
description: Run mypy type checking on a Python backend and fix all errors. Mirrors the CI mypy check. MANDATORY TRIGGERS: /python-mypy, "run mypy", "check types", "fix type errors", "mypy is failing in CI". DO NOT trigger for: frontend TypeScript errors, non-Python repos, test-only files that CI skips.
user_invocable: true
---

# python-mypy (`/python-mypy`)

Runs `mypy` on the Python backend — identical to the CI check — and fixes all errors with minimal changes. Never adds `Any` to silence errors.

## Required inputs

- Optional: directory or worktree name — defaults to the current backend directory

## Steps

### 1. Identify the target directory

If a directory is given, use it. Otherwise infer from context (running processes, recent file edits, or ask).

The root is the directory containing `pyproject.toml` or `setup.cfg`.

### 2. Run mypy

```bash
# Via Docker (if project uses Docker — matches CI exactly)
docker exec <project>-worker-1 uv run mypy /app

# Or directly
uv run mypy .          # if using uv
mypy .                 # if using system/venv mypy
```

Run from the project root.

### 3. Parse and triage errors

Group errors by file. For each error:

| Error pattern | Typical cause | Fix |
|---------------|--------------|-----|
| `No overload variant of "X" matches` | Missing `@overload` stub param | Add correct overload |
| `Argument 1 has incompatible type "X"; expected "Y"` | Wrong type at call site | Fix caller or update signature |
| `Item "None" of "X \| None" has no attribute` | Missing null guard | Add `if x is not None:` guard |
| `Module has no attribute "X"` | Wrong import path | Fix import |
| `Missing return statement` | Branch missing `return` | Add return |
| `Incompatible return value type` | Return type annotation wrong | Fix annotation or return value |

### 4. Fix errors

Fix rules:
- Fix the type properly — never use `cast(Any, ...)` or `# type: ignore` to silence
- Minimal change: only what mypy requires, no surrounding cleanup
- If the fix requires changing a function signature used in many places, fix the signature and all call sites

### 5. Re-run to confirm clean

After fixing, run mypy again to confirm zero errors before reporting done.

### 6. Report

```
mypy: ✅ clean (N errors fixed in M files)

Files changed:
- path/to/file.py — [description of fix]
```

## Handling edge cases

- **Error in a file you didn't touch**: fix it anyway — CI will fail regardless of who introduced it
- **Third-party stub missing**: add the stub package via `uv add <types-package> --dev` (or `pip install`), not a `# type: ignore`
- **Circular import introduced by fix**: restructure the import (use `TYPE_CHECKING` guard if needed)

## Quality check

- [ ] Final mypy run shows zero errors
- [ ] No `Any`, `cast`, or `# type: ignore` added
- [ ] Only minimal changes made (no opportunistic refactoring)
