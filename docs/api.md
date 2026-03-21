# AppClaw - API Reference

OpenClaw presents three main integration planes relevant to a mobile client:

- **WS control plane (typed RPC + events)** for real-time operator/node interactions (`connect`, chat send/history, sessions list/patch, node invoke)
- **HTTP "compatibility" plane** for OpenAI/OpenResponses clients (SSE streaming) and session transcript streaming (`follow=1`)
- **HTTP webhooks** for inbound triggers (wake/agent)

## WebSocket Control Plane

### connect + connect.challenge

- **Purpose**: Handshake; declare role/scopes/caps; establish session
- **Auth**: `connect.params.auth.token/password`; device signature required
- **Request fields**:
  - `minProtocol/maxProtocol`
  - `client{ id, version, platform, mode }`
  - `role` (`operator` or `node`)
  - `scopes` (e.g., `operator.read`, `operator.write`, `operator.admin`)
  - `caps/commands/permissions` (for nodes)
  - `device{ id, publicKey, signature, signedAt, nonce }`
- **Response**: `hello-ok` res; may include `auth.deviceToken`
- **Error codes**: `DEVICE_AUTH_*`, `AUTH_TOKEN_MISMATCH` with retry guidance

### chat.history

- **Purpose**: Load recent messages for a session
- **Auth**: Operator WS session; scope-gated by role
- **Request**: `{ sessionKey, limit? (<=1000) }`
- **Response**: Returns transcript payload
- **Notes**: Not cursor-paged in schema shown; large histories are a known pain point

### chat.send

- **Purpose**: Send message; may trigger agent run; optional delivery
- **Auth**: Operator WS session; idempotency required
- **Request**: `{ sessionKey, message, thinking?, deliver?, attachments?, timeoutMs?, idempotencyKey }`
- **Response events**: `{ runId, sessionKey, seq, state: delta|final|aborted|error, message?, usage? }`
- **Attachments**: Images only; base64 validated; default `maxBytes=5,000,000` decoded bytes; non-image attachments are dropped

### chat.abort

- **Purpose**: Abort in-flight run
- **Request**: `{ sessionKey, runId? }`
- **Notes**: Useful for "Stop generating" UX

### sessions.list

- **Purpose**: List sessions with optional derived title, last-message preview, and search
- **Request**: `{ limit?, activeMinutes?, includeDerivedTitles?, includeLastMessage?, label?, spawnedBy?, agentId?, search? }`
- **Response**: Returns session rows
- **Notes**: Derived titles read 8KB transcript; last message preview reads 16KB — use `limit` carefully on large stores

### sessions.preview

- **Purpose**: Fetch previews for selected session keys
- **Request**: `{ keys[], limit?, maxChars? }`
- **Notes**: Enables efficient session list rendering without full history

### sessions.messages.subscribe / unsubscribe

- **Purpose**: Subscribe to transcript updates for one session
- **Request**: `{ key }`
- **Response**: Emits `session.message` events (appended transcript messages + live usage metadata)
- **Notes**: Prefer over reloading `chat.history` on every event for mobile efficiency

### sessions.patch

- **Purpose**: Update per-session settings
- **Request**: `{ key, label?, thinkingLevel?, fastMode?, verboseLevel?, reasoningLevel?, responseUsage?, elevatedLevel?, execHost?, ... }`
- **Notes**: Directly supports "fast mode" and "reasoning level" UX without parsing slash-command text

## HTTP APIs

### POST /v1/chat/completions

- **Purpose**: OpenAI-compatible wrapper for agent runs
- **Auth**: `Authorization: Bearer <token/password>`
- **Request**: Standard chat-completions body; select agent via `model:"openclaw:<agentId>"` or `x-openclaw-agent-id`; session via `x-openclaw-session-key`
- **Streaming**: SSE when `stream:true`; ends with `[DONE]`
- **Notes**: Treated as "full operator access"; keep private

### POST /v1/responses

- **Purpose**: OpenResponses-compatible wrapper; supports files/images
- **Auth**: `Authorization: Bearer <token/password>`
- **Request**: OpenResponses items; `stream:true` enables SSE; various file/image guards and allowlists
- **Streaming**: SSE event types: `response.output_text.delta`, etc.
- **Limits**: `maxBodyBytes=20MB`, `files.maxBytes=5MB`, `images.maxBytes=10MB`, SSRF guards + allowlists

### GET /sessions/{sessionKey}/history

- **Purpose**: Paged transcript access; optional SSE follow
- **Query params**: `limit`, `cursor`, `includeTools=1`, `follow=1`
- **Streaming**: `follow=1` upgrades to SSE transcript updates
- **Errors**: Unknown sessions return `404` with `error.type="not_found"`

## HTTP Webhooks

### POST /hooks/wake

- **Purpose**: Wake/nudge hook
- **Auth**: `Authorization: Bearer ...`
- **Request**: JSON `{}`
- **Notes**: Tokens in query string are rejected

### POST /hooks/agent

- **Purpose**: Trigger agent run via webhook
- **Auth**: `Authorization: Bearer ...`
- **Request**: JSON `{ message, sessionKey? }`
- **Notes**: Session routing via `sessionKey` is critical for conversational continuity

## Authentication Flows

### Gateway Shared Auth (token/password)

Both WS clients and HTTP endpoints use Gateway auth configuration. WS uses `connect.params.auth.token/password`; HTTP uses `Authorization: Bearer <token>` (token or password depending on mode).

### Device Pairing + Per-Device Token

New devices require pairing approval. After pairing, the Gateway issues a per-device token returned in `hello-ok` that clients persist.

### Bootstrap Setup Codes (mobile-friendly)

Setup codes (via `/pair` or `openclaw qr`) are base64-encoded JSON carrying a WS URL and a short-lived `bootstrapToken` used for initial pairing. This replaced an earlier design that embedded long-lived shared credentials in setup payloads; a security advisory documents the impact and patch in 2026.3.12.

### Provider Auth (OAuth/API keys)

Provider credentials (OAuth tokens, API keys, setup-tokens) are stored per agent under `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`. OAuth uses PKCE and supports refresh/expiry handling.

### Tailscale Identity Headers (optional)

For Control UI/WebSocket, tokenless access via identity headers can be enabled (trusting the gateway host); HTTP APIs still require token/password. Remote access is explicitly positioned as "preferred: VPN/Tailscale; fallback: SSH tunnel."

## Rate Limits & Security Boundaries

- **Config RPC throttling**: `config.apply`/`config.patch` rate-limited to 3 requests per 60 seconds per `(ws deviceId + clientIP)`
- **HTTP auth failure rate limiting**: Optional `gateway.auth.rateLimit` returning `429` + `Retry-After`
- **Origin allowlist for web UI**: `gateway.controlUi.allowedOrigins` must be set explicitly for non-loopback deployments
- **Bearer token = operator**: Bearer auth for `/v1/*` endpoints is "full operator access," not per-user scoped
