# TomChat Protocol

Shared API and realtime event contract for TomChat clients and servers.

## Protocol rules

- HTTP endpoints are versioned under `/api/v1`.
- WebSocket events use explicit names and schema versions.
- Message payloads are opaque ciphertext at the server boundary.
- Never invent cryptographic primitives; use reviewed, maintained protocol implementations.
- Breaking changes require a documented protocol version transition.

## Current status

Initial draft only. Authentication, device identity, encrypted messaging, group membership, and call signalling schemas remain to be specified and reviewed.
