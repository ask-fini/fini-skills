---
name: fini-api-conversations
description: Use when the user wants to list, export, inspect, summarize, analyze, paginate, retrieve, delete, or bulk-delete Fini conversations through the public API; build QA, analytics, compliance, BI, RCA, golden-set, support-review, CSAT/feedback, knowledge-improvement, fix-review iteration, or agent-configuration-improvement workflows from Fini conversation data; approve or reject a reviewed answer event; or understand conversation filters, cursors, evidence fields, used articles, user attributes, and deletion guardrails.
---

# Fini API Conversations

Use this skill for conversation data workflows. It is not for sending a new turn into an agent; use `fini-api-generate-answer` for that.

Before endpoint-specific work, fetch `https://docs.usefini.com/llms.txt` and the relevant conversation docs page.

## Workflow

1. Clarify the goal: export, inspect one conversation, analyze evidence, run fix review, approve a reviewed event, or delete.
2. Resolve agent identity only if needed. If the user gives an agent name instead of an ID, use List agents from the docs to map it to `botId`.
3. Choose a bounded window. Do not rely on defaults for exports.
4. List conversations with the narrowest useful filters.
5. Page newest to older using the documented cursor and direction semantics. Treat 100 as a page-size ceiling, not an export ceiling.
6. Continue until `hasMore=false` or the user-requested bound is reached. If interrupted or rate-limited repeatedly, stop cleanly and preserve the resume cursor.
7. Process or write each page incrementally instead of accumulating an unbounded transcript payload in context.
8. For analytics/RCA, extract evidence fields and preserve source conversation IDs. Avoid fetching every conversation again when the list response already contains the required events and evidence.
9. For QA/golden-set work, preserve expected vs actual answer fields, used articles, user attributes, feedback, failure type, and next action.
10. For fix review, target the exact conversation and answer event, create an iteration from concise feedback, then inspect the resulting session. Treat recommendations and replay answers as proposals, not applied configuration.
11. For event approval, show the exact conversation/event and requested approved state, then require explicit confirmation because it changes QA/resolution state.
12. For deletion, produce an exact deletion plan and wait for explicit confirmation before calling delete routes.

## Defaults

- Use a narrow time window for exploration before large exports.
- Treat 90 days as the maximum single query window unless current docs say otherwise.
- Use at most 100 records per page. For larger exports, traverse cursors instead of raising `limit` above the documented maximum.
- Store or summarize cursor state when paging long exports, including the final cursor and last timestamp processed.
- Respect the documented 100-request-per-60-second throttle. On `429`, retry the same cursor with bounded backoff; after repeated failures, return an incomplete export with a resume cursor.
- Return counts, filters used, time window, and pagination status in summaries.
- Preserve event IDs, used articles, user attributes, status/resolution, and feedback fields when the export is for quality improvement.
- Route evidence by failure type: content gaps to Knowledge/Sources, behavior config issues to Agent Configuration, and runtime context issues to Generate Answer metadata.
- Prefer fix-review iteration when the user wants a grounded proposed correction from a specific bad answer. Route any approved recommendation to the owning Knowledge or Agent Configuration workflow for a separate write.
- Do not bulk delete unless the user explicitly confirms exact IDs.

## Gotchas

- List conversations defaults to a recent window, so "all conversations" needs explicit paging and windows.
- Results are sorted newest-first; `direction=next` moves older.
- A page containing 100 records is not proof that only 100 conversations matched. Check `hasMore` and continue with `nextCursor`.
- Do not create an N+1 read pattern by fetching every conversation again unless fields required by the task are absent from the list response.
- The endpoint returns conversations Fini touched, not necessarily every customer conversation in an external system.
- `userAttributes` can be workspace-defined and schema-less; do not build brittle assumptions around it.
- A fix-review iteration does not apply or publish its recommendations. Verify the session output, then use the owning skill for any real KB, prompt, or rule change.
- Event approval is not read-only; it changes review/resolution state and needs explicit confirmation.
- Bulk delete can have partial-effect behavior. Check current docs and warn before execution.

## Proof Of Completion

- Export: report filters, time range, count, pages processed, whether `hasMore` remains, final cursor/timestamp, and where data was written or how it was summarized. Label bounded or interrupted exports as incomplete.
- Single conversation lookup: include the conversation ID, event count, key timestamps, and whether referenced articles or user attributes were present.
- QA/golden-set export: include question/evidence, actual answer, used articles, feedback/resolution signal, and recommended next action.
- Fix review: include conversation ID, event ID, session/iteration ID, old and replay answers, recommendations, and whether any recommendation was separately applied.
- Deletion plan: include exact IDs, count, route type, expected irreversibility, and confirmation status.
- Completed deletion: re-check or report API response evidence from current docs.

## References

Read [conversation-export-playbook.md](references/conversation-export-playbook.md) for export pagination, analysis outputs, and deletion planning templates.

Read [qa-and-golden-set.md](references/qa-and-golden-set.md) when using conversations to build a golden set, find KB gaps, evaluate user attributes, or plan knowledge improvements.
