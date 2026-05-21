# PRD: CV Creator MVP

## Problem Statement

Professionals often have more relevant experience than fits into one CV. Adapting a CV manually is slow because the user has to remember which accomplishments, projects, tools, metrics, and responsibilities matter for each opportunity.

Existing document editors let users write a CV, but they do not help users maintain a reusable career knowledge base. Existing AI tools can rewrite text, but they often blur the line between grounded career facts and generated phrasing.

The user needs a structured CV creator that starts with a reusable experience database, lets them assemble different CVs from those experiences, and preserves a foundation for future AI-assisted import, reconciliation, retrieval, and generation.

## Solution

Build an MVP that lets authenticated users create structured blank CV drafts, create and manage career contexts and fine-grained experiences, optionally select experiences for a CV draft, edit the resulting semantic CV document through structured controls and Slate-powered rich text fields, and finalize or export immutable CV snapshots.

The MVP does not include AI recommendations or RAG runtime. It still establishes the domain and architecture required for those features later: experience-centered modeling, canonical and user tags, immutable evidence concepts, object storage for large blobs, split/merge lineage, worker orchestration, and clean backend boundaries.

## UX Principles

- The app should feel like a structured CV creator, not a blank document editor.
- The experience database is the source-of-truth workflow, not an implementation detail hidden behind the editor.
- Users should be able to create reusable career contexts and experiences before building a CV.
- CV creation may start from selected experiences or from a structured blank CV draft.
- Blank CV drafts should use resume-specific blocks rather than arbitrary freeform page content.
- CV document blocks should be semantically typed so the app can distinguish resume concepts from generic rich text.
- The final CV builder should support manual rich text editing while preserving resume structure.
- Styling should be controlled in the MVP: templates, fonts, margins, line height, section order, and show/hide controls are allowed; arbitrary free-position layout editing is deferred.
- Draft-local edits must never mutate canonical experience data by default.
- Manually typed CV content may prompt the user to save or link reusable experiences, but canonical data only changes after confirmation.
- MVP save/link suggestions may be rule-based from semantic CV blocks; future versions may replace or enrich this with AI and vector search.
- The CV builder should be desktop-first with a central read-only preview canvas and supporting panels for outline, experience suggestions/library, styling, selected block settings, and fit feedback.
- The desktop builder should use a left panel with Outline, Experiences, and Suggestions tabs, and a right panel with Design, Block, and Fit tabs.
- The CV preview canvas must behave as a real-time 1:1 preview of the exported PDF, sharing semantic document structure and print layout rules with the export renderer. Users edit through structured controls and rich-text fields rather than directly editing the preview canvas.
- CV draft edits should autosave with visible save state; finalization and export remain explicit actions.
- The MVP is online-only; autosave failures should be visible, while offline editing/resilience is deferred.
- Builder navigation should warn when autosave is saving, failed, or unsaved; saved drafts can be left freely.
- Finalized CVs should open in a read-only completed-document view with download, metadata, and duplicate-as-draft actions.
- In the MVP, exporting a PDF finalizes the CV automatically, stores the immutable snapshot and artifact, and preserves old PDF downloads.
- CV drafts should default to `maxPages = 1`; overflow pages remain visible in preview, but export is blocked when content exceeds the configured max page target.
- Setting `maxPages` above 1 should show a softer guidance warning about the common one-page-per-ten-years recommendation.
- The builder top bar should show compact page-fit status, while the Fit tab owns detailed page target controls, overflow details, and export blockers.
- The MVP Create CV setup should ask for title, optional target/context, max pages, and whether to start blank or from selected experiences.
- Recent CV cards should include a preview thumbnail and key status metadata, with direct quick actions limited to PDF download when available.
- CV preview thumbnails should be generated during export/finalization, stored as artifacts, and loaded by the home page with browser caching.
- Draft CV cards do not need generated thumbnails in the MVP; use a placeholder/status treatment until export/finalization.
- Opening a draft should go directly to the builder; opening a finalized/exported CV should go to the read-only completed-document view.
- Creating from saved experiences may include an initial selection step, and the builder Experiences panel remains available for ongoing add/remove/link actions.
- The MVP experience database UI should cover basic context/experience management, tags, and metrics; full split/merge/reconciliation UX is deferred while the domain model preserves those concepts.
- The experience database should default to a context-grouped, unfiltered-CV-like view, with an all-experiences toggle/list mode for scanning and filtering individual experiences.
- Context and experience editing should use side sheets in the MVP, keeping users oriented in the database while editing details.
- Canonical tag chips in the experience database should visibly show strength as primary, supporting, or incidental; user/private tags remain separate and do not need strength in the MVP.
- The builder Experiences panel should use compact selection cards with expandable details for tags, metrics, strength, and narrative preview.
- The builder Suggestions tab should provide deterministic save/link and draft-quality guidance, not AI recommendations in the MVP.
- The builder Design tab should provide opinionated template, page, spacing, density, and font-style-category controls rather than arbitrary full theme editing.
- The builder Block tab should provide common selected-block controls plus type-specific fields for the selected resume block.
- Users should be able to select semantic blocks from either the read-only preview canvas or the Outline tab; selection opens Block tab editing without direct text editing in the preview.
- Block reordering should happen through Outline drag-and-drop and Block tab move controls, not preview drag-and-drop.
- The MVP should support a template concept but ship with one strong ATS-friendly template first.
- If final export rendering exceeds `maxPages` despite preview estimates, offer the user a choice to return to the builder or increase `maxPages` and export; this should be rare because preview/export parity is a quality requirement.
- Motion should be functional and restrained, helping users follow state changes without making the CV canvas feel decorative.
- Authenticated navigation should be hybrid: the home and CV builder stay workspace-focused, while sidebars are reserved for settings, admin, or deeper management pages where they add orientation.
- Global search should be unified across CV drafts/finalized CVs, career contexts, experiences, and relevant tags from one discreet search entry point.
- Global search should behave like a command palette with grouped keyboard-navigable results and direct navigation.
- Experience data should be available in both a dedicated management page and a lightweight builder panel; the former manages career knowledge, while the latter supports CV assembly and linking.
- The public MVP should include a very short landing page that explains the product and routes users to authentication.
- Application UI styling and CV document styling should remain separate: shadcn/ui presets style the product chrome, while CV templates and style settings control the document/PDF.
- Mobile editing does not need full parity in the MVP, though the app should degrade safely on smaller screens.
- MVP mobile behavior is view/download only; the full CV builder may require a desktop-sized viewport.
- Finished/exported CVs are immutable snapshots.
- The MVP user-facing export format is PDF.
- PDF export should be server-rendered through semantic HTML and print CSS using Playwright/Chromium, preserving selectable text and ATS-friendly reading order.
- PDF export should be worker-backed while feeling synchronous to the user through the authenticated workflow status WebSocket channel.
- Empty states and creation flows should teach the product model without blocking a structured blank CV draft.
- UX for advanced operations such as split, merge, hard delete, and export should be designed explicitly before implementation.

