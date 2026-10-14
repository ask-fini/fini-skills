---
name: fini-api-agents
description: Use when the user wants to list, find, create, or delete Fini agents; resolve a botId; inspect agent-level analytics, performance summaries, trends, knowledge usage, rule performance, escalations, or CSAT; manage a bare agent lifecycle; or route a newly created agent into prompts, knowledge, Actions/Attributes, and testing. Do not use it to claim that the public API creates a workspace, installs Slack, or provisions channel integrations.
---

# Fini API Agents

Manage the workspace agent record and read its aggregate analytics. Treat `agent` in product language and `bot` in API paths as the same entity.

Before endpoint-specific work, fetch `https://docs.usefini.com/llms.txt` and the current Agents or Analytics pages. Keep endpoint schemas in the live docs; use this skill for workflow choice, sequencing, verification, and handoff.

## Choose The Workflow

| User intent | Workflow |
| --- | --- |
| "What agents exist?" / "Find the billing bot" | List agents and resolve the exact `botId` |
| "Create an agent" | Duplicate-name preflight -> create bare agent -> verify -> configure |
| "Delete this agent" | Resolve exact ID -> inventory dependencies -> obtain exact-target authorization when absent -> delete -> verify absence |
| "How is this agent performing?" | Full analytics over an explicit window |
| "Show only escalations/rules/trends" | Narrow analytics section over an explicit window |
| "Set up prompts, knowledge, actions, or tests" | Resolve the agent here, then route to the owning skill |

## Agent Lifecycle

1. List agents before any name-based create or delete.
2. Resolve and retain the exact `botId`; names are display labels, not mutation targets.
3. For create, check for an existing exact-name agent, create one unique agent name, capture the returned ID, and re-list to verify it.
4. Treat the created object as a bare agent. Inspect its actual prompts, knowledge, rules, Actions/Attributes, and integrations before claiming readiness.
5. For delete, inventory dependencies tied to the exact ID and summarize what may stop working. Do not infer cascade or recovery behavior that the docs do not specify.
6. Require clear deletion authorization bound to the exact agent ID when the current request has not already supplied it. Do not ask twice when exact-target authorization is already explicit. For the MCP connector, pass the tool-required `confirmationPhrase: "CONFIRM"` only after that authorization.
7. Delete only the exact agent the current request authorizes. Do not expand a single-agent request into related-object cleanup.
8. Verify the delete response and confirm that List agents no longer returns the ID.

Read [agent-lifecycle.md](references/agent-lifecycle.md) for endpoint behavior, dependency preflight, deletion verification, and post-create routing.

## Agent Analytics

1. Resolve the agent ID.
2. Define `startEpoch` and `endEpoch` explicitly in Unix epoch milliseconds, with start before end.
3. Apply only filters supported by the current docs. Keep the timezone and filter set stable when comparing periods or agents.
4. Request the narrowest useful section:
   - `summary` for totals, rates, status, and channel usage.
   - `trends` for time-series and hourly data.
   - `knowledge` for folder usage.
   - `rules` for rule-level performance.
   - `escalations` for escalation reasons.
5. Use the full analytics endpoint only when multiple sections are required.
6. Preserve the returned structured fields and distinguish missing data from zero. Use conversation evidence for root-cause claims; aggregates alone do not prove why a metric changed.

Read [agent-analytics.md](references/agent-analytics.md) for filters, section selection, interpretation, and reporting.

## Route The Next Layer

| Need after agent resolution or creation | Owning skill |
| --- | --- |
| Prompts, prompt drafts, rules, tags | `fini-api-agent-configuration` |
| Source ingestion | `fini-api-sources` |
| Articles, folders, generation, agent knowledge assignment | `fini-api-knowledge` |
| Actions, Attributes, external API calls, agent assignments | `fini-api-actions-attributes` |
| Replays, persistent Test Suite runs, criteria, or rule evaluation | `fini-api-testing` |
| One direct behavioral turn | `fini-api-generate-answer` |

Creating an agent does not create a Fini workspace, install Slack, authorize a provider, or provision a customer channel. State that boundary plainly and continue only with capabilities the public API actually exposes.

## Proof Of Completion

- **Resolve:** report the exact agent ID and name.
- **Create:** report the returned ID, prove it appears in List agents, and separate creation from configuration/readiness.
- **Delete:** record the reviewed dependencies, report the API result, and prove the exact ID is absent from List agents.
- **Analytics:** report agent ID, exact time window, timezone, filters, requested section, returned metrics, and any missing-data caveats.
