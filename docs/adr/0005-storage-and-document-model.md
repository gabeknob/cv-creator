# ADR 0005: Storage and Document Model

## Status

Accepted

## Context

The app stores large and user-editable content: uploaded CVs, extracted text, canonical experience narratives, semantic CV documents, and exported PDFs. DynamoDB has a 400 KB item size limit, so large blobs should not be stored directly in table items.

## Decision

Use DynamoDB for metadata and indexed fields. Use object storage for large blobs and generated artifacts.

DynamoDB stores:

- IDs
- Ownership fields
- Status fields
- Sort/query keys
- Summary previews
- Object storage keys
- Tag relationships
- Metric metadata
- Draft metadata

Object storage stores:

- Uploaded CV files.
- Extracted text files.
- Canonical experience narratives as Markdown.
- Semantic CV document snapshots as JSON.
- Exported PDFs and future export formats.
- Generated CV preview thumbnails.

Local development uses a filesystem-backed object storage adapter. Production can use an S3-compatible adapter.

## CV Document Format

Use versioned semantic CV JSON for CV document snapshots.

CV documents use semantic resume block types as the document model. The document should preserve whether a block is a header, summary section, experience section, experience entry, project section, skills section, education section, certification section, or custom section instead of storing only generic paragraphs and headings.

SlateJS-compatible JSON is used for rich text field values inside semantic blocks. Slate is not the source of the whole document layout and the preview canvas is not directly edited as a Slate document.

The backend stores a versioned wrapper:

```ts
type CvDocumentSnapshot = {
  schemaVersion: number;
  documentModel: "semantic-cv";
  richTextFormat: "slate";
  templateId: string;
  blocks: unknown[];
  pageSettings: {
    size: "A4" | "Letter";
    margins: {
      top: number;
      right: number;
      bottom: number;
      left: number;
    };
    lineHeight: number;
    fontStyle: "serif" | "sans" | "mono";
    fontSize: number;
    density: "compact" | "standard" | "comfortable";
  };
};
```

The backend validates the document lightly:

- Valid JSON.
- Known schema version.
- `blocks` is an array.
- Top-level CV blocks use known semantic resume block types.
- Rich text fields use the supported Slate-compatible value shape.
- Page settings are within sane bounds.
- Serialized size is acceptable.
- No unsafe embedded content.

## Page Fit Estimation

Use Pretext in the MVP to estimate wrapped text height for CV document blocks.

The estimator should account for:

- Page size such as A4 or Letter.
- Max pages target.
- Margins.
- Template spacing.
- Section headings.
- Bullet indentation and list markers.
- Font family.
- Font size.
- Line height.

Pretext is used as a fast preflight estimator, not as the final export renderer. It helps the editor warn about overflow, estimate remaining page space, and guide users toward a CV that fits the selected format without repeatedly forcing DOM layout measurement.

CV drafts have a configurable `maxPages` target, defaulting to 1. The preview should still render overflow pages so the user can understand the layout, but export is blocked if the document exceeds `maxPages`. A `maxPages` value above 1 should show softer guidance that one page per roughly ten years of career is commonly recommended.

The final renderer remains the authority before export/finalization. If the final PDF renderer reports a different page count or overflow state, the renderer result wins.

## PDF Export Rendering

Use server-side Playwright/Chromium rendering for MVP PDF export.

The export pipeline should be worker-backed from the MVP. The API starts an export through a REST endpoint, enqueues a BullMQ job, and relays job status through the authenticated workflow status WebSocket channel. The worker renders the semantic CV document to HTML with print CSS, then uses Chromium's PDF output to create the artifact. Export must preserve real selectable text; it must not render the CV as screenshots or image-only pages.

The CV HTML should remain ATS-friendly:

- DOM order should match the intended reading order.
- Core text should use normal text nodes, headings, sections, articles, paragraphs, and lists.
- Avoid image-based text.
- Avoid absolute positioning for core resume content.
- Avoid complex table-heavy or multi-column layouts in the MVP.
- Use opinionated CV templates with simple print CSS.

The frontend preview and server export should both consume the same semantic CV document model. Pretext provides fast editing-time fit estimation; Playwright/Chromium remains the final export authority.

Slate rich text fields should be rendered into semantic inline HTML such as `strong`, `em`, and `a` by a shared renderer/serializer. The preview and export paths should use the same rich-text serialization rules instead of mounting a Slate editor in the preview.

The shared rendering rules should live in a TypeScript package, initially `packages/cv-renderer`. The frontend uses it for the real-time preview canvas, and the backend/export path uses it to generate the semantic HTML passed to Playwright/Chromium. This package is the parity boundary between preview and export.

The worker should emit workflow progress states such as started, rendering, storing, completed, and failed. The frontend should present export as an active operation rather than as a passive queued background task.

Export/finalization should generate a preview thumbnail artifact alongside the PDF and store it in object storage. Recent CV cards load stored thumbnails instead of rendering miniature CV previews live.

## Future Layout Engine

A custom CV layout engine is a future improvement path if semantic HTML, constrained print CSS, Pretext estimation, and Playwright/Chromium export do not provide enough control over preview/export parity, pagination, or ATS behavior.

That future engine would compute page layout from the semantic CV document first, then feed both the preview renderer and PDF renderer from the same computed layout. It is deliberately out of MVP scope because it would require owning text measurement, wrapping, rich-text inline layout, pagination, page breaks, and PDF drawing behavior.

## Experience Narrative Format

Use Markdown for canonical experience narratives.

Markdown is:

- Portable.
- Diffable.
- Easy for humans to review.
- Easy for agents and LLMs to consume.
- Less tied to the CV builder's rich text field implementation.

## Immutability Rules

- Draft CV documents are editable.
- Draft CV document and style changes autosave with visible save state.
- Finalized/exported CVs are immutable snapshots.
- Exporting a PDF finalizes the CV automatically in the MVP.
- Old exported PDF artifacts remain downloadable.
- User-facing MVP export is PDF.
- The finalized semantic CV JSON snapshot is stored internally for exact reconstruction, audit, and export recovery.
- To edit a finalized CV, duplicate it into a new draft.
- CV document edits do not mutate source experiences by default.
- Evidence content is immutable, but evidence status/trust can change.

## Deletion Rules

Archive is the normal removal path.

Hard deletion is explicit and destructive. Hard-deleting a career context cascades to:

- Experiences.
- Experience-tag relationships.
- Metrics.
- Evidence links.
- Object storage blobs owned only by deleted records.
- Future vector index entries.

Cascade deletion is worker-orchestrated so cleanup can be retried.

## Consequences

Benefits:

- Avoids DynamoDB item size pressure.
- Keeps large documents and exports out of the database.
- Supports future S3 migration.
- Keeps final CVs stable after source data changes.
- Gives the MVP a practical mechanism for A4/Letter fit feedback while editing.

Tradeoffs:

- Requires object storage adapter from the beginning.
- Reads may need both DynamoDB and object storage.
- Document snapshot versioning must be handled deliberately.
- Text measurement accuracy depends on keeping Pretext inputs aligned with loaded fonts and rendered CV styles.
