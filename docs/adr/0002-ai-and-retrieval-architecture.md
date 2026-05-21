# ADR 0002: AI and Retrieval Architecture

## Status

Proposed

## Context

AI is central to the long-term product, but it is not required for the first MVP. The app should eventually import existing CVs, parse experiences, suggest tags, detect duplicates, recommend relevant experiences for a target role, create a CV strategy, and generate adapted CV drafts.

The product must avoid allowing generated text to become ungrounded or factually inaccurate. A CV tool must preserve trust because users rely on it to represent their career accurately.

## Decision

Use structured storage as the source of truth and AI as an assistant layer.

DynamoDB stores metadata and relationships for:

- User-approved career contexts.
- User-approved experiences.
- Tags and experience-tag relationships.
- Metrics.
- Evidence metadata.
- Import batches and suggestions.
- Target profiles.
- CV strategies.
- CV drafts.
- AI job status and suggestion state.

Object storage stores large blobs:

- Uploaded source files.
- Extracted text.
- Experience canonical narratives as Markdown.
- Semantic CV document snapshots with Slate-compatible rich text fields.
- Exported PDFs or other generated files.

Vector search is planned but deferred until recommendation features begin. When introduced, it should store embeddings for:

- Experience canonical narratives.
- Experience summaries.
- Target profiles and job descriptions.
- Canonical tag profile documents.

LLMs are planned for:

- Extracting structured experiences from imported CV text.
- Suggesting tags, metrics, and competencies.
- Detecting duplicate or overloaded experiences.
- Proposing split/merge operations.
- Creating a CV strategy.
- Ranking experiences against a target profile and CV strategy.
- Rendering selected experiences into a cohesive CV story.

## Rationale

RAG is useful for semantic matching, but it should not replace the application database. A vector database is optimized for similarity search, not for being the canonical source of user data.

The safest long-term architecture is:

1. Store canonical user facts and relationships in structured records.
2. Store large text/document artifacts in object storage.
3. Embed approved records for retrieval when recommendation features exist.
4. Retrieve relevant records for a target role.
5. Use canonical tag relationships to boost exact and related matches.
6. Ask an LLM to plan, rank, explain, and render based only on retrieved records.
7. Let the user review final output.

## Import Workflow

Future AI import:

1. User uploads or pastes a CV.
2. API stores the source file in object storage.
3. API creates an import batch and enqueues a BullMQ job.
4. Worker extracts text.
5. LLM suggests experiences, tags, metrics, evidence, and possible duplicate/split/merge operations.
6. User triages staged suggestions.
7. System builds a proposed knowledge-base diff.
8. User reviews the diff.
9. Approved changes update canonical records.
10. Embeddings are generated later when retrieval is enabled.

## CV Generation Workflow

Future AI-assisted CV generation:

1. User creates a target profile from a prompt or job description.
2. System retrieves broad candidates from experiences.
3. System expands desired tags through canonical tag relations.
4. User reviews or edits a CV strategy.
5. LLM uses the strategy to select and render experiences as a cohesive story.
6. App creates a semantic CV draft document.
7. User manually edits the final document.

CV generation can remain synchronous for the initial non-recommendation MVP. If generation later becomes slow or unreliable, it can move behind an async job without changing the use case boundary.

## Consequences

Benefits:

- AI features remain grounded in approved data.
- The architecture supports RAG without requiring it immediately.
- Import review can handle multiple historical CVs and duplicate detection.
- Canonical tag relationships can improve ranking without requiring a graph database on day one.
- The system can later add agents that edit drafts via controlled operations.

Tradeoffs:

- Requires careful prompt/schema design later.
- Requires UX for triage, reconciliation, and correction.
- Requires eventual embedding and vector index lifecycle management.
- Requires clear distinction between evidence, canonical narratives, and generated CV renderings.

## Follow-ups

- Define AI extraction output schemas before implementing import.
- Define CV strategy schema.
- Define the canonical tag seed catalog.
- Decide the first vector store when recommendations begin.
- Decide the first embedding model when recommendations begin.
- Design the late-correction UX for disputed evidence.
