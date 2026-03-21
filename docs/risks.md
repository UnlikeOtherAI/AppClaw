# AppClaw - Risks & Mitigations

## Security & Credential Leakage

**Risk**: A mobile app handles powerful credentials (shared gateway token/password and/or per-device tokens), and the boundary is not per-user scoped.

**Mitigations**:
- Store secrets only in Keychain (iOS)
- Require biometric unlock for "operator mode"
- Prefer bootstrap tokens via setup code
- Implement rapid token rotation/revocation UX
- Never log secrets
- Treat WebView as privileged content, lock it to allowed origins
- Disable dangerous fallback settings (e.g., Host-header origin fallback)

## Pairing-Code & Onboarding Safety

**Risk**: Users may share setup codes via screenshots/chats. While bootstrap tokens are now short-lived, trust mistakes remain possible.

**Mitigations**:
- In-app copy about setup-code sensitivity
- Expiry countdown display
- Auto-clear clipboard after paste
- Encourage upgrading Gateway >= 2026.3.12
- Recommend rotating gateway credentials if earlier setup codes were shared

## Media/File Expectations Mismatch

**Risk**: WS chat attachments currently parse as images only. Non-image files are dropped after MIME sniffing. Images have a 5,000,000 decoded-byte default limit.

**Mitigations**:
- Enforce client-side validation and compression
- Show clear "images only" UI until document support exists
- For documents, fall back to `/v1/responses` `input_file` when enabled
- Clearly warn about operator-secret boundary

## Background Execution & Push Constraints

**Risk**: Reliable "always available" background behaviour is hard. iOS silent pushes can be throttled.

**Mitigations**:
- Make "Always-on" an explicit opt-in
- Use established patterns: APNs relay-backed approach on iOS distributed builds
- Degrade gracefully to pull-on-open
- Use BGTaskScheduler for opportunistic sync

## App Store Brand Confusion

**Risk**: Unofficial third-party "OpenClaw" apps exist in public stores. Users may install the wrong app, harming trust and increasing phishing risk.

**Mitigations**:
- Publish official package identifiers and signing info prominently in docs
- Add in-Gateway "Download mobile app" deep links that verify origin
- Implement in-app Gateway identity verification step before accepting pairing

## Rate Limits & Scalability

**Risk**: Aggressive mobile polling or misdesigned reconnect loops can trigger rate limits (e.g., config RPC throttles) and degrade the Gateway.

**Mitigations**:
- Prefer subscription-based updates (`sessions.messages.subscribe` / SSE follow) and bounded refresh
- Implement exponential backoff and jitter
- Cap list/history loads
- Respect server policy intervals

## Legal & Channel Policy Exposure

**Risk**: OpenClaw integrates with many third-party services with their own terms. A mobile client must avoid implying endorsement or bundling unauthorised access paths.

**Mitigations**:
- Position the app as a client for a self-hosted Gateway
- Avoid embedding third-party service credentials in the mobile client
- Route such configuration to the Gateway/Control UI
- Provide clear disclaimers and per-channel guidance via links or WebView pages
