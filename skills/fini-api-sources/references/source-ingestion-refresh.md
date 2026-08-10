# Source Ingestion And Refresh

Use this reference for source import, refresh, polling, changed-source detection, and source deletion. Fetch current docs first for exact request fields and supported source types.

## Contents

- [Source Lifecycle](#source-lifecycle)
- [Exact Source Inspection](#exact-source-inspection)
- [Source Listing And Pagination](#source-listing-and-pagination)
- [Web Import](#web-import)
- [Provider Import](#provider-import)
- [Refresh Existing Sources](#refresh-existing-sources)
- [Polling Checklist](#polling-checklist)
- [Delete Guardrails](#delete-guardrails)

## Source Lifecycle

Sources are raw content records. They do not become live bot knowledge by themselves.

Lifecycle:

1. Discover or choose source inputs.
2. Register provider resources when needed.
3. Ingest or refresh source records.
4. For provider registration, send `addedDocumentIds` to `documentIdsToAdd` and `updatedDocumentIds` to `documentIdsToRefresh`.
5. For the dedicated refresh route, preflight exact source records and separate uploaded-file targets whose returned wire value is `source="files"`.
6. Poll exact known source IDs through Get source; use a filtered List sources query for cohorts.
7. Detect changed linked sources when updating existing content.
8. Hand processed source IDs to Knowledge generation.
9. Publish and assign resulting knowledge before expecting answer changes.

If content is already current but behavior is wrong, route prompt/rule/tag issues to Agent Configuration and configured Attribute, Data Step, or Action failures to Actions/Attributes rather than repeating source refresh.

## Exact Source Inspection

Use Get source when a source ID is already known. It returns the same source record shape as List sources and avoids scanning unrelated records.

For a focused audit or poll, preserve:

- source ID, source/provider type, original/provider URL, title, and external ID
- latest job ID/status, `success`, and `error`
- linked knowledge ID, operation, and reason
- `changed`, creation time, and update time
- MIME/provider subtype when relevant

Large `paragraphs` and `oldParagraphs` arrays can be useful for change diagnosis, but do not dump them by default. Summarize or inspect only what the task requires.

## Source Listing And Pagination

List sources supports filters such as source type, success, ID, title, URL, exact web hostname, linked article, linked/unlinked state, linked operation, and changed state. Check current docs for exact parameter shapes.

The response includes `hasMore` but no `nextCursor`. Direction is relative to the selected sort order:

- use the last returned source ID with `direction=next` to continue in the selected `order`
- use the first returned source ID with `direction=previous` to move against the selected `order`
- with default `order=desc`, `next` moves older; with `order=asc`, `next` moves newer
- treat source IDs as opaque cursors and preserve the boundary ID in resumable exports

## Web Import

1. Decide whether seed URLs should be crawled or used directly.
2. If crawling, call the crawl flow and let the user or caller choose URLs to ingest.
3. For first-time web records, call Ingest Sources with `source=web` and URLs in `documentIdsToAdd`.
4. Poll source status until terminal.
5. Inspect failures before moving to knowledge generation.
6. Hand successful source IDs to `fini-api-knowledge`.

Key mistake: using source IDs instead of URLs in the Ingest Sources web arrays. The dedicated Refresh Sources route is different: it accepts existing Fini source IDs in `documentIds`, including for web sources.

## Provider Import

1. Confirm provider type and connected account state.
2. List importable provider resources.
3. Register selected resources to create/update Fini source records.
4. Put registration `addedDocumentIds` into Ingest Sources `documentIdsToAdd`.
5. Put registration `updatedDocumentIds` into Ingest Sources `documentIdsToRefresh`.
6. Poll source status.
7. Hand successful source IDs to `fini-api-knowledge`.

Key mistake: skipping registration and trying to ingest provider-native IDs directly.

Provider discovery is for new importable content. If the user asks what is already in Fini, use the source-listing route with provider/source filters instead.

The current public provider discovery and registration routes support Notion, Zendesk, and Confluence. `googledrive` is supported by List/Get/Ingest for already registered source IDs, but no public Google Drive discovery or registration route is documented. A source type appearing in read results does not by itself prove that provider discovery supports importing it.

## Refresh Existing Sources

1. Fetch the exact target source records before queueing refresh.
2. Separate any uploaded-file records whose returned `source` is `files`. The dedicated route rejects them. No public API or MCP route uploads or replaces that source-record type; a separate CSV knowledge-tree helper does not replace a file source. Require an explicit dashboard/manual re-upload plan before considering deletion.
3. Send only refreshable existing Fini source IDs in the dedicated route's `documentIds`. Do not pass web URLs here.
4. Poll each known source ID through Get source and interpret job status together with `success` and `error`.
5. Preserve the targeted source IDs and their pre-generation `changed` state.
6. For a broad cohort, List sources with `changed=true` and linked-knowledge filters. For a small known set, use the exact-source results.
7. Keep changed sources that are linked to existing knowledge when updating live content.
8. Bulk-generate knowledge from changed source IDs, usually as drafts.
9. Poll knowledge jobs and review/publish through `fini-api-knowledge`.

A successful source-backed generate-and-save operation clears the source's `changed` flag, so record the changed cohort before generation when auditability matters.

Customer phrase translation:

- "Update the bot from changed docs" means refresh sources, find changed linked sources, generate draft knowledge, review/publish, and verify assignment.
- "Sync only what changed" means do not bulk-generate from every source unless the user explicitly accepts that broader update.
- "Train the bot" means continue into `fini-api-knowledge`; completed ingestion alone does not change answers.

## Polling Checklist

For each source, inspect:

- Processing status.
- Success boolean or equivalent completion field.
- Error field.
- Linked knowledge ID.
- Linked operation and reason.
- Linked job ID.
- Changed flag.
- Extracted paragraphs/content signal when available.

Stop polling when the docs-defined terminal status appears, or when a bounded timeout is reached and the source should be reported as stuck.

Do not interpret `linkedJobStatus=COMPLETED` in isolation. A completed job can still be unusable for the requested next step when `success` is false, `error` is populated, linkage is absent, or the operation indicates no knowledge change.

## Delete Guardrails

Before deleting sources, clarify:

- Exact source IDs.
- Whether linked articles should remain.
- Whether `deleteArticles` or equivalent behavior is intended.
- Whether source deletion alone satisfies the business goal.

Never infer linked article deletion from "delete source" unless the user explicitly asks for it.

When replacing an uploaded-file source, preserve the old source ID and linked-article impact. Source deletion is a destructive public API write; replacement upload must happen in the Fini dashboard or another separately verified non-public surface. Do not delete until that plan is explicit, and do not include ordinary refreshable sources in the delete batch.
