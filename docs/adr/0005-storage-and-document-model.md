# ADR 0005: Storage and Document Model

## Status

Accepted

## Context

The app stores large and user-editable content: uploaded CVs, extracted text, canonical experience narratives, Slate CV documents, and exported PDFs. DynamoDB has a 400 KB item size limit, so large blobs should not be stored directly in table items.

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
- Slate CV document snapshots as JSON.
- Exported PDFs and future export formats.

Local development uses a filesystem-backed object storage adapter. Production can use an S3-compatible adapter.

## CV Document Format

Use SlateJS-compatible JSON for CV document snapshots.

The backend stores a versioned wrapper:

```ts
type CvDocumentSnapshot = {
  schemaVersion: number;
  editor: "slate";
  content: unknown[];
  pageSettings: {
    size: "A4" | "Letter";
    margins: {
      top: number;
      right: number;
      bottom: number;
      left: number;
    };
    lineHeight: number;
    fontFamily: string;
    fontSize: number;
  };
};
```

The backend validates the document lightly:

- Valid JSON.
- Known schema version.
- `content` is an array.
- Page settings are within sane bounds.
- Serialized size is acceptable.
- No unsafe embedded content.

## Page Fit Estimation

Use Pretext in the MVP to estimate wrapped text height for CV document blocks.

The estimator should account for:

- Page size such as A4 or Letter.
- Margins.
- Template spacing.
- Section headings.
- Bullet indentation and list markers.
- Font family.
- Font size.
- Line height.

Pretext is used as a fast preflight estimator, not as the final export renderer. It helps the editor warn about overflow, estimate remaining page space, and guide users toward a CV that fits the selected format without repeatedly forcing DOM layout measurement.

The final renderer remains the authority before export/finalization. If the final PDF renderer reports a different page count or overflow state, the renderer result wins.

## Experience Narrative Format

Use Markdown for canonical experience narratives.

Markdown is:

- Portable.
- Diffable.
- Easy for humans to review.
- Easy for agents and LLMs to consume.
- Less tied to a frontend editor than Slate JSON.

## Immutability Rules

- Draft CV documents are editable.
- Finalized/exported CVs are immutable snapshots.
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
