# CV Creator

CV Creator is a smart career knowledge base for building adapted CVs. Users register career contexts and fine-grained experiences, then assemble targeted CV documents from those reusable experiences.

The long-term goal is to support AI-assisted import, reconciliation, tagging, retrieval, and CV generation. The MVP is smaller: it focuses on a manual experience database and structured CV builder that preserves the architecture needed for those future features.

## MVP Scope

- Create career contexts such as jobs, projects, education, certifications, awards, and open-source work.
- Register fine-grained experiences under those contexts.
- Add canonical tags, private user tags, and metrics to experiences.
- Select experiences for a CV draft.
- Edit the generated structured CV document with SlateJS.
- Finalize and export immutable CV snapshots.

## Tech Stack

- Runtime and package management: Bun workspaces.
- Frontend: Vite, React, TanStack Router, SlateJS.
- Backend: NestJS.
- API: GraphQL for application data; REST for upload, export, health, and webhook-style operations.
- Auth: Clerk, mapped to internal user IDs.
- Database: DynamoDB with single-table design for user-owned app data.
- Canonical tags: separate DynamoDB catalog table.
- Local database: DynamoDB Local.
- Background jobs: BullMQ with Redis.
- Worker: separate NestJS worker app.
- Object storage: local filesystem adapter first, S3-compatible adapter later.
- Future retrieval: vector search and RAG are planned but not part of the first MVP.

## Documentation

- [Product context](./CONTEXT.md)
- [Domain model](./docs/domain-model.md)
- [MVP PRD](./docs/prd/0001-cv-creator-mvp.md)
- [Architecture decisions](./docs/adr/)

## Current Status

The repository is currently in planning/scaffolding mode. Product context, architecture decisions, and the MVP PRD are documented before implementation begins.
