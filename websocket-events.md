# TomChat WebSocket Events — v1

Endpoint: GET /api/v1/ws, upgraded to WebSocket.

Authenticate the HTTP upgrade with the normal bearer Authorization header.
Do not put bearer tokens in URL query parameters. The socket is bound to the
specific session used for the handshake. Logging out that session closes its
sockets; session expiry is checked on heartbeat. A user may have up to five
simultaneous sockets. The server sends ping frames every 25 seconds and closes
idle/unresponsive connections. Browser origins use the WebSocket upgrader's
same-host default. Compression is disabled.

## Envelope

Every application frame is UTF-8 JSON with these fields:
- version: 1
- event: event name
- request_id: client-generated UUID
- payload: event-specific JSON object

Version 1 and a non-empty request_id are required. Maximum frame size is 64 KiB.
Clients should correlate replies using request_id.

## Client event: message.send

The payload contains:
- conversation_id: conversation UUID
- client_message_id: stable UUID generated once per logical message
- ciphertext: base64-encoded ciphertext
- nonce: base64-encoded AEAD nonce
- encryption_version: 1
- key_id: optional sender key identifier

The server verifies conversation membership, payload size and encoding, then
stores the opaque ciphertext and nonce. It does not decrypt message contents.
The pair (sender_id, client_message_id) is unique, making retries idempotent.
A client must reuse the same client message ID when retrying a send.

## Server event: message.ack

Sent to the sender after successful persistence. Its payload contains the
server message_id and sequence. The event request_id matches the original
message.send request_id.

## Server event: message.deliver

Broadcast to currently connected members of the conversation. The payload is
the complete Message object defined in openapi.yaml, including the
server-assigned sequence and timestamp. The sender also receives this event
when connected.

## Server event: error

The payload contains a stable code. Possible codes include invalid_event,
unsupported_event, invalid_message, invalid_encrypted_message,
not_a_conversation_member, client_message_id_conflict, rate_limit_exceeded,
session_expired and internal_error.

## Delivery and recovery

WebSocket delivery is best-effort to online sockets. PostgreSQL is the source
of truth: after reconnecting, clients must fetch message history over the REST
endpoint using the last received sequence as the before cursor. The initial
server hub is process-local and intended for a single home-server instance;
multi-instance fan-out requires a shared pub/sub layer (for example PostgreSQL
LISTEN/NOTIFY or Redis) before horizontal scaling.

## Encryption boundary

The server treats ciphertext, nonce, encryption_version and key_id as opaque
transport metadata. This API is E2EE-ready, not a completed E2EE
implementation. Clients must not send plaintext in the ciphertext field.
A reviewed cryptographic protocol, identity/device keys, key distribution,
verification and recovery are separate requirements before claiming E2EE.
