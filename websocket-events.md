# TomChat WebSocket Events — Draft

Events use a JSON envelope:

```json
{
  "version": 1,
  "event": "message.send",
  "request_id": "client-generated-id",
  "payload": {}
}
```

Planned events:
- `message.send`
- `message.deliver`
- `message.ack`
- `presence.update`
- `typing.update`
- `group.member_added`
- `group.member_removed`
- `call.invite`
- `call.signal`

For end-to-end encrypted conversations, message bodies and sensitive group state
must remain ciphertext at the server boundary. This document is not a cryptographic
protocol specification.
