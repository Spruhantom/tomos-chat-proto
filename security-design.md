# TomChat Security Design

## Implemented in server stage 2

- Passwords are hashed with Argon2id (3 iterations, 64 MiB memory, parallelism 2, 32-byte output).
- Login issues a cryptographically random 256-bit bearer token.
- Only the SHA-256 hash of the bearer token is stored in PostgreSQL.
- Sessions expire after 24 hours and can be revoked by logout.
- Account registration can be disabled by the server operator.
- Login has a basic per-process limit of 10 attempts per source IP per minute; this is not a substitute for distributed edge rate limiting.
- First administrator bootstrap is allowed only while the user table is empty.
- Remote desktop connections require HTTPS; HTTP is permitted only for local development.

## Requirements before public deployment

- Extend rate limiting to registration and authenticated abuse controls; deploy distributed edge rate limiting for multi-instance/proxy deployments.
- Add account recovery and administrator recovery procedures.
- Add audit logging and session/device management.
- Configure TLS, backups, monitoring, and secret rotation.
- Complete threat modelling and an independent security review.

## End-to-end encryption plan

- Private messaging: adopt a maintained Signal Protocol implementation; do not design a replacement.
- Group messaging: evaluate MLS (RFC 9420) through a maintained implementation such as OpenMLS.
- Device identity and recovery must be designed before multi-device E2EE is released.
- The server must route and store ciphertext without receiving plaintext message keys.

## Voice and video

WebRTC transport encryption alone is not sufficient to claim end-to-end encryption when an SFU is used. Define and review an additional media E2EE design (for example, an SFrame-based approach) before making that claim.

## Threat model to review

Server compromise, malicious administrators, compromised endpoints, lost devices,
account recovery, group membership changes, key verification, backup handling,
and metadata exposure.
