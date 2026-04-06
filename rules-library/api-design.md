---
paths: ["src/api/**", "src/routes/**", "src/handlers/**", "src/controllers/**", "**/*.proto"]
---
# API Design Rules

## REST Conventions

- RESTful resource naming: `GET /v1/users/:id`, `POST /v1/users`, `DELETE /v1/users/:id`
- Version all API paths (`/v1/...`) — never expose unversioned endpoints
- Consistent error response format across all endpoints:
  ```json
  {"error": "VALIDATION_FAILED", "message": "Email is required", "request_id": "..."}
  ```
- Include `X-Request-ID` in all responses for distributed tracing
- Validate all input at the boundary — use zod (TS), pydantic (Python), or protobuf (gRPC)
- Document every REST endpoint with OpenAPI/Swagger — keep spec in sync with code
- Rate limiting: be aware of upstream limits and document your own

## gRPC / Protobuf Conventions

- One service per `.proto` file; service name matches the file name
- Use `PascalCase` for service and message names, `snake_case` for fields
- Return canonical gRPC error codes (`NOT_FOUND`, `INVALID_ARGUMENT`, `PERMISSION_DENIED`)
- Proto field numbers are permanent — never reuse or renumber after deployment
- Reserve removed field numbers to prevent accidental reuse

## General

- Never expose internal IDs, stack traces, or system details in error responses
- Idempotency: `PUT` and `DELETE` must be idempotent; use idempotency keys for `POST` where needed
- Pagination: use cursor-based pagination for large collections, not offset-based
- Auth checks on every protected route — verify both identity and permission
