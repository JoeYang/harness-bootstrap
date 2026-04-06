## Security (Strict)

- No hardcoded secrets -- read from env vars or secrets manager
- Validate all input at system boundaries (network packets, FIX messages, config files)
- No buffer overflows, format string bugs, or use-after-free
- Least privilege for network sockets, file handles, and IPC channels
- Auth checks on every admin and control plane endpoint
- Never log passwords, tokens, API keys, or order details at INFO level
- Crypto: standard libs only (OpenSSL/BoringSSL), no custom crypto
- Audit log all auth events, privilege changes, and configuration modifications
- No deps with known CVEs; flag unmaintained packages for review
