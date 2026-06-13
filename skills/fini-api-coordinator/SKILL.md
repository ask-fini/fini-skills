---
name: fini-api-coordinator
description: Use when the user asks which Fini public API workflow or endpoint to use for Fini client bot/runtime or knowledge-base management; wants to train a bot from help center, internal docs, websites, Notion, Zendesk, or Confluence; refresh or sync changed knowledge sources; generate, review, publish, organize, or assign knowledge; export conversations for QA or golden-set work; send or test a message turn with metadata/user attributes; or choose between source-backed generation, direct article management, Generate Answer, and conversation APIs. Route to the narrower Fini API skill and verify current endpoint details through docs.usefini.com/llms.txt.
---

# Fini API Coordinator

Use this as the front door for Fini public API work. Route the user's intent to the smallest focused Fini API skill, then let that skill handle the workflow.

Do not copy endpoint tables from memory. Before relying on paths, parameters, scopes, response fields, or enum values, fetch `https://docs.usefini.com/llms.txt` and then the relevant `.md` docs page.

## Freshness Check

When this skill comes from an installed Fini skills package, check whether the package should be updated before doing endpoint-specific work. Keep this scoped to Fini skills; do not run a broad `skills update` that updates every installed skill.

For project-scoped installs, run from the project root:

```bash
npx skills update --project fini-api-coordinator fini-api-conversations fini-api-generate-answer fini-api-sources fini-api-knowledge -y
```

For global installs, use `--global` instead of `--project`. If the install is not tracked by the CLI or the update command is unavailable, re-run `npx skills add ask-fini/fini-skills --all --copy -y`, then continue. Always verify live endpoint details through `https://docs.usefini.com/llms.txt`.

## Routing

| User intent | Use |
| --- | --- |
| Export, inspect, analyze, or delete conversations | `fini-api-conversations` |
| Build QA exports, golden-set candidates, or conversation evidence | `fini-api-conversations` |
| Send a test/user turn into a Fini agent, continue a conversation, or pass user metadata | `fini-api-generate-answer` |
| Import web/provider content, crawl links, refresh sources, poll source status, or detect changed sources | `fini-api-sources` |
| Generate knowledge, manage articles/drafts, publish, move articles, inspect the tree, or assign folders to agents | `fini-api-knowledge` |
| "Train the bot", "sync docs", "update the KB", or "make this visible to one agent" | Sequence Sources -> Knowledge -> assignment -> Generate Answer proof |
| Customer-owned business actions such as refunds, account updates, payment retries, or subscription changes | Out of scope unless current Fini docs expose a public route; gather requirements and state the gap |

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
- Translate customer language into Fini concepts: "train/sync docs" usually means Sources plus Knowledge, while "make the bot use it" usually also needs folder assignment and a runtime test.
- Do not imply Fini public APIs can perform customer-owned business actions unless the live docs show a supported route.

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
- Public APIs may expose bot/KB state but not the customer's own action APIs. Treat refund, subscription, payment, order, and account-update requests as integration discovery unless current Fini docs say otherwise.

## References

Read [workflow-routing.md](references/workflow-routing.md) when the user needs a multi-step plan, a skill routing table, or a customer-facing explanation of which Fini API workflow to use.

Read [customer-recipes.md](references/customer-recipes.md) when the user describes a customer job such as training a bot from docs, refreshing changed sources, proving quality with golden sets, using user attributes, or making knowledge visible to one agent.