Empty home state:

- Primary action: create first CV.
- Secondary action: open experience database.
- Keep copy short and focused on the structured CV workflow.

Minimal settings:

- Profile/contact defaults for CV headers.
- Defaults are copied into new CV drafts at creation time and are not live-linked to existing drafts.
- User account display/sign-out through Clerk-owned auth UI.
- Avoid broad account-management scope in the MVP.

## User Stories

1. As a user, I want to sign in, so that my career data is private to me.
2. As a user, I want my account to have an internal app identity, so that the app is not permanently coupled to the auth provider.
3. As a user, I want to create a career context with an initial experience, so that I never create empty containers.
4. As a user, I want to classify a career context as a job, personal project, open-source work, education, certification, award, or other, so that my experience database reflects where work happened.
5. As a user, I want to record organization, role, dates, location, and description for a career context, so that selected CV entries have useful context.
6. As a user, I want to add many experiences under one career context, so that a long job or project can be reused selectively.
7. As a user, I want each experience to represent a fine-grained accomplishment or responsibility, so that I can choose only the parts relevant to a specific CV.
8. As a user, I want to write a detailed canonical narrative for an experience, so that future CV generation has rich source material.
9. As a user, I want canonical narratives to support Markdown, so that they remain readable, portable, and useful for future agents.
10. As a user, I want to add a short title and preview summary to an experience, so that I can scan my experience database quickly.
11. As a user, I want to tag an experience with canonical skills, tools, practices, domains, and impact areas, so that I can organize and filter my experience database.
12. As a user, I want canonical tags to normalize aliases like Go and Golang, so that similar skills do not fragment my data.
13. As a user, I want to add private user tags, so that I can store personal guidance that should not appear in exported CVs.
14. As a user, I want to mark tag strength as primary, supporting, or incidental, so that future recommendations can distinguish deep experience from passing mentions.
15. As a user, I want to store metrics for an experience, so that measurable impact can be reused in CV drafts.
16. As a user, I want to archive a career context or experience, so that old data stops appearing in active workflows without being destroyed.
17. As a user, I want hard deletion to require explicit confirmation, so that I do not accidentally remove important career history.
18. As a user, I want hard deletion to clean up related records and storage artifacts, so that deleted data does not linger unexpectedly.
19. As a user, I want to split an overloaded experience into multiple experiences, so that one large project can become reusable CV-ready claims.
20. As a user, I want to merge duplicate or overlapping experiences, so that repeated imports or manual entries do not create confusion.
21. As a user, I want split and merge operations to preserve lineage, so that I can understand where an experience came from.
22. As a user, I want evidence concepts in the data model, so that future imports can preserve traceability from claims to sources.
23. As a user, I want bad evidence to be marked disputed or rejected instead of edited in place, so that corrections do not erase audit history.
24. As a user, I want to create a structured blank CV draft, so that I can start manually before my experience database is complete.
25. As a user, I want to create a CV draft from selected experiences, so that my CV can be assembled from reusable career data.
26. As a user, I want to manually select which experiences go into a CV, so that I control the story being told.
27. As a user, I want selected experiences to be grouped by their career context, so that the CV still reads like a conventional resume.
28. As a user, I want to edit the generated CV draft text manually, so that the final wording fits the opportunity.
29. As a user, I want edits in a CV draft to stay local to that draft, so that I do not accidentally corrupt canonical experience data.
30. As a user, I want the CV builder to be structured around resume sections, so that the app is more useful than a blank document editor.
31. As a user, I want rich text editing inside the CV document, so that I can control wording and presentation.
32. As a user, I want to add, remove, and reorder resume-specific blocks, so that I can adapt the CV structure without arbitrary layout editing.
33. As a user, I want the app to suggest saving manually typed experience-like content as reusable experiences, so that my database can grow while I create CVs.
34. As a user, I want to confirm save/link suggestions before canonical experiences change, so that draft editing does not unexpectedly mutate my career knowledge base.
35. As a user, I want controlled styling options such as template, font, font size, margins, line height, section order, and show/hide sections, so that I can tune the CV without arbitrary layout complexity.
36. As a user, I want a max-pages target defaulting to one page, so that I can control the intended CV length.
37. As a user, I want to see overflow pages in the preview with clear warnings, so that I understand what content exceeds my target.
38. As a user, I want export to be blocked when the CV exceeds the configured max-pages target, so that I do not accidentally finalize an overlong CV.
39. As a user, I want finalized CVs to become immutable snapshots, so that completed documents do not change when source experiences change.
40. As a user, I want to duplicate a finalized CV into a new editable draft, so that I can create a variation without modifying the original.
41. As a user, I want exported CV files to be stored as artifacts, so that I can retrieve the exact document I finalized.
42. As a user, I want large narratives, document snapshots, uploads, and exports to be stored outside the database, so that the system handles large content safely.
43. As a user, I want to export finalized CVs as PDFs, so that I can use them in job applications.
44. As a developer, I want the API to expose application data through GraphQL, so that the frontend can query nested career and CV data naturally.
45. As a developer, I want file upload and export operations to use REST, so that transport-shaped operations stay simple.
46. As a developer, I want the backend to use Clean Architecture, so that domain rules, use cases, infrastructure, and delivery code stay separated.
47. As a developer, I want API and worker apps to share application use cases without importing from each other, so that background work and user requests use the same rules.
48. As a developer, I want repository ports to be abstract contracts, so that infrastructure implementations can be swapped and tested.
49. As a developer, I want presenters at delivery boundaries, so that use case output is adapted explicitly to GraphQL and REST response shapes.
50. As a developer, I want DynamoDB single-table design for user-owned app data, so that the project teaches real DynamoDB access-pattern modeling.
51. As a developer, I want canonical tags in a separate catalog, so that global tag definitions and relations are not mixed with user-owned career data.
52. As a developer, I want a local object storage adapter, so that development can work without production cloud storage.
53. As a developer, I want a future S3-compatible object storage adapter, so that public deployment can store documents and exports safely.
54. As a developer, I want Redis-backed worker queues, so that imports and destructive cleanup can be retried outside request/response flows.
55. As a developer, I want imports to be modeled as asynchronous workflows, so that future CV parsing can handle files, extraction, LLM calls, and reconciliation.
56. As a developer, I want CV generation to remain synchronous for the initial MVP, so that the first product avoids unnecessary job complexity.
57. As a developer, I want the model to include target profiles and CV strategies for future work, so that recommendations can be added without redesigning the core domain.
58. As a developer, I want vector retrieval and RAG deferred until recommendations begin, so that the MVP does not run unused infrastructure.

