# UpClaw - Sources & References

## Official OpenClaw Resources

- **Documentation**: docs.openclaw.ai — protocols, configuration, security, platforms, plugins, and APIs
- **Source Code**: Published on GitHub, including the Gateway, Control UI, and mobile apps under `apps/ios` and `apps/android`
- **License**: MIT License
- **Community**: Discord server alongside the GitHub repository and releases pages
- **Plugin SDK**: `plugin-sdk/*` subpaths for external extensions (channels, providers, tools)
- **Skill Registry**: ClawHub — discovery surface for skills with versioning and "usage signals"

## Existing Mobile Clients

### Official (In-Repo)

- **iOS app**: Labelled "Super Alpha" and "internal-use only," connects to Gateway as `role: node`
- **Android app**: "Has not been publicly released yet," buildable from source under `apps/android`. Documents a foreground service that keeps the connection alive and enumerates chat/canvas/camera/voice capabilities via WS methods

### Third-Party

- A Play Store listing named "OpenClaw" published by "Moltbook" exists under `com.openclaw.ai` with generic "AI coworker" marketing. Should be treated as unaffiliated unless proven otherwise. This external-brand collision is a concrete distribution risk.

## Protocol References

- **Gateway WebSocket Protocol**: TypeBox/JSON-Schema as source of truth for protocol models
- **Legacy Bridge Protocol** (deprecated): TCP JSONL, kept for historical reference. Current builds no longer ship the TCP bridge listener.
- **Security Advisory**: Setup code bootstrap token change in 2026.3.12 (previously embedded long-lived credentials)

## Debug Tooling

- **AppReveal**: https://github.com/UnlikeOtherAI/AppReveal — Debug-only in-app MCP server for iOS. Gives LLM agents Playwright-like control over native apps (UI, state, navigation, diagnostics over local network). Contains Apple private APIs — must never be included in release builds.
