---
name: fini-api-coordinator
description: Use when the user asks which Fini public API workflow or endpoint to use for Fini client bot/runtime, agent configuration, or knowledge-base management; wants to train a bot from help center, internal docs, websites, Notion, Zendesk, or Confluence; configure prompts, prompt history, rulebooks/rules, rule drafts, tags, or tag groups; refresh or sync changed knowledge sources; generate, review, publish, organize, or assign knowledge; export conversations for QA or golden-set work; send or test a message turn with metadata/user attributes; or choose between agent configuration, source-backed generation, direct article management, Generate Answer, and conversation APIs. Route to the narrower Fini API skill and verify current endpoint details through docs.usefini.com/llms.txt.
---

# Fini API Coordinator

Use this as the front door for Fini public API work. Route the user's intent to the smallest focused Fini API skill, then let that skill handle the workflow.

Do not copy endpoint tables from memory. Before relying on paths, parameters, scopes, response fields, or enum values, fetch `https://docs.usefini.com/llms.txt` and then the relevant `.md` docs page.

## Freshness Check

When this skill comes from an installed Fini skills package, check whether the package should be updated before doing endpoint-specific work. Keep this scoped to Fini skills; do not run a broad `skills update` that updates every installed skill.

For project-scoped installs, run from the project root:

```bash
npx skills update --project fini-api-coordinator fini-api-agent-configuration fini-api-conversations fini-api-generate-answer fini-api-sources fini-api-knowledge -y
```

For global installs, use `--global` instead of `--project`. If the install is not tracked by the CLI, the update command is unavailable, or a newly released Fini skill is missing after update, re-run `npx skills add ask-fini/fini-skills --all --copy -y`, then continue. Always verify live endpoint details through `https://docs.usefini.com/llms.txt`.

## Routing

| User intent | Use |
| --- | --- |
| Configure prompts, prompt history, rulebooks/rules, rule drafts, tags, or tag groups | `fini-api-agent-configuration` |
| Onboard/configure an existing Slack/support agent's behavior | `fini-api-agent-configuration` first, then Sources/Knowledge/Test as needed |
| Export, inspect, analyze, or delete conversations | `fini-api-conversations` |
| Build QA exports, golden-set candidates, or conversation evidence | `fini-api-conversations` |
| Send a test/user turn into a Fini agent, continue a conversation, or pass user metadata | `fini-api-generate-answer` |
| Import web/provider content, crawl links, refresh sources, poll source status, or detect changed sources | `fini-api-sources` |
| Generate knowledge, manage articles/drafts, publish, move articles, inspect the tree, or assign folders to agents | `fini-api-knowledge` |
| "Train the bot", "sync docs", "update the KB", or "make this visible to one agent" | Sequence Sources -> Knowledge -> assignment -> Generate Answer proof |
| Customer-owned business actions such as refunds, account updates, payment retries, or subscription changes | Out of scope unless current Fini docs expose a public route; gather requirements and state the gap |

If the request spans multiple areas, sequence skills in the product workflow order:

1. Agent configuration: design tags, draft/publish rulebooks, and adjust prompts for existing agents.
2. Sources: ingest or refresh raw content.
3. Knowledge: generate drafts or live articles from processed sources.
4. Knowledge: organize articles into folders and assign folders to agents.
5. Generate Answer: test whether the intended agent behaves correctly.
6. Conversations: export or inspect real conversations for QA, analytics, or evidence.

## Default Behavior

- Prefer read-only discovery before write calls.
- Prefer draft/review paths before live publish.
- Prefer source-backed generation for imported content.
- Use direct article management only when the user wants exact manual control over the article body or lifecycle.
- Prefer rule drafts before published rule creation when configuring behavior from natural-language instructions.
- Treat prompt updates, rule publish/update/delete, tag delete, knowledge publish/delete, folder assignment, and production answer generation as live-impacting.
- Produce a dry-run style plan before live-impacting actions unless the user already gave exact IDs and explicit execution intent.
- Do not ask the user to paste API keys into chat. Use environment variables or the caller's secure API flow when available.
- Translate customer language into Fini concepts: "train/sync docs" usually means Sources plus Knowledge, while "make the bot use it" usually also needs folder assignment and a runtime test.
- Translate "configure/onboard the agent" into prompts, rulebooks/rules, tags, knowledge, and runtime tests depending on the requested layer.
- Do not imply Fini public APIs can perform customer-owned business actions unless the live docs show a supported route.

## Pack Boundary

The docs already contain endpoint references. The skills add workflow judgment: which path to choose, which response determines the next step, where review/polling is required, and what proof shows the workflow worked.

## High-Value Gotchas

- Agent configuration APIs are for existing Fini agents; do not promise full agent/workspace/Slack provisioning unless current docs expose those routes.
- Prompt updates save a new version and require the full prompt section arrays.
- Natural-language rulebook creation should default to a draft rule, not direct published rule creation.
- Tag groups marked as output-only are not available to Rulebooks.
- Source ingestion is not live knowledge.
- Generated drafts do not affect answers until reviewed/published and visible to the agent.
- Knowledge can exist but still be invisible to an agent if folder assignment is wrong.
- Conversation and source list pagination is newest-first, so `direction=next` moves older.
- Generate Answer returns created events, not the full conversation wrapper.
- Public source APIs use `document*` wire names even though the product language is "sources".
- For web ingestion, source inputs are URLs; for provider ingestion, inputs are registered source IDs.
- Public APIs may expose bot/KB state but not the customer's own action APIs. Treat refund, subscription, payment, order, and account-update requests as integration discovery unless current Fini docs say otherwise.

## References

Read [api-mental-model.md](references/api-mental-model.md) when the user asks what the Fini public APIs can do, how the API areas fit together, how to triage a bad answer, or how to configure/onboard an existing agent end to end.

Read [workflow-routing.md](references/workflow-routing.md) when the user needs a multi-step plan, a skill routing table, or a customer-facing explanation of which Fini API workflow to use.

Read [customer-recipes.md](references/customer-recipes.md) when the user describes a customer job such as training a bot from docs, refreshing changed sources, proving quality with golden sets, using user attributes, or making knowledge visible to one agent.
