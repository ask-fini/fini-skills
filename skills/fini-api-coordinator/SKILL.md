---
name: fini-api-coordinator
description: Use when the user asks which Fini public API workflow or endpoint to use, wants to integrate with Fini APIs, import or refresh sources, generate or manage knowledge, export conversations, send a message turn into a Fini agent, organize folders, assign knowledge to agents, or choose between source-backed generation, direct article management, and conversation APIs. Route to the narrower Fini API skill and verify current endpoint details through docs.usefini.com/llms.txt.
---

# Fini API Coordinator

Use this as the front door for Fini public API work. Route the user's intent to the smallest focused Fini API skill, then let that skill handle the workflow.

Do not copy endpoint tables from memory. Before relying on paths, parameters, scopes, response fields, or enum values, fetch `https://docs.usefini.com/llms.txt` and then the relevant `.md` docs page.

## Freshness Check

When this skill comes from an installed Fini skills package, check whether the package should be updated before doing endpoint-specific work. Keep this scoped to Fini skills; do not run a broad `skills update` that updates every installed skill.

Use the package-specific update path once packaging exists:

```bash
npx skills update fini-api-coordinator fini-api-conversations fini-api-generate-answer fini-api-sources fini-api-knowledge -y
```

If the package later uses different skill names or a project lockfile, follow the package README instead. If update commands are unavailable, continue with the installed skill but verify live endpoint details through `https://docs.usefini.com/llms.txt`.

## Routing

| User intent | Use |
| --- | --- |
| Export, inspect, analyze, or delete conversations | `fini-api-conversations` |
| Send a test/user turn into a Fini agent or continue a conversation | `fini-api-generate-answer` |
| Import web/provider content, crawl links, refresh sources, poll source status, or detect changed sources | `fini-api-sources` |
| Generate knowledge, manage articles/drafts, publish, move articles, inspect the tree, or assign folders to agents | `fini-api-knowledge` |

If the request spans multiple areas, sequence skills in the product workflow order:

1. Sources: ingest or refresh raw content.
2. Knowledge: generate drafts or live articles from processed sources.
3. Knowledge: organize articles into folders and assign folders to agents.
4. Generate Answer: test whether the intended agent can answer from the updated knowledge.
5. Conversations: export or inspect real conversations for QA, analytics, or evidence.

## Default Behavior

- Prefer read-only discovery before write calls.
- Prefer draft/review paths before live publish.
- Prefer source-backed generation for imported content.
- Use direct article management only when the user wants exact manual control over the article body or lifecycle.
- Treat delete, publish, folder assignment, and production answer generation as live-impacting.
- Produce a dry-run style plan before live-impacting actions unless the user already gave exact IDs and explicit execution intent.
- Do not ask the user to paste API keys into chat. Use environment variables or the caller's secure API flow when available.

## What This Pack Adds

The docs already contain endpoint references. The skills should add the judgment layer:

- Which workflow to choose.
- Which API response determines the next step.
- Where async polling is required.
- Whether draft, live, or assignment state means behavior has changed.
- When read vs write scope is needed.
- What proof shows the workflow worked.
- When to stop and ask before destructive or live-impacting changes.

## High-Value Gotchas

- Source ingestion is not live knowledge.
- Generated drafts do not affect answers until reviewed/published and visible to the agent.
- Knowledge can exist but still be invisible to an agent if folder assignment is wrong.
- Conversation and source list pagination is newest-first, so `direction=next` moves older.
- Generate Answer returns created events, not the full conversation wrapper.
- Public source APIs use `document*` wire names even though the product language is "sources".
- For web ingestion, source inputs are URLs; for provider ingestion, inputs are registered source IDs.

## References

Read [workflow-routing.md](references/workflow-routing.md) when the user needs a multi-step plan, a skill routing table, or a customer-facing explanation of which Fini API workflow to use.
