## Security

- No hardcoded secrets -- read from env vars or a secrets manager
- Validate all input at system boundaries (REST requests, Kafka messages, config)
- No SQL injection: use parameterized queries exclusively; never concatenate SQL
- No insecure deserialization: whitelist allowed types for JSON/XML parsing
- Auth checks on every protected endpoint (authn + authz via Spring Security)
- Least privilege for database roles, service accounts, and API scopes
- Never log passwords, tokens, PII, or full request bodies at INFO level
- Crypto: standard libs only (javax.crypto, BouncyCastle); no custom crypto
- No dependencies with known CVEs; flag unmaintained packages for review
