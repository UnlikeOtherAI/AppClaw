# AppClaw - Architecture

## Design Principles

- Treat the Gateway as the **source of truth** for sessions and routing
- The phone provides: (a) a fast chat renderer, (b) session navigation & settings, (c) device-integrated IO (camera, mic, notifications) with minimal setup friction
- Keep discovery/advertising in the Gateway; clients are consumers — aligning with a thin-client mobile approach

## Feature Router Pattern

Use a **Feature Router** abstraction for incremental web-to-native migration:

- Each app "screen" is addressed by a stable route: `chat/<sessionKey>`, `sessions`, `sessionSettings/<key>`, `connect`, `web/<path>`, etc.
- For each route, choose an implementation:
  - **Native module** (SwiftUI) for prioritised features
  - **Web module** (WebView wrapper) that deep-links into the Gateway Control UI for everything else
- A local feature-flag service decides at runtime which implementation is active (based on Gateway version, client app version, or user toggles)
- Migrating a feature from web to native is primarily a routing/config change; the WebView route remains as a fallback

## Gateway Client Core

Build a shared protocol layer:

### Protocol Models

Auto-generate strongly typed models from OpenClaw's TypeBox/JSON-Schema source of truth. iOS: Swift package with generated models + WebSocket client.

### Connection Engine

- Implements `connect.challenge` wait, device signing, `connect` request, and `hello-ok` parsing
- Manages reconnection policy, one bounded retry for `AUTH_TOKEN_MISMATCH` using cached `deviceToken`
- Surfaces "recommendedNextStep" guidance to UI

### RPC Layer

- Request ID generation
- Idempotency key generation for side-effecting calls like `chat.send`

### Event Bus

- Typed dispatch for `chat` and `session.message`-style events
- Handshake -> connected lifecycle management

## iOS App Architecture (Phase 1)

### Presentation Layer

SwiftUI + unidirectional data flow store (reducer-based), keyed around:

- `ConnectionState` — discovered endpoints, selected gateway profile, auth mode, pairing status
- `SessionListState` — rows, search query, paging cursor (if using HTTP history)
- `ChatState` per session — local DB messages + "in-flight run" markers keyed by `runId`

### Data Layer

- `GatewayWebSocketClient` (core)
- Optional `GatewayHttpClient` for `/v1/*` and `/sessions/{key}/history` SSE follow, especially for efficient paging/streaming on poor networks

### Storage

- **Keychain**: gateway token/password/bootstrapToken/deviceToken
- **Local DB** (SwiftData or SQLite): transcript caching and offline read

### Background & Push

- Follow OpenClaw's relay-backed push design: iOS app registers with a relay using App Attest + receipt, forwards an opaque handle via `push.apns.register`; Gateway can send reconnect wakes and wake nudges without storing raw APNs tokens
- Use BGTaskScheduler for periodic refresh; treat silent pushes as best-effort due to platform throttling

## WebView Hosting & Bridging

The Control UI is served by the Gateway (Vite + Lit SPA) and speaks WebSocket directly:

- **Navigation constraints**: Only allow the WebView to load content from the configured Gateway origin(s); deny external navigations or open them in the system browser
- **Credential handling**: Prefer not to persist long-lived tokens in WebView storage. The Control UI stores tokens in sessionStorage. If you need single sign-on into the WebView, inject a short-lived bootstrap flow (setup-code) rather than a long-lived shared token
- **Incremental migration**: Implement deep linking so that when the user taps "Chat" inside the web UI, the app can intercept and route to native chat. Over time, route more paths to native modules

## Setup UX

### Entry Points

- Scan QR from `openclaw qr` (camera scanner + base64 decode)
- Paste setup code from a channel-based pairing flow (e.g., Telegram `/pair` flow returns a copyable setup code)
- Manual host/port with optional TLS fingerprint pinning support

### Pairing UX

- After bootstrap connect, show the pending `requestId` and provide explicit approval instructions (CLI approval is canonical)
- Once paired, persist the issued `deviceToken` for future connects

### Multi-Gateway Profiles

Support multiple gateways (home vs cloud) as first-class profiles; warn users that the Gateway host is the source of truth for sessions and auth state in remote mode

## Architecture Diagram

```
Mobile App
+--------------------------------------------------+
|  Feature Router                                   |
|  +---------------------+  +--------------------+ |
|  | Native Screens       |  | WebView Host       | |
|  | (Chat, Sessions,     |  | (Control UI        | |
|  |  Settings, Voice)    |  |  fallback)         | |
|  +----------+----------+  +---------+----------+ |
|             |                        |             |
|  +----------v-----------+           |             |
|  | Gateway Client Core  |<----------+             |
|  | (WS + optional       |                         |
|  |  HTTP/SSE)           |                         |
|  +---+--------+---------+                         |
|      |        |                                    |
|  +---v--+  +--v---+                               |
|  |Secure|  |Local |                               |
|  |Store |  |DB    |                               |
|  +------+  +------+                               |
+--------------------------------------------------+
                    |
                    v
+--------------------------------------------------+
|  OpenClaw Gateway                                 |
|  +---------------+  +----------+  +------------+ |
|  | WS Control    |  | HTTP APIs|  | Hooks      | |
|  | Plane         |  | /v1/*    |  | /hooks/*   | |
|  +---------------+  +----------+  +------------+ |
+--------------------------------------------------+
```

## Debug Tooling

AppReveal (https://github.com/UnlikeOtherAI/AppReveal) provides Playwright-like control over native apps for LLM agents. It is:

- **Debug-only**: Contains Apple private APIs, must never ship in release builds
- **SPM conditional**: Add via Swift Package Manager with a platform/configuration condition so it only compiles in DEBUG
- **Required for verification**: Every feature must be visually clicked through and verified via AppReveal before it's considered done
