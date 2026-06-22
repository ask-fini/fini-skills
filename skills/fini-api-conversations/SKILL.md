---
name: fini-api-conversations
description: Use when the user wants to list, export, inspect, summarize, analyze, paginate, retrieve, delete, or bulk-delete Fini conversations through the public API; build QA, analytics, compliance, BI, RCA, golden-set, support-review, CSAT/feedback, knowledge-improvement, or agent-configuration-improvement workflows from Fini conversation data; or understand conversation filters, cursors, evidence fields, used articles, user attributes, and deletion guardrails.
---

# Fini API Conversations

Use this skill for conversation data workflows. It is not for sending a new turn into an agent; use `fini-api-generate-answer` for that.

Before endpoint-specific work, fetch `https://docs.usefini.com/llms.txt` and the relevant conversation docs page.

## Workflow

1. Clarify the goal: export, inspect one conversation, analyze evidence, or delete.
2. Resolve agent identity only if needed. If the user gives an agent name instead of an ID, use List agents from the docs to map it to `botId`.
3. Choose a bounded window. Do not rely on defaults for exports.
4. List conversations with the narrowest useful filters.
5. Page newest to older using the documented cursor and direction semantics.
6. For analytics/RCA, extract evidence fields and preserve source conversation IDs.
7. For QA/golden-set work, preserve expected vs actual answer fields, used articles, user attributes, feedback, failure type, and next action.
8. For deletion, produce an exact deletion plan and wait for explicit confirmation before calling delete routes.

## Defaults

- Use a narrow time window for exploration before large exports.
- Treat 90 days as the maximum single query window unless current docs say otherwise.
- Store or summarize cursor state when paging long exports.
- Return counts, filters used, time window, and pagination status in summaries.
- Preserve event IDs, used articles, user attributes, status/resolution, and feedback fields when the export is for quality improvement.
- Route evidence by failure type: content gaps to Knowledge/Sources, behavior config issues to Agent Configuration, and runtime context issues to Generate Answer metadata.
- Do not bulk delete unless the user explicitly confirms exact IDs.

## Gotchas

- List conversations defaults to a recent window, so "all conversations" needs explicit paging and windows.
- Results are sorted newest-first; `direction=next` moves older.
- The endpoint returns conversations Fini touched, not necessarily every customer conversation in an external system.
- `userAttributes` can be workspace-defined and schema-less; do not build brittle assumptions around it.
- Bulk delete can have partial-effect behavior. Check current docs and warn before execution.

## Proof Of Completion

- Export: report filters, time range, count, whether `hasMore` remains, and where data was written or how it was summarized.
- Single conversation lookup: include the conversation ID, event count, key timestamps, and whether referenced articles or user attributes were present.
- QA/golden-set export: include question/evidence, actual answer, used articles, feedback/resolution signal, and recommended next action.
- Deletion plan: include exact IDs, count, route type, expected irreversibility, and confirmation status.
- Completed deletion: re-check or report API response evidence from current docs.

## References

Read [conversation-export-playbook.md](references/conversation-export-playbook.md) for export pagination, analysis outputs, and deletion planning templates.

Read [qa-and-golden-set.md](references/qa-and-golden-set.md) when using conversations to build a golden set, find KB gaps, evaluate user attributes, or plan knowledge improvements.
