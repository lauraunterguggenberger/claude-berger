---
name: ui-architecture
description: Design the frontend for a feature: route structure, component hierarchy, data fetching, real-time subscriptions, and state management. Run after RFC exists. MANDATORY TRIGGERS: /ui-architecture <name>, "design the frontend", "what routes/components do I need", "frontend architecture for". DO NOT trigger for: backend-only features, pre-RFC exploration.
user_invocable: true
---

# ui-architecture (`/ui-architecture`)

Produces a frontend architecture design: route structure, component hierarchy, data fetching strategy, real-time wiring, and state management decisions. Reads the RFC first.

## Required inputs

- Feature name (must match an RFC in `rfcs/`)
- Optional: worktree or directory name — if given, review existing implementation against RFC

## Steps

### 1. Gather context (parallel)

- Read `rfcs/<feature-name>.md` — extract UI requirements, screen states, data contract
- Read the data contract section or `rfcs/<feature-name>-data-contract.md` for API response types
- If a directory is given: read the route file and key component files to assess current state vs RFC

### 2. Map existing patterns

Find the closest existing route and component set to use as a pattern:
- Route/page file structure (data loading, actions, component export)
- How API data is fetched and typed (REST loader, GraphQL, real-time subscription)
- How data is passed to child components (typed props, not untyped records)
- How API client types flow into component props

### 3. Design the architecture

**Route/page structure:**
```
routes/ (or pages/)
  <feature>/
    index.tsx (or page.tsx)  ← data loading, page component
    components/
      <FeatureName>Chart.tsx
      <FeatureName>Table.tsx
      <FeatureName>Header.tsx
```

**Data flow:**
1. Route/page loader fetches from API (typed response from generated or declared client types)
2. Page component receives typed data
3. Any columnar → row transformation happens **once** in the page, not inside child components
4. Typed row/record arrays passed down to chart/table components

**State management:**
- What lives in URL params vs component state (prefer URL params for shareable/bookmarkable state)
- Real-time subscriptions (WebSocket, SSE, polling) vs one-time fetch
- Derived vs stored state

**Component props contract:**
- Derive props from API response types — avoid `Record<string, unknown>` or untyped equivalents
- Centralise any shared transformation in the parent, pass typed results down

### 4. Write test plan

| Test | Type | What to assert |
|------|------|----------------|
| Page renders with data | Integration | Key elements visible |
| Empty state | Unit | Empty state component shown |
| Loading state | Unit | Skeleton/spinner shown |
| Error state | Unit | Error message shown |

### 5. Output

Append or update the frontend architecture section in `rfcs/<feature-name>.md`, or produce a standalone design note.

## Handling edge cases

- **No RFC yet**: stop and prompt `/rfc <feature-name>` first
- **Existing implementation**: compare route and component files against RFC, report gaps and anti-patterns
- **Chart/table feature**: explicitly call out the columnar → row conversion location to prevent it drifting into child components
- **Shared component already exists**: reference it rather than proposing a duplicate

## Quality check

- [ ] Route path matches existing URL conventions in the codebase
- [ ] Any columnar → row conversion explicitly located in the page/parent (not in chart components)
- [ ] All component props typed (no `Record<string, unknown>`)
- [ ] State management decisions explained (why URL param vs useState)
- [ ] Test plan covers loading, error, and empty states
