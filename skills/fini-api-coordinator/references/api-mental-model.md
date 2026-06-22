# Fini Public API Mental Model

Use this reference for broad API architecture questions, bad-answer triage, and existing-agent onboarding plans. Fetch `https://docs.usefini.com/llms.txt` before relying on route names, fields, or enums.

## API Layers

| Layer | What it controls | Does it change live answers by itself? |
| --- | --- | --- |
| Agents | Discover existing Fini agents and resolve names to `botId` | No. It is identity/discovery. |
| Agent configuration | Prompts, rulebooks/rules, rule drafts, tags, tag groups | Yes, after approved writes such as prompt version update or rule publish/update. |
| Sources | Raw/provider/web content ingestion and refresh | No. Sources must become knowledge first. |
| Knowledge | Generated or direct articles, drafts, folders, assignment | Only after publish/live state and agent visibility are correct. |
| Runtime | Generate Answer message events and metadata/user attributes | It creates conversation events and tests behavior; it does not edit config or knowledge. |
| Conversations | Conversation export, evidence, QA, deletion | No, except deletion affects stored conversation records. |

## What Actually Changes Bot Behavior

| Desired change | Required API path | Proof |
| --- | --- | --- |
| Change instructions/tone/planning | Prompt update on the target agent | Prompts re-fetched, then Generate Answer test. |
| Change routing/escalation/workflow behavior | Rule draft/refine/publish or rule update | Rule re-fetched with target `botIds`, then Generate Answer test. |
| Add routing or reporting categories | Custom tag group/tag design | Tags re-listed; if used by rules, rulebook availability is clear. |
| Add new facts from docs | Sources -> Knowledge drafts -> publish -> assignment | Published article exists and scoped tree shows visibility. |
| Update facts from changed docs | Refresh sources -> changed filter -> draft updates -> publish | Changed source IDs, job results, published article IDs. |
| Prove a scenario works | Generate Answer | Created events, answer text, evidence/used articles, pass/fail. |
| Build improvement backlog | Conversations export | Golden-set rows with failure type and recommended workflow. |

## Bad Answer Triage

Classify before changing anything:

| Symptom | Likely layer | Next skill |
| --- | --- | --- |
| Bot lacks a factual answer or cites stale content | Sources/Knowledge | `fini-api-sources` then `fini-api-knowledge` |
| Article exists but bot cannot use it | Knowledge visibility | `fini-api-knowledge` |
| Bot says the right facts in the wrong tone/style | Prompts | `fini-api-agent-configuration` |
| Bot escalates, routes, or follows a workflow incorrectly | Rulebooks/rules | `fini-api-agent-configuration` |
| Bot applies the wrong category/outcome label | Tags/tag groups or rules | `fini-api-agent-configuration` |
| Bot answer depends on customer/account context that was not sent | Runtime metadata | `fini-api-generate-answer` |
| Customer wants refunds, account updates, payments, or subscriptions | Customer-owned action API | Coordinator only; verify docs, then gather integration requirements. |
| Need evidence from real conversations | Conversations | `fini-api-conversations` |

## Existing-Agent Onboarding Model

For an existing Fini agent, the API-backed onboarding sequence is:

1. Resolve agent identity.
2. Design tags for routing, outcomes, and reporting.
3. Draft/refine/publish rules for behavior and routing.
4. Inspect/update prompts for global instructions.
5. Ingest/refresh sources for customer content.
6. Generate, review, publish, organize, and assign knowledge.
7. Test with Generate Answer.
8. Export conversations for QA and golden-set improvement.

Do not claim the public APIs create new workspaces, install Slack, or provision brand-new agents unless the current docs expose those routes.
