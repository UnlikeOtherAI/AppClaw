# UpClaw - Project Brief

## Mission

Build free, open-source, native iOS and Android clients for OpenClaw. Every existing mobile client is paid — UpClaw is a community-driven alternative.

## What is OpenClaw?

OpenClaw is an MIT-licensed, self-hosted "Gateway" that sits between your existing chat surfaces (messaging apps, web UI, CLI) and one or more agent runtimes. It owns session state, routing, delivery, pairing, and integrations.

The core control-plane transport is a typed WebSocket (default port 18789) with a mandatory `connect` handshake, role declaration (`operator` vs `node`), scopes, and device identity + signature (challenge/nonce). After first approval ("device pairing"), the Gateway issues a per-device token (`hello-ok.auth.deviceToken`) that clients persist for subsequent connects, with explicit retry guidance and stable auth/detail error codes.

In practice, "chat is the UI": users interact from the channels they already use, while the Gateway manages multi-agent routing, tools/plugins, media handling, and automation primitives like cron/heartbeat. OpenClaw also exposes selected HTTP surfaces on the same multiplexed port, notably OpenAI-compatible `POST /v1/chat/completions` and OpenResponses-compatible `POST /v1/responses` (disabled by default), both supporting SSE streaming. A small authenticated webhook surface (e.g., `/hooks/wake`, `/hooks/agent`) exists when enabled.

## How OpenClaw Works

OpenClaw's architecture is intentionally "single source of truth": one long-lived Gateway process owns sessions, routing, and channel connections. Clients (web UI, CLI, companion apps) connect to the Gateway over the same WebSocket control plane. "Nodes" (mobile/desktop) connect as peripherals with a declared capability surface invoked via `node.invoke`.

### Gateway WebSocket Protocol

- **Transport & frames**: WebSocket text frames carrying JSON. The first frame must be a `connect` request. Requests/responses follow `{type:"req", id, method, params}` -> `{type:"res", id, ok, payload|error}`. Server push uses `{type:"event", event, payload, seq?, stateVersion?}`.
- **Handshake hardening**: Before `connect`, the server emits `event:"connect.challenge"` with `{ nonce, ts }`. Clients must sign the challenge-bound payload and return matching `connect.params.device.nonce`. Device-auth migration codes (e.g., `DEVICE_AUTH_NONCE_REQUIRED`, `DEVICE_AUTH_SIGNATURE_INVALID`) are explicitly documented for client implementers.
- **Role separation**:
  - `operator`: control plane clients needing scopes like `operator.read`, `operator.write`, `operator.admin`, etc.
  - `node`: capability hosts (camera/screen/canvas/location/voice/device control) declaring `caps`, `commands`, and `permissions` in the `connect` payload.

### Session State

Session state is conceptually "keyed chat threads": a main bucket, group buckets, cron buckets, hook buckets, and node buckets, with key formats explicitly described (e.g., `agent:<agentId>:<channel>:group:<id>`). "Chat" in the Control UI or mobile app may represent multiple distinct session keys across channels and agents, and the client must be able to list, resolve, subscribe, and patch those sessions efficiently.

## Strategy

For a modern native mobile experience, a hybrid strategy is the fastest path: implement **native chat** plus **5-10 high-value functions/settings** (see [mobile-features.md](mobile-features.md)), and host everything else in a WebView that loads the Gateway-served Control UI. To make migration from web to native incremental and low-risk, architect the app as a **feature-router**: each capability is addressable by a stable route, implemented either by a native module or a "web module" wrapper, switchable via feature flags and version gating.

## Key Technical Constraints

- **Credential model is powerful, not least-privilege**: Gateway HTTP bearer auth is effectively "operator access" for that Gateway instance. Treat any token/password that can access `/v1/*`, `/tools/invoke`, or `/api/channels/*` as a full-access secret.
- **Device pairing UX is central**: Setup codes now carry a short-lived `bootstrapToken` (not long-lived gateway credentials), following a security advisory that affected versions <= 2026.3.11 and was patched in 2026.3.12.
- **Chat history and session lists are performance-sensitive**: The project is actively adding richer dashboard chat tooling (search/export/pins) and has multiple issues around large histories/UI responsiveness — strong evidence that efficient paging/subscription and local caching matter for mobile.

## Phase 1 Scope

- **Platform**: iOS only (iPhone + iPad)
- **Framework**: Native SwiftUI
- **Architecture**: Clean Architecture
- **Debug tooling**: AppReveal (debug-only, contains Apple private APIs)
- **Testing**: Against a real OpenClaw instance

## Related Docs

- [API Reference](api.md) - Protocols, endpoints, and authentication
- [Architecture](architecture.md) - App structure, modules, and data flow
- [Mobile Features](mobile-features.md) - Priority native features
- [Roadmap](roadmap.md) - Implementation milestones and timeline
- [Risks](risks.md) - Risk assessment and mitigations
- [Sources](sources.md) - Official repositories and documentation references
