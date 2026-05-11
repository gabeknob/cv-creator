# CV Creator Context

## Product Idea

CV Creator is a smart career knowledge base for building adapted CVs. Users register career contexts and fine-grained experiences, then assemble targeted CV documents from those reusable experiences.

The core idea is to treat a CV as a projection of structured career evidence, not as a single static document.

## Product Direction

The long-term product should help users:

- Capture reusable career experiences once.
- Attach each experience to a concrete career context.
- Tag experiences with normalized skills, domains, practices, impact areas, and private user guidance.
- Import existing CVs and reconcile them into the user's career knowledge base.
- Detect duplicate or overloaded experiences and guide merge/split decisions.
- Build targeted CVs from selected experiences.
- Eventually use AI, vector retrieval, and tag graph relationships to recommend experiences and generate cohesive CV narratives.

## MVP Direction

The MVP is intentionally smaller than the full AI recommendation system. It should still validate the core product loop:

1. User signs in.
2. User creates career contexts with at least one experience.
3. User maintains an experience database.
4. User manually selects experiences for a CV.
5. App creates a structured Slate-based CV document.
6. User edits the final document and styling manually.
7. User can export or finalize the CV.

The MVP should not become a generic Google Docs clone. The editor should be structured around resume concepts, even if the final document content is manually editable.

## Example Experience

Source text:

> Modernized critical infrastructure by migrating legacy services to Docker Swarm and refactoring the CI/CD pipeline using GitHub Actions. Orchestrated environment configuration via Infrastructure as Code (IaC) with Ansible, eliminating manual server setup. This initiative reduced deployment time by 50%, achieved zero downtime during updates, and standardized staging and production environments, significantly improving system resilience.

Possible canonical tags:

- Docker Swarm
- CI/CD
- GitHub Actions
- Infrastructure as Code
- Ansible
- Deployment automation
- Infrastructure modernization
- Resilience

Possible metric:

- Reduced deployment time by 50%.

Possible private user tag:

- Do not mention on backend CV.

## Product Principles

- `Experience` is the central domain concept.
- Every experience belongs to exactly one career context.
- No canonical orphan experiences are allowed.
- A career context must have at least one experience in normal user flows.
- CV draft edits do not mutate canonical experience data by default.
- Finished/exported CVs are immutable snapshots.
- AI output is a suggestion until the user confirms it.
- Imported evidence can be disputed or rejected without editing the original source text.
- Canonical experience narratives should be rich enough for future agents to reason over.
- User-facing workflows should stay simple even when the backend model is explicit.

## Initial Stack Direction

- Runtime/package manager: Bun workspaces.
- Frontend: Vite, React, TanStack Router, SlateJS.
- Backend: NestJS.
- API style: GraphQL for application data, REST for file upload/export/health.
- Architecture: Clean Architecture with layer-based packages.
- Primary database: DynamoDB single-table design for user-owned app data.
- Canonical tag catalog: separate DynamoDB table.
- Local database: DynamoDB Local.
- Auth: Clerk for public MVP, mapped to internal user IDs.
- Async jobs: BullMQ with Redis.
- Worker: separate NestJS worker app.
- Object storage: local filesystem adapter first, S3-compatible adapter later.
- AI/retrieval: documented and modeled for future work, not required for first MVP.

## Open Questions

- What is the first CV export format: PDF only, or PDF plus DOCX?
- How much PDF rendering should happen client-side versus server-side?
- What is the exact DynamoDB single-table access pattern list?
- What is the minimum useful canonical tag seed catalog?
- What styling controls are included in the MVP editor?
- How much of split/merge should be in the first UI versus only the domain model?
