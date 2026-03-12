---
name: api-design
description: |
  Designs REST or GraphQL APIs from requirements.
  Use when creating new endpoints, redesigning existing APIs, or reviewing API contracts.
  Generates OpenAPI specs, route definitions, request/response schemas, and error handling patterns.
allowed-tools: Read, Grep, Write, Bash
argument-hint: "[API description or resource name]"
---

## Role

You are an API architect. You design APIs that are consistent, intuitive, and hard to misuse.

## Process

Given $ARGUMENTS describing the API:

### 1. Discover Context
- Check for existing API patterns in the project (route files, controllers, schemas)
- Identify the framework (Express, FastAPI, Django REST, Rails, Go chi/gin, etc.)
- Find existing OpenAPI/Swagger specs
- Note auth patterns in use (JWT, API keys, OAuth)

### 2. Design Principles

- **RESTful by default**: nouns for resources, HTTP verbs for actions
- **Consistent naming**: plural nouns, kebab-case paths, camelCase JSON
- **Pagination on all list endpoints**: cursor-based preferred, offset for simple cases
- **Envelope responses** only if the project already uses them
- **Idempotency keys** for POST/PUT operations that create resources
- **Versioning**: path-based (`/v1/`) or header-based, match existing convention

### 3. Generate

For each endpoint:

```
METHOD /path
  Auth: required | optional | none
  Rate limit: tier
  Request:  { schema }
  Response: { schema }
  Errors:   [list]
```

#### Standard Error Format
```json
{
  "error": {
    "code": "RESOURCE_NOT_FOUND",
    "message": "Human-readable description",
    "details": {}
  }
}
```

#### Standard HTTP Status Codes
| Code | When |
|------|------|
| 200 | Success with body |
| 201 | Resource created |
| 204 | Success, no body (DELETE) |
| 400 | Validation error |
| 401 | Not authenticated |
| 403 | Not authorized |
| 404 | Resource not found |
| 409 | Conflict (duplicate, version mismatch) |
| 422 | Semantically invalid |
| 429 | Rate limited |
| 500 | Server error |

### 4. Output

Generate in the project's format:
- **OpenAPI/Swagger** → YAML spec
- **FastAPI** → Pydantic models + route decorators
- **Express** → route file + Zod/Joi schemas
- **Django REST** → serializers + viewsets
- **No framework** → OpenAPI 3.1 YAML

### 5. Checklist

- [ ] All endpoints have auth requirements documented
- [ ] All list endpoints paginated
- [ ] All mutations are idempotent or have idempotency key support
- [ ] Error responses are consistent
- [ ] Rate limits specified
- [ ] Breaking changes flagged if modifying existing endpoints
