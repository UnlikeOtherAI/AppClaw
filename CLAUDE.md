# AppClaw - Project Rules

All standards live in `@./AGENTS.md`.

## File Size Limits

- No documentation file may exceed 1,000 lines
- No code file may exceed 500 lines
- These limits do NOT apply to JSON or YAML data files
- If a document must exceed 1,000 lines, split it into a folder with an `overview.md` containing a table of contents and links to sub-sections

## UI & Wireframes

- For any UI-related query, consult the wireframes in `docs/wireframes/` first
- Wireframes are the source of truth for screen layout, navigation flow, and component structure
- Build with `wf build` and preview with `wf serve` from the `docs/wireframes/` directory

## Architecture

- Clean Architecture: domain, data, presentation layers
- Phase 1: iOS only (iPhone + iPad), native SwiftUI
- Use native components wherever possible

## Testing & Verification

- No feature is considered finished until it has been visually clicked through and verified using AppReveal
- AppReveal (https://github.com/UnlikeOtherAI/AppReveal) is debug-only — it MUST NOT be included in production/release builds (contains Apple private APIs)
- AppReveal must be configured to only compile and link in DEBUG mode
- Test against a real OpenClaw instance — ask for connection details if needed

## Dependencies

- AppReveal: debug-only, excluded from release builds via Swift Package Manager conditions or build configuration
