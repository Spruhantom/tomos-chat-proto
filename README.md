# TomChat Protocol

Shared API and realtime event contract for TomChat clients and servers.

## Version 0.3.0

The OpenAPI specification defines registration, bearer authentication, current-user lookup, logout, direct/group conversations, paginated encrypted-message history, message submission and the authenticated WebSocket endpoint. Passwords and message ciphertext must be transported over TLS in remote deployments.

## Security rules

- HTTP endpoints are versioned under `/api/v1`.
- WebSocket events use explicit names and schema versions.
- Message payloads must be opaque ciphertext at the server boundary once E2EE is implemented.
- Never invent cryptographic primitives.
- Breaking changes require a documented protocol version transition.

The server currently persists opaque encrypted payloads and relays live message events. Client-side cryptography, device registration and key distribution, member management, shared multi-instance pub/sub, and call signalling remain future work.
