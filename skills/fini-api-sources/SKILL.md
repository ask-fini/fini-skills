---
name: fini-api-sources
description: Use when the user wants to import, crawl, register, ingest, refresh, list, filter, inspect, poll, or delete Fini sources through the public API; inspect one exact source and its job, change, linkage, or error metadata; train a Fini bot from help-center articles, internal docs, websites, Notion, Zendesk, or Confluence; inspect already registered Google Drive or uploaded-file sources; sync only changed linked content; distinguish provider discovery from already imported sources; or hand processed source IDs into knowledge generation.
---

# Fini API Sources

Use this skill for raw content source workflows. Sources feed knowledge; they are not the live knowledge layer.

Before endpoint-specific work, fetch `https://docs.usefini.com/llms.txt` and the relevant Sources docs page.

## Workflow

1. Identify the intent: discover/import, list/filter, inspect one exact source, refresh/poll, or delete.
2. When an exact source ID is known, use Get source for focused inspection and polling. Use List sources for discovery, filtered cohorts, or pagination.
3. For first-time web imports, optionally crawl seed URLs, then call Ingest Sources with selected URLs in `documentIdsToAdd`. For existing web records, prefer the dedicated Refresh Sources route with their Fini source IDs in `documentIds`.
4. For provider imports, list and register resources. Send registration `addedDocumentIds` to Ingest Sources as `documentIdsToAdd`, and `updatedDocumentIds` as `documentIdsToRefresh`.
5. Before the dedicated refresh route, fetch the exact targets and separate uploaded-file sources whose returned wire value is `source="files"`. The route cannot refresh them. No public API or MCP route uploads or replaces that source-record type; the separate CSV knowledge-tree helper is not a file-source replacement. Do not delete one until the user has an explicit dashboard/manual replacement-upload plan and accepts the linked-article impact. Do not send a mixed batch that can fail as a unit.
6. Poll each known source after ingestion or refresh. Verify terminal job status together with `success` and `error`; do not rely on one field alone.
7. For refresh workflows, preserve the targeted IDs, identify sources that are both changed and linked to knowledge, and hand only that set to generation unless broader regeneration was requested.
8. For a fresh or empty workspace, verify a usable folder tree exists before source-backed generation.
9. Hand off processed source IDs to `fini-api-knowledge` for generation, drafts, publishing, and assignment.
10. For delete, produce an exact source/article impact plan and require confirmation.

## Defaults

- Treat ingestion and refresh as async.
- Prefer Get source when polling or auditing a known source ID; it returns the same source shape as a list item without a broad scan.
- For List sources pagination, `next` continues in the selected `order` and `previous` walks against it. Use the last returned source ID for `next` and the first for `previous`. With default `order=desc`, next moves older; with `order=asc`, next moves newer. The response has `hasMore` but no `nextCursor`.
- Prefer reviewable generated drafts after source import or refresh.
- Do not bulk-generate from sources in an empty workspace until `fini-api-knowledge` has created or initialized folders.
- For refresh, generate knowledge only for sources that actually changed and already link to knowledge, unless the user explicitly wants broader behavior.
- Preserve the changed-source cohort before generation because a successful source-backed generate-and-save operation clears `changed`.
- Do not promise answer changes until knowledge generation, publish/review, and assignment are verified.
- Translate "train the bot" or "sync docs" into source processing plus a knowledge handoff; source import alone is not enough.
- If source content is current, route wrong prompts/routing/escalation/classification to `fini-api-agent-configuration`; route missing dynamic context, Data Step failures, or Action failures to `fini-api-actions-attributes` instead of refreshing sources again.

## Gotchas

- The Ingest Sources contract uses `documentIdsToAdd` and `documentIdsToRefresh`; the product concept is "sources."
- On Ingest Sources with `source=web`, both arrays contain URLs. That is distinct from the dedicated Refresh Sources route, whose `documentIds` are existing Fini source IDs, including for web records.
- For providers, registration returns `addedDocumentIds` and `updatedDocumentIds`. Send new IDs to `documentIdsToAdd` and existing updated IDs to `documentIdsToRefresh`.
- The dedicated Refresh Sources route rejects uploaded-file records whose wire value is `source="files"`; `raw-file` is descriptive docs prose, not the List Sources enum. Preflight mixed targets. A replacement upload requires the Fini dashboard or another separately verified non-public surface, so never delete first without that plan.
- A queued ingestion response is not completion.
- `linkedJobStatus=COMPLETED` is not sufficient by itself; inspect `success`, `error`, `linkedKnowledgeId`, `linkedOperation`, and `linkedReason` for the intended workflow.
- Source list/get responses also carry identity and processing metadata such as original/provider URL, external ID, MIME/provider type, job ID, paragraph snapshots, and timestamps. Preserve the fields needed for auditability without dumping large `paragraphs` payloads by default.
- Completed source processing does not mean live agent answers changed.
- Completed source processing also does not mean knowledge generation can succeed; fresh workspaces need a usable folder tree first.
- Provider discovery may exclude already-imported resources.
- Provider discovery requires the provider to be connected in the workspace first.
- The public provider discovery/registration workflow currently supports Notion, Zendesk, and Confluence; check current docs before claiming another provider is importable through those routes. `googledrive` is a supported List/Get/Ingest source value for already registered IDs, but that does not expose public Google Drive discovery or registration.
- Discovery is for new importable provider content; list sources for content already imported into Fini.
- Source deletion can optionally delete linked articles; treat this as destructive.

## Proof Of Completion

- Import: source records exist and status fields show completion or failure.
- Exact-source inspection: report the source ID, type/origin, current processing state, job/linkage metadata, changed state, and error without requiring a broad list query.
- Polling: terminal status reached, with `success`, `error`, job ID/status, and linkage interpreted together.
- Refresh: targeted IDs and changed linked sources are identified before generation, and unchanged/unlinked sources are explicitly excluded or explained.
- Paginated list: report filters, number of pages and sources, `hasMore`, and the last/first source ID used as the cursor.
- Delete plan: source IDs, linked article impact, `deleteArticles` behavior, and confirmation status are explicit.
- Knowledge handoff: identify which source IDs should feed bulk generation.

## References

Read [source-ingestion-refresh.md](references/source-ingestion-refresh.md) for exact-source inspection, list pagination, web/provider import, refresh, changed-source detection, polling, and deletion guardrails.
