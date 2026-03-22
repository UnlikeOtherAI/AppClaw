# AppClaw - Agent Standards

## Project Overview

AppClaw is a free, open-source, community-driven project building native iOS and Android clients for OpenClaw. All existing options are paid — AppClaw fills that gap.

## File Size Rules

- Documentation: max 1,000 lines per file
- Code: max 500 lines per file
- Exempt: JSON and YAML data files
- Over-limit docs: create a folder with `overview.md` (table of contents + links to sub-sections)

## Phase 1 Scope

- iOS only: iPhone + iPad (iOS and iPadOS)
- Native SwiftUI with native components
- Clean Architecture (domain, data, presentation)

## Debug Tooling

- AppReveal is debug-only — never include in release/production builds
- Every feature must be visually verified through AppReveal before it's considered done
- Test against a real OpenClaw instance

## Wireframes

- After ANY wireframe change, rebuild (`wf build`) and verify with Playwright before committing
- Check that text content renders (not default "Heading"), tab bars are present, and navigation works
- Run `wf validate` before building — fix all errors

## Code Standards

- Minimum complexity, no premature abstractions
- Native Swift conventions and patterns
- SwiftUI for all UI — no UIKit unless absolutely necessary
