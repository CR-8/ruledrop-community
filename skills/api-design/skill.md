Design clean, consistent REST APIs. Use this skill when designing new API endpoints, reviewing an existing API, writing OpenAPI specs, or deciding on URL structure, request/response shapes, and error handling conventions.

## URL Design

**Use nouns, not verbs.** The HTTP method is the verb.

```
GET    /users          → list users
POST   /users          → create a user
GET    /users/:id      → get one user
PATCH  /users/:id      → update a user
DELETE /users/:id      → delete a user
```

**Nest resources only one level deep.** Deeper nesting becomes unreadable.

```
✓  GET /posts/:id/comments
✗  GET /users/:id/posts/:postId/comments/:commentId
```

**Use kebab-case for multi-word paths.** Never camelCase or snake_case in URLs.

```
✓  /api/user-profiles
✗  /api/userProfiles
✗  /api/user_profiles
```

**Version your API.** Put the version in the path prefix.

```
/api/v1/users
/api/v2/users
```

## HTTP Methods

| Method | Use | Body | Idempotent |
|--------|-----|------|------------|
| GET | Read | No | Yes |
| POST | Create | Yes | No |
| PUT | Full replace | Yes | Yes |
| PATCH | Partial update | Yes | No |
| DELETE | Remove | No | Yes |

Use `PATCH` for partial updates (most updates). Use `PUT` only when replacing the entire resource.

## Request & Response Shapes

**Requests** — accept `application/json`. Validate all fields server-side regardless of client validation.

**Responses** — always return JSON with a consistent envelope:

```json
// Success (single resource)
{ "data": { "id": "123", "name": "Alice" } }

// Success (collection)
{
  "data": [...],
  "meta": { "total": 100, "page": 1, "perPage": 20 }
}

// Error
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Email is required",
    "details": [{ "field": "email", "message": "Required" }]
  }
}
```

Never return different shapes for the same endpoint. Consistency is more important than brevity.

## Status Codes

Use the right code — don't return 200 for everything.

| Code | When |
|------|------|
| 200 | Success (GET, PATCH, DELETE) |
| 201 | Resource created (POST) |
| 204 | Success, no body (DELETE) |
| 400 | Bad request — invalid input |
| 401 | Not authenticated |
| 403 | Authenticated but not authorized |
| 404 | Resource not found |
| 409 | Conflict (duplicate, state mismatch) |
| 422 | Validation failed |
| 429 | Rate limited |
| 500 | Server error |

## Pagination

Use cursor-based pagination for large or frequently-updated datasets. Use offset pagination only for small, stable datasets.

```json
// Cursor-based
{
  "data": [...],
  "meta": {
    "nextCursor": "eyJpZCI6MTAwfQ==",
    "hasMore": true
  }
}

// Offset-based
{
  "data": [...],
  "meta": { "total": 500, "page": 3, "perPage": 20 }
}
```

## Filtering, Sorting, Searching

Use query parameters:

```
GET /posts?status=published&authorId=42
GET /posts?sort=createdAt&order=desc
GET /posts?q=search+term
```

## Error Handling

Every error response must include:
- A machine-readable `code` (screaming snake case: `NOT_FOUND`, `RATE_LIMITED`)
- A human-readable `message`
- Optional `details` array for field-level validation errors

Never expose stack traces, internal paths, or database errors to clients.

## Authentication

- Use `Authorization: Bearer <token>` header
- Prefer short-lived JWTs (15–60 min) with refresh tokens
- Return `401` for missing/invalid token, `403` for valid token but insufficient permissions

## Naming Conventions

- Field names: `camelCase` in JSON
- Timestamps: ISO 8601 (`2024-01-15T10:30:00.000Z`)
- IDs: strings (not integers) — easier to migrate to UUIDs later
- Booleans: `isActive`, `hasAccess` — not `active`, `access`

## OpenAPI / Documentation

Every endpoint should document:
- Path, method, description
- Path/query/body parameters with types and whether required
- All possible response codes and their shapes
- At least one request/response example
