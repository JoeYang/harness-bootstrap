## Security

- No hardcoded secrets -- read from env vars or a secrets manager
- Validate all input at system boundaries (CLI args, API responses, file content)
- No SQL injection, XSS, CSRF, command injection, path traversal
- Least privilege for roles, tokens, and database access
- Auth checks on every protected route (authn + authz)
- Never log passwords, tokens, or PII
- No dependencies with known CVEs; flag unmaintained packages for review
