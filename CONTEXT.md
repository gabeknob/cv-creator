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
4. User creates a structured blank CV or manually selects experiences for a CV.
5. App creates or opens a structured semantic CV document.
6. User edits structured blocks, rich-text fields, and styling manually.
7. User can export or finalize the CV.

The MVP should not become a generic Google Docs clone. The builder should be structured around resume concepts, even if the final document content is manually editable.

## UX Direction

The authenticated home should behave as a CV workspace, not as a raw database browser.

The experience database remains the source of truth for reusable career knowledge, but the user's primary job-to-be-done is creating, adapting, finding, and exporting CVs. Returning users should be able to quickly find a previous CV, import career material, or start a new CV.

The initial authenticated view should prioritize:

- A discreet global search.
- A prominent Create CV action.
- A secondary Import CV entry point only when import is implemented or clearly marked as future-facing.
- A recent CV list ordered by most recent activity.

Experience database workflows should remain easy to reach, but they are supporting workflows from the user's first interaction perspective.

For the MVP, CV creation is manual and structured. Import CV should not become a fake primary workflow that only stores uploaded files without helping the user turn them into structured experiences.

The MVP must support structured blank CV drafts. A user with no experience database should still be able to start a CV, type manually inside resume-specific sections, tune styling, and export the result. Creating a CV from selected saved experiences remains an important workflow, but it is not mandatory for the first draft.

Structured CV drafts should use a block-based resume model. Users compose CVs from resume-specific blocks such as header, summary, experience section, project section, education section, skills section, certification section, and custom section. This gives users flexibility without turning the editor into an arbitrary freeform document canvas.

CV document blocks should be semantically typed. The stored document should distinguish resume concepts such as header, summary, experience section, experience entry, project section, skills section, education section, certification section, and custom section instead of relying only on generic rich-text paragraphs styled to look like a CV.

When users type experience-like content directly into a CV draft, the app should proactively suggest saving that content into the experience database. These are suggestions only: CV draft edits must not mutate canonical experiences unless the user explicitly confirms the proposed save or link.

For the MVP, save/link suggestions can be rule-based and driven by semantic CV blocks. The suggestion mechanism should remain replaceable because future versions may use AI, vector search, and richer experience matching.

The primary CV builder layout is desktop-first: read-only document preview canvas in the center, supporting panels around it for block outline, experience library or suggestions, styling controls, selected block settings, and fit/overflow feedback. Mobile CV editing is not a first-class MVP workflow before AI-assisted creation exists; smaller screens should degrade safely but do not need full editing parity.

For MVP mobile, users should be able to sign in, view the CV workspace, view finalized CVs, and download PDFs. The full CV builder can require a desktop-sized viewport and should show a clear desktop-required message instead of a degraded editing surface.

The desktop builder should use two tabbed side panels. The left panel owns document navigation and content sources with Outline, Experiences, and Suggestions tabs. The right panel owns document controls with Design, Block, and Fit tabs.

The CV preview canvas must act as a real-time 1:1 preview of the exported PDF. Users edit through structured form-like controls and rich-text fields, not by directly editing the preview canvas. The preview, fit measurement, and export renderer should share the same semantic document model and print layout rules so the user can trust that what they see while editing is what the exported artifact will contain.

CV draft editing should autosave with visible save state. Users should see whether a draft is saving, saved, failed, or has unsaved local changes. Finalization and export remain explicit actions.

The MVP is online-only. Autosave failures should be visible and recoverable through retry, but offline editing/resilience is deferred to the roadmap.

Leaving the builder should be allowed freely when the draft is saved. If autosave is saving, failed, or there are unsaved local changes, the app should warn before navigation.

Finalized CVs should open in a read-only completed-document view, not in the editable builder. The view should offer download/export artifact access, metadata, and a clear duplicate-as-draft action.

For the MVP, exporting a PDF finalizes the CV automatically. Export creates an immutable snapshot and PDF artifact. Users can download old exported PDFs later, and editing a finalized CV requires duplicating the snapshot into a new draft.

