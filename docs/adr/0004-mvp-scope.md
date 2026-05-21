# ADR 0004: MVP Scope

## Status

Accepted

## Context

The long-term product includes AI import, graph-like tag relationships, vector retrieval, RAG, and CV generation agents. The first MVP should be smaller and useful for personal/manual CV creation while preserving the foundation for future intelligent features.

## Decision

The MVP focuses on manual structured CV creation, with reusable experiences as an important source but not a prerequisite for creating the first CV.

In scope:

- Clerk authentication for public MVP.
- Internal user IDs mapped to Clerk identities.
- Career context and experience creation.
- No orphan experiences.
- Manual canonical and user tags.
- Experience selection for CV drafts.
- Structured blank CV draft creation.
- Block-based resume composition.
- Structured CV builder with SlateJS-powered rich text fields.
- Read-only 1:1 PDF preview canvas.
- Controlled styling: template, font, font size, margins, line height, section order, show/hide sections.
- Pretext-based CV fit estimation for page size, margins, font size, and line height.
- Draft-local editing.
- Finalized/exported CV immutability.
- Object storage for document snapshots and exported files.
- Redis/BullMQ worker for async imports and deletion orchestration where needed.

Out of scope for first MVP:

- AI recommendations.
- RAG/vector search runtime.
- Qdrant local infrastructure.
- Agentic document editing.
- Kafka.
- Microservices.
- Event sourcing.
- Full CQRS.
- Arbitrary free-position layout editing.

## Rationale

The app should not become a worse Google Docs clone. Even without recommendations, the core long-term value is reusable structured experiences that can be selected and adapted into different CVs. For the MVP, however, a user must be able to start from a structured blank CV because AI import and recommendations are not available yet. The CV preview is read-only; editing happens through structured controls and rich-text fields.

The MVP should prove:

- Users can maintain an experience database.
- Users can create a useful CV before fully populating their experience database.
- Fine-grained experiences are useful when assembling CVs.
- CV document edits can diverge safely from canonical experience data.
- The editor can help users stay within a practical CV page budget.
- The architecture can later support AI recommendations.

## Consequences

Benefits:

- Smaller first product.
- Still validates the unique idea.
- Avoids carrying unused Qdrant/AI infrastructure before a feature needs it.
- Keeps backend architecture meaningful from day one.

Tradeoffs:

- The first MVP is less "smart" than the long-term vision.
- Recommendation and import workflows remain design foundations until later slices.
- Manual experience entry must be ergonomic because it is the first core workflow.
- Blank CV editing must stay resume-structured so the product does not collapse into a generic document editor. The MVP uses resume-specific blocks rather than arbitrary page content.

## UX Shape

Main MVP flows:

```txt
Create experience database:
1. Create context and first experience together.
2. Add more experiences to existing contexts.
3. Add tags and metrics.

Create CV:
1. Create CV draft.
2. Start from a structured blank document or select saved experiences.
3. Generate or open the initial semantic CV document.
4. Add, remove, reorder, and edit resume-specific blocks through structured controls.
5. Show fit/overflow feedback for the selected page format.
6. Finalize/export.
```