## Implementation Decisions

- The MVP is a manual structured CV creator that supports structured blank CV drafts and is backed by a reusable experience database.
- CV drafts are composed from resume-specific blocks rather than arbitrary freeform page content.
- CV document snapshots preserve semantic resume block types and Slate-compatible rich text field values.
- Experience is the central aggregate. Career contexts, CV drafts, imports, tags, metrics, evidence, and exports are workflows or relationships around experiences.
- Every experience belongs to exactly one career context.
- Career contexts must be created with at least one experience in normal user flows.
- A career context can have many experiences.
- Similar work in the same career context should be merged into one experience when appropriate.
- Similar work in different career contexts remains separate experiences.
- Experiences can be split and merged as first-class domain operations.
- Split and merge operations preserve lightweight lineage records.
- Career contexts and experiences support archive as the normal removal path.
- Hard deletion is explicit, destructive, cascade-aware, and orchestrated by a worker.
- Canonical experience narratives are Markdown blobs stored in object storage.
- DynamoDB stores metadata, queryable fields, relationships, statuses, and object storage keys.
- CV document snapshots are semantic JSON blobs stored in object storage.
- The backend validates CV document snapshots lightly for structure, version, page settings, size, and unsafe embedded content.
- CV draft edits do not mutate source experiences by default.
- A CV draft may have no source experiences when it starts from a structured blank document.
- Manually typed CV content can produce save/link suggestions for reusable experiences, but those suggestions require user confirmation before changing canonical data.
- Initial save/link suggestions can be rule-based and should not assume the final AI/vector recommendation behavior.
- The primary builder layout is desktop-first, centered on a read-only preview canvas with side panels for supporting workflows.
- Finalized and exported CVs are immutable snapshots.
- User-facing MVP export is PDF; the finalized semantic CV JSON snapshot remains stored internally for exact reconstruction and audit.
- PDF export uses server-side Playwright/Chromium over semantic HTML and print CSS, not screenshot/image-based rendering.
- A shared TypeScript CV renderer should produce the HTML used by both the frontend preview and backend export path.
- Export starts through the API, runs in the worker through BullMQ, sends status through the API-owned workflow WebSocket channel, stores the PDF artifact, and downloads through REST.
- Export/finalization also generates and stores a CV preview thumbnail artifact for recent CV cards.
- CV draft document and style changes autosave with visible saving/saved/failed/unsaved states.
- Finalized CVs open in a read-only view rather than the editable builder; editing requires duplicating into a new draft.
- Exporting a PDF finalizes automatically in the MVP; old exported PDFs remain downloadable.
- Drafts have a configurable `maxPages` target defaulting to 1; overflow remains visible but blocks export when it exceeds the target.
- Large uploaded/generated artifacts live in object storage.
- Public MVP authentication uses Clerk.
- Domain records use internal user IDs mapped to Clerk identities.
- Internal user IDs are UUIDv7 values; Clerk IDs remain in identity mapping records.
- The NestJS API app is the public backend entrypoint for the MVP; no separate API gateway service is introduced.
- GraphQL is used for application data.
- REST is used for file upload, file download/export, health checks, webhooks, and auth callbacks where appropriate.
- The backend uses NestJS.
- The worker is a separate NestJS application context, not a plain script.
- The API app and worker app do not import from each other.
- API and worker share the application layer and infrastructure adapters.
- The monorepo uses Bun workspaces.
- Packages are layer-based, with internal organization by domain area.
- Application ports are represented as abstract contracts that can be injected by NestJS.
- Generic repository base contracts are allowed, but domain-specific repository methods remain explicit.
- Presenters are used consistently at delivery boundaries.
- User-owned domain data uses DynamoDB single-table design.
- Canonical tags and canonical tag relations use a separate catalog table.
- Canonical tag relation strength starts as curated heuristic data.
- User tags are private guidance and do not appear in exported CVs unless manually typed.
- Imports are future async workflows backed by BullMQ and Redis.
- Kafka, microservices, CQRS, and event sourcing are out of scope for the MVP.
- AI import, AI recommendation, vector search, Qdrant runtime, and agentic editing are out of scope for the MVP but preserved as future architecture directions.