CV drafts should have a configurable `maxPages` target, defaulting to 1. The preview should still render overflow pages so users can see what happened, but the Fit panel should show red overflow feedback when the document exceeds `maxPages`. Export is blocked while the document exceeds `maxPages`; the user should receive clear form-style feedback and a toast prompting them to reduce content or increase the page target. If `maxPages` is greater than 1, show a softer warning that one page per roughly ten years of career is widely recommended. Future AI recommendation flows may prefill or adjust `maxPages` based on the user's career length and target.

The builder top bar should always show compact page-fit status such as `1 / 1 pages` or `2 / 1 overflow`. Clicking the status should open the Fit tab, where users can adjust `maxPages`, inspect overflow details, and see export blockers.

The MVP Create CV flow should be lightweight. Before opening the builder, ask for a required CV title, optional target/context, max pages defaulting to 1, and whether the draft should start blank or from selected experiences.

Recent CV cards on the authenticated home should include a visual preview thumbnail, title, target/context when available, status, page/export state, and recent activity date. Keep card actions restrained; the only direct quick action should be downloading the exported PDF when available.

CV preview thumbnails should be generated server-side during export/finalization and stored in object storage alongside PDF artifacts. The frontend should load thumbnail artifacts for recent CV cards and may rely on browser caching rather than rendering mini previews live on the home page.

Draft CV cards do not need live thumbnails in the MVP. Drafts should use a clean placeholder or status treatment until they are exported/finalized.

Opening a CV should depend on status. Drafts open directly in the builder. Finalized/exported CVs open in the read-only completed-document view.

When creating a CV from saved experiences, users may select experiences before entering the builder to seed the draft. The builder's Experiences panel remains the ongoing place to add, remove, or link saved experiences while editing.

The MVP experience database UI should support basic context and experience management, including create, edit, archive, tags, and metrics. Advanced split/merge and reconciliation concepts should remain in the domain model and future UX plans, but their full UI can be deferred.

The experience database should default to a context-grouped view, almost like an unfiltered CV: career contexts are the top-level groups and experiences appear underneath them. An all-experiences toggle/list mode should exist for scanning, filtering, and reuse by individual experience.

Context and experience editing in the MVP should use side sheets from the experience database view. Side sheets keep users oriented in the grouped database while exposing fields such as title, narrative, tags, metrics, status, and archive actions.

Experience database UI can be dense and explicit. Canonical tag chips should visibly show strength as primary, supporting, or incidental. User/private tags remain separate from canonical tags and do not need strength in the MVP.

The builder Experiences panel should use compact selection cards by default, with expandable details for tags, metrics, strength, and narrative preview. CV creation should remain focused even though the underlying experience database is dense.

The builder Suggestions tab should provide deterministic MVP guidance, not AI recommendations. It can show save/link suggestions for manually typed content, missing tags or metadata, page overflow issues, overly long bullets, and other draft hygiene checks.

The builder Design tab should be opinionated. MVP controls should include template preset, paper size, margins, font style category such as serif/sans/mono rather than arbitrary font family, font size, line height, section spacing, and density/compactness.

The builder Block tab should edit the currently selected resume block. It should include common controls such as visibility, move, duplicate, delete, and source links where relevant, plus type-specific controls for blocks such as header, summary, experience entry, skills, education, certification, and custom section.

Users should be able to select blocks from both the read-only preview canvas and the Outline tab. Preview selection should highlight semantic blocks and open the Block tab without making preview text directly editable.

Block reordering should happen through Outline drag-and-drop and Block tab move controls. The preview canvas should remain selectable and stable rather than becoming a drag-and-drop editing surface.

The MVP should model CV templates but implement one strong ATS-friendly template first. Additional templates can be added later after preview/export parity and fit behavior are stable.

If final export rendering disagrees with Pretext/page-fit estimation and exceeds `maxPages`, the user should be offered a clear choice: return to the builder or increase `maxPages` and export. The system should be battle-tested so this fallback is rare; preview/export parity is a quality requirement.

Frontend testing should use Vitest and React Testing Library for components/forms, plus Playwright E2E for core CV workflows such as creating a draft, editing blocks, fit warnings, export status updates, and finalized CV views. Visual regression can be added later once the first template stabilizes.

