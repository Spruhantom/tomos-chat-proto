# Device identity and MLS delivery protocol proposal

Status: proposed v1 design only. These endpoints are not implemented by the server and must not be treated as available API.

## Design constraints

- MLS follows RFC 9420; the server is an untrusted delivery service and never receives private keys or plaintext.
- Use OpenMLS behind a client-only adapter. The server treats credentials, KeyPackages, Welcome messages, Commit messages and Application messages as opaque byte strings.
- Device identity is distinct from account identity. Each installation has its own device ID and MLS credential.
- All endpoints require TLS and bearer authentication. Never place session tokens or key material in URLs.
- Directory reads are limited to the authenticated user and users sharing a conversation. Do not expose a global device directory.
- All mutations are idempotent where possible and enforce strict size/count limits.

## Proposed REST surface

### Device registration

`POST /api/v1/devices`

Register a new installation. Request fields:
- `device_id`: client-generated UUID.
- `credential`: base64-encoded MLS credential bytes.
- `credential_signature`: base64 signature proving possession of the credential signing key over the canonical registration transcript.
- `display_name`: optional user-visible device label, length-limited.
- `protocol_version`: supported MLS protocol version.

The server binds the record to the authenticated account and records it as `pending` unless it is the account's first device. First-device enrolment is trust-on-first-use and must display a safety code to the user. Subsequent devices require an approval proof signed by an existing verified device; the exact canonical transcript and signature scheme must be specified in the implementation contract before coding.

### Device directory

`GET /api/v1/devices` returns the authenticated user's devices and their status.

`GET /api/v1/conversations/{conversationID}/devices` returns only devices belonging to current conversation members, including credential fingerprints and verification status. It must not reveal unrelated accounts' devices.

`POST /api/v1/devices/{deviceID}/approve` accepts an approval statement signed by an existing verified device. Approval must bind account ID, new device ID, credential fingerprint, protocol version, expiry and a unique nonce.

`DELETE /api/v1/devices/{deviceID}` revokes a device. Revocation is irreversible for that device ID; a reinstall creates a new ID and credential. The API records the event but clients must also remove the device from each MLS group.

### KeyPackage publication and retrieval

`POST /api/v1/devices/{deviceID}/key-packages` uploads a bounded batch of signed, single-use MLS KeyPackages. The server stores them as opaque bytes with a unique package ID and expiry.

`POST /api/v1/conversations/{conversationID}/key-package-claims` claims one KeyPackage for a target device that belongs to a conversation member. A claim is atomic and single-use; a package cannot be claimed twice. Return the opaque KeyPackage, package ID and credential fingerprint. Expired or exhausted inventories return a stable error so clients can replenish.

Do not accept client-supplied account/device ownership as authoritative; derive it from the authenticated session and stored directory records.

### MLS message relay

`POST /api/v1/conversations/{conversationID}/mls-messages` stores and fans out one opaque MLS message. Fields:
- `client_message_id`: UUID, stable across retries.
- `message_type`: `application`, `proposal`, `commit` or `welcome`.
- `epoch`: non-negative integer supplied for routing/diagnostics only; the server cannot authenticate it.
- `content`: base64-encoded MLS message bytes.
- `protocol_version`: explicit supported version.

The authenticated device must be an active member device for the conversation. The server must enforce size limits, idempotency and rate limits, but must not parse/decrypt MLS content or make membership decisions based on untrusted fields. Clients authenticate all MLS transitions cryptographically.

`GET /api/v1/conversations/{conversationID}/mls-messages?after={sequence}&limit={limit}` returns opaque messages oldest first with a forward cursor. The server assigns its own monotonic sequence; never use the client-supplied epoch as a history cursor.

## Event contract additions

Add WebSocket event `mls.message` with the same message ID, server sequence, sender account/device IDs, message type, protocol version, opaque content and server timestamp. The sender receives a durable acknowledgement only after database commit. Reconnect uses REST history as the source of truth.

Device events: `device.pending`, `device.approved`, `device.revoked`, and `key_package.low`. These events are notifications only; clients must fetch and validate authoritative state over REST.

## Error codes

Use stable codes: `device_not_found`, `device_pending`, `device_revoked`, `device_approval_required`, `invalid_device_proof`, `key_package_unavailable`, `key_package_already_claimed`, `unsupported_mls_version`, `mls_message_too_large`, `mls_message_id_conflict`, `not_a_conversation_device`.

## Security and consistency requirements

- Use database transactions for device registration, approval, revocation, KeyPackage claim and message persistence.
- A successful KeyPackage claim must atomically mark it consumed.
- Revocation must prevent future submissions from that device immediately; MLS group removal is a separate client-generated commit and cannot retract past plaintext.
- Never treat a directory response as proof of identity. Clients pin/verify credential fingerprints and display changes.
- Avoid a server API that returns private key material, account recovery secrets or plaintext.
- Define canonical encodings, signature transcripts, nonce lengths, expiry semantics, limits and pagination before implementation.
- Do not enable legacy `/messages` plaintext submission or the UI composer as an alternative to MLS. Keep the existing encrypted-message route disabled at the client until MLS is integrated.
