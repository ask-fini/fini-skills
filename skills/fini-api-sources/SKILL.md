---
name: fini-api-sources
description: Use when the user wants to import, crawl, register, ingest, refresh, list, inspect, poll, detect changed content for, or delete Fini sources through the public API; train a Fini bot from help center articles, internal docs, website pages, Notion, Zendesk, or Confluence; sync changed source content; or hand processed sources into knowledge generation.
---

# Fini API Sources

Use this skill for raw content source workflows. Sources feed knowledge; they are not the live knowledge layer.

Before endpoint-specific work, fetch `https://docs.usefini.com/llms.txt` and the relevant Sources docs page.

## Workflow

1. Identify source type: web URL, connected provider resource, existing source refresh, or source deletion.
2. For web imports, optionally crawl seed URLs first, then ingest selected URLs.
3. For provider imports, list provider resources, register chosen resources, then ingest returned source IDs.
4. Poll source status after ingestion or refresh.
5. For refresh workflows, filter to changed linked sources before generating knowledge.
6. Hand off processed source IDs to `fini-api-knowledge` for generation, drafts, publishing, and assignment.
7. For delete, produce an exact source/article impact plan and require confirmation.

## Defaults

- Treat ingestion and refresh as async.
- Prefer reviewable generated drafts after source import or refresh.
- For refresh, generate knowledge only for sources that actually changed and already link to knowledge, unless the user explicitly wants broader behavior.
- Do not promise answer changes until knowledge generation, publish/review, and assignment are verified.
- Translate "train the bot" or "sync docs" into source processing plus a knowledge handoff; source import alone is not enough.
- If the source content is current but the bot still routes, escalates, classifies, or speaks incorrectly, hand off to `fini-api-agent-configuration` instead of refreshing sources again.

## Gotchas

- The API contract uses `documentIdsToAdd` and `documentIdsToRefresh`; the product concept is "sources".
- For web, values are URLs.
- For providers, values are source IDs returned after registration.
- A queued ingestion response is not completion.
- Completed source processing does not mean live agent answers changed.
- Provider discovery may exclude already-imported resources.
- Provider discovery requires the provider to be connected in the workspace first.
- Discovery is for new importable provider content; list sources for content already imported into Fini.
- Source deletion can optionally delete linked articles; treat this as destructive.

## Proof Of Completion

- Import: source records exist and status fields show completion or failure.
- Polling: terminal status reached, with `success` and `error` interpreted.
- Refresh: changed linked sources identified before generation.
- Delete plan: source IDs, linked article impact, `deleteArticles` behavior, and confirmation status are explicit.
- Knowledge handoff: identify which source IDs should feed bulk generation.

## References

Read [source-ingestion-refresh.md](references/source-ingestion-refresh.md) for web import, provider import, refresh, changed-source detection, polling, and deletion guardrails.
