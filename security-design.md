# TomChat Security Design — Initial Decisions

## Requirements
- Passwords: Argon2id, with parameters selected for supported server hardware.
- Transport: TLS for all remote connections.
- Private messaging: adopt a maintained Signal Protocol implementation; do not design a replacement.
- Group messaging: evaluate MLS (RFC 9420) through a maintained implementation such as OpenMLS.
- Voice/video: WebRTC transport security alone is not sufficient to claim end-to-end encryption when an SFU is used. Define and review an additional media E2EE design before making that claim.
- Device identity and recovery must be designed before multi-device E2EE is released.

## Threat-model review required
Document server compromise, malicious administrators, compromised endpoints,
lost devices, account recovery, group membership changes, key verification,
backup handling, and metadata exposure.

No E2EE feature should be described as production-ready until independently reviewed.
