---
name: data-contract
description: Define or validate the API contract for a feature: endpoint shape, request/response schemas, Pydantic models, database queries, TypeScript types. Run after RFC. MANDATORY TRIGGERS: /data-contract <name>, "define the API schema", "write the data contract", "what should the request/response look like". DO NOT trigger for: pre-RFC work, UI-only features with no new endpoints.
user_invocable: true
---

# data-contract (`/data-contract`)

Defines the request/response contract for a feature: endpoint URL, request/response models, database query signatures, and corresponding TypeScript types. Saves to `rfcs/<feature-name>-data-contract.md` or updates the Data Contract section of the RFC.

## Required inputs

- Feature name (must match an RFC in `rfcs/`)
- Optional: specific endpoint to focus on (e.g. `GET /api/v1/projects/{project_id}/summary`)

## Steps

### 1. Gather context (parallel)

- Read `rfcs/<feature-name>.md` — extract the goal, data flow, and any existing Data Contract section
- Read 1–2 similar existing endpoints + models to establish naming and field conventions
- Read relevant database schema/queries if data comes from a database layer

### 2. Define the contract

For each endpoint:

**Endpoint:**
```
METHOD /api/v1/<resource>
  ?param=type    # query params
Auth: session cookie / API key / JWT / none
```

**Request (POST/PUT only):**
```json
{
  "field_name": "type — description"
}
```

**Response:**
```json
{
  "field_name": "type — description"
}
```

**Python models (if using Pydantic or dataclasses):**
```python
class <Feature>Request(BaseModel):
    field_name: type

class <Feature>Response(BaseModel):
    field_name: type
```

**TypeScript types (frontend):**
```typescript
// Derive from the generated or declared API client types
// Do not redeclare manually — import from the shared type source
```

**Database queries (if applicable):**

| Query | Purpose | Returns |
|-------|---------|---------|
| `queryName` | … | … |

### 3. Validate against RFC (if contract already exists)

Compare the existing contract in the RFC against:
- What the handler actually accepts/returns (read handler code)
- What the frontend actually uses (read page/loader code)
- Flag any mismatches as "Drift: RFC says X, code does Y"

### 4. Save output

Append or update the Data Contract section in `rfcs/<feature-name>.md`. For large contracts, create `rfcs/<feature-name>-data-contract.md` and link from the RFC.

## Handling edge cases

- **Multiple endpoints for one feature**: document all of them, group by resource
- **No REST endpoint (database-only feature)**: document the database query/mutation signatures instead
- **Existing contract with drift**: produce a diff table (RFC spec vs actual) and recommend which is correct

## Quality check

- [ ] Every field has a type and a description
- [ ] Auth type is explicit
- [ ] Python/backend models match the JSON shape exactly
- [ ] TypeScript note directs to shared type source (not a manual re-declaration)
- [ ] Database queries table populated if a database layer is used