Major modules:

- Identity and user mapping.
- Career context management.
- Experience management.
- Experience tagging and metrics.
- Canonical tag catalog.
- CV draft lifecycle.
- Semantic CV document storage and validation.
- Object storage.
- DynamoDB persistence.
- Worker queue and deletion orchestration.
- GraphQL delivery and presenters.
- REST upload/export delivery.
- Configuration and environment validation.
- Future import/reconciliation foundation.
- Future AI/retrieval contracts.
- Shared CV rendering for preview/export parity.

Deep modules to extract:

- Experience lifecycle module for creating, archiving, splitting, merging, and validating experience invariants.
- CV document module for draft lifecycle, snapshot validation, finalization, and source-reference behavior.
- CV renderer module for semantic document to preview/export HTML rendering.
- Object storage module with a stable interface for local filesystem and S3-compatible storage.
- DynamoDB access module that hides single-table key construction and query patterns.
- Canonical tag module that owns normalization, aliases, relation strengths, and tag lookup.
- Worker orchestration module that owns retryable destructive cleanup and future import jobs.

## Testing Decisions

Good tests should verify external behavior and domain invariants rather than implementation details. They should test what the user or caller can observe: records created, invariants enforced, statuses changed, snapshots preserved, and errors returned.

Test the experience lifecycle module:

