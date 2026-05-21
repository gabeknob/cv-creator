# ADR 0001: Initial Application Stack

## Status

Accepted

## Context

The project is a smart CV creator with a strong portfolio and learning angle. It should demonstrate modern frontend development, GraphQL, Clean Architecture, local development ergonomics, cloud-oriented data modeling, and a future path toward AI-assisted CV generation.

The first MVP is not the full AI recommendation system. It should focus on manual experience registration, structured CV assembly, and a strong architecture foundation.

## Decision

Use the following initial stack:

- Package manager/runtime tooling: Bun workspaces.
- Frontend: Vite, React, TanStack Router.
- UI components and app theme: shadcn/ui with a generated base theme preset.
- Server state and caching: React Query.
- Local client UI state: Zustand where route state, server state, editor state, or component state are not the right owner.
- HTTP client: Axios for REST-shaped operations.
- REST client generation: Orval from OpenAPI/Swagger.
- GraphQL client generation: GraphQL Code Generator.
- Motion: Framer Motion for functional interaction feedback.
- Rich text fields: SlateJS inside structured CV editing forms.
- Text measurement and fit estimation: Pretext.
- Frontend/component testing: Vitest and React Testing Library.
- Browser workflow testing: Playwright E2E.
- Backend framework: NestJS.
- API style: GraphQL for application data.
- REST endpoints: file upload, file download/export, health checks, webhooks, and auth callbacks where needed.
- WebSocket endpoints: authenticated long-running workflow status updates, with PDF export as the first producer.
- Public backend entrypoint: the NestJS API app; no separate API gateway service in the MVP.
- Primary database: DynamoDB.
- Local database: DynamoDB Local.
- App data modeling: single-table DynamoDB design for user-owned domain data.
- Canonical tag catalog: separate DynamoDB table.
- Authentication: Clerk for public MVP, mapped to internal user IDs.
- Async queue: BullMQ with Redis.
- Worker: separate NestJS worker app using application context and shared application layer.
- Object storage: local filesystem adapter first, S3-compatible adapter later.
- Vector store: deferred until recommendation features begin.

## Rationale

NestJS gives first-class support for GraphQL, decorators, dependency injection, modules, and a backend style that fits the desired Clean Architecture learning goal.

GraphQL is useful for application data because the UI will need nested career data, CV drafts, tags, contexts, and experience relationships. REST remains better for file-shaped operations such as uploads and exports.

GraphQL Code Generator keeps application-data operations typed. Orval keeps the smaller REST surface typed as well, even if the initial REST API only covers uploads, downloads, exports, health checks, webhooks, or transport-shaped endpoints. The goal is to make generated clients the default rather than hand-written request wrappers.

shadcn/ui provides the product UI component foundation and generated base theme for app chrome. CV document styling remains separate because exported CVs are print/document artifacts controlled by CV templates and editor style settings, not by the application theme.

DynamoDB fits the portfolio goal because it forces deliberate access-pattern design and gives an AWS-oriented architecture path. DynamoDB Local keeps development local.

BullMQ and Redis are enough for import jobs, deletion orchestration, PDF export jobs, retries, and worker processing. Kafka is deliberately deferred until there is a real scale or integration need.

WebSockets are used for user-facing status updates when a workflow is worker-backed but should feel active and synchronous to the user. PDF export is the first such workflow; future import, generation, and destructive cleanup workflows can use the same channel. The API owns the WebSocket connection and authentication; workers remain queue-driven and not directly user-facing.

The MVP does not introduce a separate API gateway service. The NestJS API app owns public backend concerns such as Clerk authentication, internal user resolution, validation, CORS, rate limits, GraphQL limits, REST transport endpoints, WebSocket auth, and queue admission. A dedicated gateway can be reconsidered if the system later has multiple independently deployed public services or stronger edge/security requirements.

Domain records use internal UUIDv7 user IDs, not Clerk IDs. Clerk identities map to internal users through an identity mapping record. User creation should be lazy-first: the API creates or resolves the internal user on the first authenticated request, while Clerk webhooks can be added later for lifecycle sync and cleanup.

Object storage is required because DynamoDB has a 400 KB item limit and because large Markdown narratives, semantic CV document snapshots, uploaded files, and exported PDFs should not live directly inside table items.

Pretext gives the frontend a fast way to estimate wrapped text height for CV blocks without repeatedly measuring hidden DOM nodes. This is important for the MVP because CV drafts should respect page constraints such as A4 size, margins, font size, and line height.

Framer Motion should be used for functional motion such as panel transitions, reorder feedback, modals, drawers, status changes, and suggestion appearance or dismissal. Motion should clarify editor state changes rather than make the CV canvas feel decorative.

## Consequences

Benefits:

- The project has a clear portfolio architecture.
- API and worker are separate runnable apps without functional coupling.
- Clean Architecture boundaries are visible.
- Local development is realistic without production AWS resources.
- The model supports future AI and retrieval features without requiring them in the MVP.
- CV editing can warn about page overflow and estimate fit without expensive layout thrashing.
- Generated GraphQL and REST clients reduce frontend/backend contract drift.
- Functional motion can make complex editor state changes easier to follow.

Tradeoffs:

- NestJS, GraphQL, Clean Architecture, DynamoDB, Redis, and object storage add learning overhead.
- Single-table DynamoDB design requires access patterns to be documented before implementation.
- SlateJS rich text values must be serialized and rendered consistently in structured forms, preview HTML, and export HTML.
- Pretext estimation must stay synchronized with the active CV template, font settings, margins, and line height.
- Orval and GraphQL Code Generator add frontend scaffolding and generation steps that must stay integrated with local development.
- Motion usage must stay restrained so the editor remains a precise work surface.
- Splitting infrastructure into multiple packages increases workspace configuration.

## Follow-ups

- Define root Bun workspace configuration.
- Define GraphQL schema boundaries.
- Define DynamoDB single-table access patterns.
- Define local Docker Compose for DynamoDB Local and Redis.
- Define object storage adapter interface.
- Define the first Clerk auth integration approach.
