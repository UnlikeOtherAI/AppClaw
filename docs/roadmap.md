# AppClaw - Roadmap

## Phase 1: iOS Only

Phase 1 focuses exclusively on iOS (iPhone + iPad) with native SwiftUI.

## Milestones

### Milestone A: Foundation

- Gateway profiles + secure credential storage (Keychain)
- QR/setup-code bootstrap (camera scanner + base64 decode)
- WS connection manager with `connect.challenge`, device signing, and `hello-ok` parsing
- Device identity generation and pairing workflow
- WebView host for Control UI (safety net fallback for features not yet native)

### Milestone B: Chat MVP

- Native session picker with list/search/preview (`sessions.list`, `sessions.preview`)
- Native chat renderer with streaming events
- `chat.send` with idempotency + `chat.abort` ("Stop generating")
- Image attachments (image-only, 5MB decoded cap, client-side validation + compression)
- Local transcript cache (SwiftData/SQLite) for offline read

### Milestone C: Live Sync & Settings

- `sessions.messages.subscribe` for real-time transcript updates without reloading
- `sessions.preview` for efficient session list rendering
- `sessions.patch` for per-session settings (fast mode, reasoning level, thinking level, usage display)
- Reduce reliance on slash-command parsing and full-page reloads

### Milestone D: Voice & Notifications

- Voice capture UI + talk mode UX
- iOS push relay integration (App Attest + receipt, `push.apns.register`)
- BGTaskScheduler for periodic refresh
- Silent push as best-effort reconnect wake

## Illustrative Timeline

```
Milestone A (Foundation)
  Gateway profiles + secure storage .............. ~2 weeks
  WS connect + device signing + pairing UX ....... ~2.5 weeks
  WebView host (Control UI fallback) ............. ~1.5 weeks

Milestone B (Chat MVP)
  Sessions list/search + previews ................ ~2 weeks
  Native chat UI + streaming + abort ............. ~2.5 weeks
  Image attachments (5MB decoded cap) ............ ~1.5 weeks
  Local transcript cache + offline read .......... ~1.5 weeks

Milestone C (Live Sync & Settings)
  sessions.messages.subscribe + incremental UI ... ~1.5 weeks
  Session settings (fast/reasoning/usage) ........ ~1 week

Milestone D (Voice & Push)
  Voice input + talk mode UI ..................... ~2.5 weeks
  iOS push relay integration ..................... ~2 weeks
```

## Testing Strategy

### Contract Tests

Use TypeBox-derived schemas as the contract: generate fixtures for `connect`, `chat.send`, `sessions.patch` and verify encode/decode invariants for iOS models.

### Integration Tests

Stand up a local Gateway (loopback) and run scripted sessions: security handshake, pairing, send/stream/abort, subscribe/unsubscribe, patch settings, and attachment upload.

### Adversarial Tests

- Token drift and `AUTH_TOKEN_MISMATCH` retry behaviour (bounded retry, then require user action)
- Oversized attachment rejection and non-image attachment drop behaviour (ensure UI makes this visible)
- WebView origin enforcement and navigation denial (security regression suite)

### Visual Verification

Every feature must be clicked through and visually verified using AppReveal (debug-only) against a real OpenClaw instance before it's considered done.
