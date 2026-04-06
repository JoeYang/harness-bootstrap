## Security

- No hardcoded secrets -- read from env vars via import.meta.env
- Validate all input at system boundaries (form data, API responses, URL params)
- No XSS: sanitize user content before rendering; avoid dangerouslySetInnerHTML
- No CSRF: use anti-CSRF tokens on state-changing requests
- Auth checks on every protected route (authn + authz)
- Never log passwords, tokens, or PII to the console
- Use HttpOnly cookies for session tokens -- never store in localStorage
- Content Security Policy headers on all responses
- No dependencies with known CVEs; flag unmaintained packages for review
