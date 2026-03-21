# UpClaw - Priority Native Features

## Selection Criteria

Features selected for native implementation are:
- Explicitly supported by current protocol/schema/docs
- High-frequency in the "daily driver" workflow
- Provide clear native value (latency, offline, device integration, notifications)

Everything not in this list — full config editing, plugin management, advanced ops dashboards, niche tools — remains in the Control UI WebView initially.

## Feature Matrix

| # | Feature | Value | Native vs Web | Effort | Dependencies |
|---|---------|-------|---------------|--------|--------------|
| 1 | Gateway connect & discovery | Fast setup; reduces friction vs manual URLs | Native | Medium | WS `connect` + discovery (LAN/tailnet/SSH) |
| 2 | QR/setup code bootstrap | Camera scan and paste are mobile-native | Native | Medium | Setup code carries `url` + `bootstrapToken` (`openclaw qr`, `/pair`) |
| 3 | Device pairing workflow | Required first-run; must be understandable on mobile | Native | Medium | Device pairing approvals; device token returned in `hello-ok` |
| 4 | Session list + search + previews | Navigation across main, groups, jobs; reduces web dependency | Native | Medium | `sessions.list` with `search`, `includeLastMessage`, `includeDerivedTitles`, filters |
| 5 | Live transcript subscription | Real-time updates without reload; avoids heavy polling | Native | Medium | `sessions.messages.subscribe/unsubscribe` + `session.message` semantics |
| 6 | Session settings (fast/reasoning/usage) | High leverage on cost/latency/UX | Native | Low-Medium | `sessions.patch` with `fastMode`, `reasoningLevel`, `thinkingLevel`, `responseUsage` |
| 7 | Image attachments in chat | Core modern chat expectation | Native | Medium | `chat.send` attachments + image-only parsing & 5MB decode limit |
| 8 | Voice input + talk mode UI | Hands-free use is a main differentiator for mobile nodes | Native | High | Android node voice behaviour; "Talk" config; node caps/commands |
| 9 | Push notifications / reconnect wakes | Makes mobile reliable; enables "always available" feel | Native | High | iOS relay-backed push flow + `push.apns.register` publishing |

## How People Use OpenClaw

The dominant usage model is "message the assistant where you already are," with OpenClaw acting as the always-on bridge between messaging channels and agent runtimes. Key patterns:

- Multi-channel messaging, media in/out, multi-agent routing, and mobile nodes (voice/chat + device commands)
- "Do things" workflows combining chat with tools/plugins (automation, job search, Jira skill building, etc.)
- Users run the Gateway on a "master" machine and connect from other devices via LAN discovery (Bonjour/mDNS/NSD) or via remote access (Tailscale/SSH tunnel)
- The project is investing heavily in session UX and chat tooling: modular dashboard with mobile-friendly navigation and richer chat tools like slash commands, search, export, and pinned messages
- Users accumulate long-running sessions — efficient paging, local caching, and targeted subscriptions are critical on mobile

## Telemetry

OpenClaw is self-hosted with no single global analytics feed. It provides structured diagnostics: a "Diagnostics + OpenTelemetry" pipeline that can emit model usage and message-flow events via OTLP/HTTP when enabled. A mobile client can offer per-session usage/cost views by querying gateway surfaces (session metadata and usage fields) rather than relying on external tracking.
