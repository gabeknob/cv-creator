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
- Rich text editor: SlateJS.
- Backend framework: NestJS.
- API style: GraphQL for application data.
- REST endpoints: file upload, file download/export, health checks, webhooks, and auth callbacks where needed.
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

DynamoDB fits the portfolio goal because it forces deliberate access-pattern design and gives an AWS-oriented architecture path. DynamoDB Local keeps development local.

BullMQ and Redis are enough for import jobs, deletion orchestration, retries, and worker processing. Kafka is deliberately deferred until there is a real scale or integration need.

Object storage is required because DynamoDB has a 400 KB item limit and because large Markdown narratives, Slate documents, uploaded files, and exported PDFs should not live directly inside table items.

## Consequences

Benefits:

- The project has a clear portfolio architecture.
- API and worker are separate runnable apps without functional coupling.
- Clean Architecture boundaries are visible.
- Local development is realistic without production AWS resources.
- The model supports future AI and retrieval features without requiring them in the MVP.

Tradeoffs:

- NestJS, GraphQL, Clean Architecture, DynamoDB, Redis, and object storage add learning overhead.
- Single-table DynamoDB design requires access patterns to be documented before implementation.
- SlateJS document snapshots couple the CV document format to the editor.
- Splitting infrastructure into multiple packages increases workspace configuration.

## Follow-ups

- Define root Bun workspace configuration.
- Define GraphQL schema boundaries.
- Define DynamoDB single-table access patterns.
- Define local Docker Compose for DynamoDB Local and Redis.
- Define object storage adapter interface.
- Define the first Clerk auth integration approach.
