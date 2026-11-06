# TomChat Protocol

Shared API and realtime event contract for TomChat clients and servers.

## Version 0.2.0

The OpenAPI specification defines registration policy, account registration,
login, current-user lookup, and logout. Authentication uses short-lived bearer
tokens. Passwords are transmitted only over TLS in remote deployments.

## Security rules

- HTTP endpoints are versioned under `/api/v1`.
- WebSocket events use explicit names and schema versions.
- Message payloads must be opaque ciphertext at the server boundary once E2EE is implemented.
- Never invent cryptographic primitives.
- Breaking changes require a documented protocol version transition.

Messaging, device registration, encrypted payload schemas, group membership,
and call signalling remain future work.
