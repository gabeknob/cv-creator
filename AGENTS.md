@/Users/gabrielnobrega/.codex/RTK.md

# Agent Instructions

These instructions apply to all agent work in this repository.

## Project Context

Read `CONTEXT.md`, `docs/domain-model.md`, and the relevant ADRs before making architectural or product-shaping changes.

The MVP is a structured CV creator backed by a reusable experience database. Do not turn the product into a generic document editor.

## Skill Commands

Treat slash-prefixed skill names as skill invocations.

Examples:

- `/caveman` means use `$caveman`
- `/diagnose` means use `$diagnose`
- `/tdd` means use `$tdd`
- `/to-issues` means use `$to-issues`
- `/to-prd` means use `$to-prd`
- `/zoom-out` means use `$zoom-out`

If a user message starts with `/NAME` and `NAME` matches an available skill, load that skill and follow it.

## Architecture Directives

- Preserve Clean Architecture dependency direction.
- Keep domain code free from framework, database, GraphQL, NestJS, and SDK imports.
- Keep API and worker apps functionally decoupled; they may share lower-level packages, but must not import from each other.
- Use GraphQL for application data and REST for upload, export, health, webhook, and transport-shaped operations.
- Store large blobs in object storage, not directly in DynamoDB items.
- Treat `Experience` as the central domain concept.
- Do not allow canonical orphan experiences.
- Do not mutate canonical experience data from CV draft edits by default.

## Frontend Directives

- Prefer deriving state during render over synchronizing state with effects.
- Do not use `useEffect` unless the component is synchronizing with an external system or there is a concrete reason that cannot be modeled with rendering, event handlers, router loaders, query state, or server state.
- Keep the app structured around CV workflows: experience database, CV selection, structured editor, finalize/export.
- Prefer controlled, resume-specific editing and styling over arbitrary free-position layout tools in the MVP.

## Version Control Directives

- Follow a trunk-based development style: keep changes small, integrate frequently, and avoid long-lived divergent branches.
- Commits must follow Conventional Commits.
- Use the format `type(scope): summary` where `scope` is the app, package, or domain area affected.
- Prefer these commit types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`, `build`, `ci`, and `perf`.
- Use `!` or a `BREAKING CHANGE:` footer for breaking changes.
- Commit summaries should explain the user-visible or architectural intent of the change.
- Do not commit unrelated changes together.