- Creating a career context requires at least one experience.
- Experiences cannot exist without a career context.
- One career context can have many experiences.
- Splitting an experience creates target experiences, archives or supersedes the source, and records lineage.
- Merging experiences combines tags, metrics, evidence references, and records lineage.
- Archive removes records from active selection without hard deletion.

Test the CV draft module:

- Drafts can be created as structured blank documents.
- Drafts can be created from selected experiences.
- Draft-local edits do not mutate source experiences.
- Finalized/exported CVs cannot be edited.
- Finalized CVs can be duplicated into editable drafts.
- Source experience changes do not mutate existing CV snapshots.
- Preview fit and export rendering should agree on page count for representative CV documents.

Test frontend workflows:

- Component and form behavior with Vitest and React Testing Library.
- Core browser flows with Playwright E2E: create CV, edit blocks, see fit warnings, export, receive workflow status, and open finalized CV view.
- Defer broad visual regression until the first template stabilizes.

MVP accessibility bar:

- Semantic controls and labels.
- Keyboard-accessible panels and forms.
- Visible focus states.
- Accessible dialogs.
- Warnings that do not rely on color alone.

Test the tag module:

- Canonical tags normalize aliases.
- User tags remain private guidance.
- Experience-tag strength is preserved.
- Canonical and user tags are distinguishable.

Test object storage:

- Large blobs can be written and read by key.
- Missing objects return expected errors.
- Local storage and future S3-compatible implementations satisfy the same contract.

Test DynamoDB persistence:

- Repository behavior matches use case expectations.
- Single-table keys support required MVP access patterns.
- User scoping prevents cross-user reads and writes.

Test GraphQL delivery:

- Resolvers call use cases and return presenter-shaped responses.
- Authenticated user context is required for user-owned operations.
- Validation errors are returned clearly.

Test worker orchestration:

- Hard deletion jobs cascade through related records.
- Failed cleanup can be retried.
- Deletion does not mutate immutable finalized CV artifacts unexpectedly.

Test configuration:

- Required environment variables are validated.
- API and worker can require different config groups while sharing common schema definitions.

There is no existing test prior art in the repo because the repo is still documentation-only.

## Out of Scope

- AI recommendations.
- RAG runtime.
- Qdrant or another vector store in local development.
- AI CV import implementation.
- Agentic CV editing with tool calls.
- Kafka.
- Microservices.
- Event sourcing.
- Full CQRS.
- Arbitrary free-position layout editing.
- Custom CSS templates.
- Public taxonomy ingestion from Kaggle, ESCO, O*NET, or similar sources.
- Deep ATS acceptance analytics.
- Automatic updates to finalized CVs after source experience changes.

## Further Notes

The first MVP should still preserve future architecture for AI-assisted workflows. Imports should eventually use staged suggestions, knowledge-base reconciliation, and a diff-like review before committing changes to canonical experience data.

Future recommendation work should use target profiles for broad retrieval, CV strategies for story-aware ranking, canonical tag relations for structured boosts, and vector search for semantic similarity.

Future export work may introduce a custom CV layout engine if the HTML/print CSS path cannot satisfy preview/export parity, pagination control, or ATS constraints. This is an improvement path, not MVP scope.

Planned future flags:

- `planned-future`: formal WCAG target and deeper screen-reader testing.
- `planned-future`: custom CV layout engine if preview/export parity, pagination control, or ATS constraints require it.
- `planned-future`: dedicated onboarding beyond empty states.
- `planned-future`: offline editing/resilience if the product needs it.
- `planned-future`: AI-assisted import, recommendation, RAG, Qdrant, and agentic editing.

The product should remain careful about trust. Evidence content is immutable, evidence trust/status can change, and generated CV renderings are draft artifacts rather than canonical career facts.
