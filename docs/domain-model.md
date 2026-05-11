# Domain Model

## Core Concepts

### User

The owner of a private career knowledge base.

Public MVP authentication uses Clerk, but domain records use an internal user ID mapped to Clerk identity data.

### Career Context

The place, period, or container where work happened. A career context groups experiences.

Context types:

- `job`
- `personal_project`
- `open_source`
- `education`
- `certification`
- `award`
- `other`

Important fields:

- ID
- User ID
- Type
- Title
- Organization name
- Role
- Start date
- End date
- Current flag
- Location
- Description
- Status

Rules:

- A career context may have many experiences.
- A canonical career context should not be empty in normal user flows.
- Archiving a career context removes child experiences from active selection.
- Hard deletion is explicit, destructive, and worker-orchestrated.

### Experience

The central domain concept. An experience is a fine-grained reusable career claim, accomplishment, responsibility, or project slice that can be selected for a CV.

Experiences are closer to CV bullet material than to full job entries.

Examples:

- Built Golang billing microservices.
- Migrated Express services to Fastify.
- Reduced deployment time by 50% with GitHub Actions.
- Mentored two junior developers.
- Designed a structured rich-text CV editor with SlateJS.

Important fields:

- ID
- User ID
- Career context ID
- Title
- Canonical narrative object key
- Summary preview
- Status
- Tags
- Metrics
- Evidence references
- Created at
- Updated at

Rules:

- Every experience belongs to exactly one career context.
- Experiences are scoped by career context.
- Similar work in two different contexts creates two different experiences.
- Similar work found across multiple imports for the same context should be merged into one experience.
- Canonical narratives are stored as Markdown blobs in object storage.
- DynamoDB stores metadata and indexed fields, not large narrative text.

### Experience Narrative

The long-form source of truth for an experience. It should be detailed, evidence-backed, and useful for future agents.

Store as Markdown in object storage.

Example sections:

- Context
- Actions
- Technologies
- Impact
- Constraints
- Evidence

### Evidence

The original source for a claim. Evidence can come from manual entry, imported CVs, LinkedIn content, portfolio text, or later integrations.

Evidence content is immutable. Evidence status and trust are mutable.

Important fields:

- ID
- User ID
- Source type
- Raw text object key or source reference
- Source file key
- Text span or page reference
- Status: `active`, `rejected`, `superseded`, `deleted`
- Trust level: `unreviewed`, `user_confirmed`, `disputed`

Rules:

- Bad imported evidence should be marked disputed/rejected, not edited in place.
- Experience narratives should be updated through explicit user review.
- Existing CV snapshots do not automatically update when evidence or experiences change.

### Canonical Tag

A global normalized tag used for skills, tools, practices, domains, impact areas, and role signals.

Examples:

- Golang
- Fastify
- GraphQL
- DynamoDB
- CI/CD
- Infrastructure as Code
- Backend Engineering
- Performance Optimization

Canonical tags live in a separate global catalog from user-owned career data.

### Canonical Tag Relation

A product-owned relationship between canonical tags.

Examples:

- `Go` is an alias of `Golang`.
- `DevOps` is strongly related to `CI/CD`.
- `DevOps` is strongly related to `Infrastructure as Code`.
- `Golang` is commonly related to `Backend Engineering`.

Important fields:

- From tag ID
- To tag ID
- Relation type: `alias`, `related`, `parent`, `child`, `prerequisite`
- Strength
- Source: `manual`, `ai_suggested`, `imported_taxonomy`
- Approval status

Initial strength scale:

- `1.0`: alias or equivalent
- `0.9`: strongly related
- `0.7`: commonly related
- `0.4`: weakly related
- `0.1`: rarely relevant

For MVP, seed relations manually. AI-assisted and public-taxonomy expansion can come later.

### User Tag

A private user-defined tag used as guidance or metadata. User tags are not exported into CVs unless the user manually types them.

Examples:

- Do not mention on backend CV.
- Favorite project.
- Needs review.
- Avoid unless asked.

