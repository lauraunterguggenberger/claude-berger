---
name: backend-architecture
description: Design the Python backend for a feature: handler/endpoint structure, async tasks, database integration, object storage patterns, and test plan. Run after RFC and data contract exist. MANDATORY TRIGGERS: /backend-architecture <name>, "design the backend", "what handlers do I need", "backend architecture for". DO NOT trigger for: frontend-only features, pre-RFC exploration.
user_invocable: true
---

# backend-architecture (`/backend-architecture`)

Produces a backend architecture design: endpoint structure, async task definitions, database integration points, object storage patterns, and test plan. Reads the RFC first — the RFC is the source of truth for scope.

## Required inputs

- Feature name (must match an RFC in `rfcs/`)
- Optional: worktree or directory name — if given, review existing implementation against RFC

## Steps

### 1. Gather context (parallel)

- Read `rfcs/<feature-name>.md` — extract goal, data flow, data contract section
- Read `rfcs/<feature-name>-data-contract.md` if it exists
- If a directory is given: read existing handler and task files to assess current state vs RFC

### 2. Map existing patterns

Find the closest existing endpoint and task to use as a pattern:
- Read 1–2 similar handlers/endpoints for: auth pattern, request parsing, async I/O handling
- Read 1–2 similar async tasks for: task signature, database client usage, object storage patterns
- Note the base class or framework conventions in use (FastAPI, Flask, Django, Tornado, etc.)

### 3. Design the architecture

Produce:

**Endpoint design:**
```
GET/POST /api/v1/<resource>
  Auth: session / API key / JWT
  Handler: <ClassName>
  Request parsing: <framework pattern>
  Response: <shape>
  Async I/O: wrap blocking calls in executor / use async client
```

**Async task design (if background work needed):**
```
@task
async def <task_name>(resource_id: str, ...) -> <ResultType>:
  client = get_db_client()
  # fetch data
  # run computation (executor if CPU-bound)
  # write result back
```

**Database integration:**
- Which queries/mutations to call
- Read vs write operations
- Connection/client management pattern

**Object storage (if file I/O needed):**
- Read from / write to storage paths
- Wrap blocking storage calls in async executor

### 4. Write test plan

| Test | Type | What to assert |
|------|------|----------------|
| Happy path | Integration | Correct response shape, status 200 |
| Missing required field | Unit | 400 with clear error message |
| Async task completes | Integration | Result persisted correctly |
| Auth rejected | Integration | 401/403 |

### 5. Output

Append or update the architecture section in `rfcs/<feature-name>.md`, or produce a standalone design note.

## Handling edge cases

- **No RFC yet**: stop and prompt `/rfc <feature-name>` first
- **Existing implementation**: compare file-by-file against RFC, report gaps
- **Async task not needed**: omit that section and explain why (synchronous response is sufficient)

## Quality check

- [ ] Endpoint follows existing URL/naming conventions in the codebase
- [ ] Auth type explicitly stated
- [ ] All blocking I/O identified and handled asynchronously
- [ ] Test plan covers happy path + at least one failure mode
- [ ] Architecture consistent with RFC data contract