The MVP should enforce an explicit accessibility bar: semantic controls, labels, keyboard-accessible panels/forms, visible focus states, accessible dialogs, and warnings that do not rely on color alone. A fuller accessibility program with formal WCAG targets and deeper screen-reader testing belongs on the longer-term roadmap.

The MVP should not include a dedicated onboarding flow. After signup, users land on the CV workspace and empty states guide them toward creating a CV or maintaining experiences. Onboarding can be revisited on the roadmap.

The empty authenticated home should primarily prompt users to create their first CV. It should also offer a quieter secondary path to the experience database so users understand they can manage reusable career material separately.

The MVP should include a very minimal settings page for profile/contact defaults that can seed CV headers. Clerk owns authentication account management, so settings should avoid broader account-management scope unless needed.

Profile/contact defaults should be copied into new CV drafts at creation time. Existing drafts and finalized CVs should not update automatically when settings change.

Motion should be functional and restrained. Use animation to clarify panel transitions, block reordering, modals, drawers, status changes, and suggestion appearance or dismissal. The CV canvas should remain a precise editing surface rather than a decorative animated space.

Authenticated navigation should be hybrid. The home and CV builder should stay focused on the CV workspace rather than a heavy persistent sidebar. Sidebars may be used on settings, admin, or deeper management pages where persistent section navigation helps.

Global search should be unified across the user's CV and career knowledge base. The MVP should search CV drafts/finalized CVs, career contexts, experiences, and relevant tags from one discreet search entry point. Future versions can expand unified search to imports, evidence, AI actions, and recommendations.

Global search should behave like a command palette: grouped keyboard-navigable results with direct navigation. CV drafts open the builder, finalized/exported CVs open the read-only view, experiences open the database with the experience side sheet, and contexts open the matching group in the experience database.

The experience database should exist both as a dedicated management workflow and as a lightweight builder panel. The dedicated experience database page is for maintaining career contexts, experiences, tags, and metrics. The builder experience panel is for finding, selecting, linking, or saving experiences in the active CV.

The public MVP should include a very short landing page before authentication. Its job is to explain the product idea briefly and route users to sign in or sign up, not to act as a full marketing site.

Application UI styling and CV document styling are separate concerns. The product interface should use a shadcn/ui-generated base theme for app chrome such as panels, buttons, dialogs, forms, navigation, and settings. Exported CV documents and builder previews should use CV-specific style settings controlled by the CV styling panel, not the application theme.

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
- Manually typed CV content may trigger suggestions to save or link experiences, but the user must confirm before canonical data changes.
- Finished/exported CVs are immutable snapshots.
- AI output is a suggestion until the user confirms it.
- Imported evidence can be disputed or rejected without editing the original source text.
- Canonical experience narratives should be rich enough for future agents to reason over.
- User-facing workflows should stay simple even when the backend model is explicit.

## Initial Stack Direction

- Runtime/package manager: Bun workspaces.
- Frontend: Vite, React, TanStack Router, SlateJS-powered rich text fields.
- Backend: NestJS.
- API style: GraphQL for application data, REST for file upload/export/health.
- Public backend entrypoint: NestJS API app; no separate API gateway service in the MVP.
- Architecture: Clean Architecture with layer-based packages.
- Primary database: DynamoDB single-table design for user-owned app data.
- Canonical tag catalog: separate DynamoDB table.
- Local database: DynamoDB Local.
- Auth: Clerk for public MVP, mapped to internal UUIDv7 user IDs.
- Async jobs: BullMQ with Redis.
- Worker: separate NestJS worker app.
- Object storage: local filesystem adapter first, S3-compatible adapter later.
- PDF export: server-side Playwright/Chromium rendering from semantic HTML and print CSS.
- Workflow status: API relays long-running workflow updates through authenticated WebSockets; PDF export is the first producer.
- AI/retrieval: documented and modeled for future work, not required for first MVP.

## Open Questions

- What is the exact DynamoDB single-table access pattern list?
- What is the minimum useful canonical tag seed catalog?
- What styling controls are included in the MVP builder?
- How much of split/merge should be in the first UI versus only the domain model?