For MVP, user tags may be stored and displayed without deeply influencing recommendations.

### Experience Tag

A first-class relationship between an experience and a tag.

Important fields:

- Experience ID
- Tag ID
- Tag kind: canonical or user
- Strength: `primary`, `supporting`, `incidental`
- Source: `user`, `ai`, `imported`
- Confidence
- Approval status

This relationship supports ranking later. A primary Golang experience should rank higher for a Golang CV than an incidental mention.

### Metric

A measurable outcome connected to an experience.

Examples:

- Reduced deployment time by 50%.
- Supported zero-downtime updates.
- Improved request throughput by 35%.

### Import Batch

A staged workflow created from an uploaded CV or other source document.

Imports are async jobs processed by the worker.

Important fields:

- ID
- User ID
- Source file key
- Extracted text key
- Status
- Created at
- Completed at

### Import Suggestion

An AI-generated or parser-generated suggestion inside an import batch.

Suggestion states:

- `pending`
- `accepted`
- `rejected`
- `edited`
- `merged`

Accepted import suggestions do not become canonical immediately if they conflict with existing data. They should go through knowledge-base reconciliation.

### Knowledge Base Change

A proposed change to the user's canonical career knowledge base.

Examples:

- Create a new experience.
- Add evidence to an existing experience.
- Merge two similar experiences.
- Split an overloaded experience.
- Add a canonical tag relation to an experience.

The long-term import UX should include a diff-like review before committing these changes.

### Experience Lineage Event

A lightweight audit record for structural changes.

Event types:

- `split`
- `merge`

Important fields:

- ID
- User ID
- Type
- Source experience IDs
- Target experience IDs
- Reason
- Created at

This is not full event sourcing. It exists to preserve traceability for split/merge operations.

### Target Profile

The opportunity or role the user is aiming at.

Examples:

- Golang Backend Engineer
- DevOps Engineer
- A pasted job description

Important fields:

- Title
- Source
- Raw input
- Desired canonical tags
- Seniority signals
- Desired domains

### CV Strategy

The positioning plan for one CV built toward a target profile.

Important fields:

- Target profile ID
- Positioning statement
- Emphasized themes
- De-emphasized themes
- Tone
- Length
- Template preference
- Section plan

One target profile can produce multiple strategies.

### CV Draft

A structured CV document assembled from selected experiences and edited by the user.

Important fields:

- ID
- User ID
- Status: `draft`, `finalized`, `exported`, `archived`
- Target profile ID
- CV strategy ID
- Slate document object key
- Page settings
- Source experience IDs
- Created at
- Updated at

Rules:

- Editing CV document text does not mutate source experiences by default.
- Finalized/exported CVs are immutable snapshots.
- To edit a finalized CV, duplicate it into a new draft.
- Exported files are stored in object storage.

## MVP Flow

1. User creates a career context together with at least one experience.
2. User adds more experiences under existing contexts.
3. User tags experiences manually.
4. User creates a CV draft.
5. User manually selects experiences.
6. App creates a structured Slate document from selected experiences.
7. User edits content and controlled styling.
8. User finalizes or exports the CV.

## Future Retrieval Flow

1. User creates a target profile from a role name or job description.
2. System extracts target skills, responsibilities, seniority signals, and keywords.
3. Vector search retrieves semantically related experiences.
4. Canonical tag matching and tag graph expansion boost structured matches.
5. CV strategy reranks experiences for story fit.
6. LLM proposes a strategy and selected experiences.
7. User approves or edits.
8. App creates a CV draft document.

## Data Integrity Rules

- AI may suggest tags, metrics, summaries, splits, merges, and renderings.
- User-approved facts are stored separately from raw AI suggestions.
- Evidence content is immutable, but its status/trust can change.
- Existing CV snapshots do not auto-update when source experiences change.
- Hard deletion is explicit and worker-orchestrated.
- Large text and document blobs belong in object storage, not DynamoDB items.
