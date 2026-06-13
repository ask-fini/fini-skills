# Source Ingestion And Refresh

Use this reference for source import, refresh, polling, changed-source detection, and source deletion. Fetch current docs first for exact request fields and supported source types.

## Web Import

1. Decide whether seed URLs should be crawled or used directly.
2. If crawling, call the crawl flow and let the user or caller choose URLs to ingest.
3. Ingest web URLs.
4. Poll source status until terminal.
5. Inspect failures before moving to knowledge generation.
6. Hand successful source IDs to `fini-api-knowledge`.

Key mistake: using source IDs instead of URLs for web `documentIdsToAdd`.

## Provider Import

1. Confirm provider type and connected account state.
2. List importable provider resources.
3. Register selected resources to create/update Fini source records.
4. Ingest returned source IDs.
5. Poll source status.
6. Hand successful source IDs to `fini-api-knowledge`.

Key mistake: skipping registration and trying to ingest provider-native IDs directly.

Provider discovery is for new importable content. If the user asks what is already in Fini, use the source-listing route with provider/source filters instead.

## Refresh Existing Sources

1. Refresh known source IDs.
2. Poll each refreshed source.
3. List sources with changed-source filters.
4. Keep changed sources that are linked to existing knowledge when updating live content.
5. Bulk-generate knowledge from changed source IDs, usually as drafts.
6. Poll knowledge jobs and review/publish through `fini-api-knowledge`.

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
- Changed flag.
- Extracted paragraphs/content signal when available.

Stop polling when the docs-defined terminal status appears, or when a bounded timeout is reached and the source should be reported as stuck.

## Delete Guardrails

Before deleting sources, clarify:

- Exact source IDs.
- Whether linked articles should remain.
- Whether `deleteArticles` or equivalent behavior is intended.
- Whether source deletion alone satisfies the business goal.

Never infer linked article deletion from "delete source" unless the user explicitly asks for it.
