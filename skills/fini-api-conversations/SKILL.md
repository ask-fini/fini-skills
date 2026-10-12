---
name: fini-api-conversations
description: Use when the user wants to list, export, inspect, summarize, analyze, paginate, retrieve, delete, or bulk-delete Fini conversations through the public API; inspect one Fini-authored event's stored execution metadata, including planning, knowledge, tag, Attribute, and Rule traces; record or clear an event's thumbs-up/down rating, teammate feedback note, or resolution state; build QA, compliance, BI, RCA, golden-set, support-review, knowledge-improvement, replay, regression-testing, or agent-analytics workflows from conversation evidence; prepare an exact bad-response handoff to fini-api-refine-with-ai; or understand conversation filters, cursors, event fields, used articles, user attributes, and deletion guardrails.
---

# Fini API Conversations

Use this skill for conversation data workflows. It is not for sending a new turn into an agent; use `fini-api-generate-answer` for that.

Before endpoint-specific work, fetch `https://docs.usefini.com/llms.txt` and the relevant conversation docs page.

## Choose The Path

| User intent | Path |
| --- | --- |
| Export or inspect conversations | Bounded list/get workflow in this skill |
| Explain why one exact Fini response behaved as it did | Resolve the exact `role=finibot` event, then read its stored event metadata |
| Give an event thumbs up/down or clear its rating | Conversation feedback with `approved: true`, `false`, or `null` |
| Save or clear a teammate explanation of why an event was good or bad | Feedback-note workflow |
| Mark a negatively rated issue handled or reopened | Feedback-resolution workflow with `resolved: true` or `false` |
| Diagnose and improve one exact bad Fini response with generated drafts and an automatic replay | Preserve the exact evidence, then use `fini-api-refine-with-ai` |
| Re-run an answer, compare configurations, or add regression coverage | Preserve the exact conversation/event evidence, then use `fini-api-testing` |
| Ask for aggregate agent metrics, trends, knowledge usage, or escalation breakdowns | Use `fini-api-agents` analytics, then return here for conversation-level evidence |
| Send a new message turn | Use `fini-api-generate-answer` |

## Workflow

1. Clarify the goal: export, inspect one conversation, analyze evidence, record feedback, prepare Refine/Replay/testing evidence, or delete.
2. Resolve agent identity only if needed. If the user gives an agent name instead of an ID, use List agents from the docs to map it to `botId`.
3. Choose a bounded window. Do not rely on defaults for exports.
4. List conversations with the narrowest useful filters.
5. Page newest to older using the documented cursor and direction semantics. Treat 100 as a page-size ceiling, not an export ceiling.
6. Continue until `hasMore=false` or the user-requested bound is reached. If interrupted or rate-limited repeatedly, stop cleanly and preserve the resume cursor.
7. Process or write each page incrementally instead of accumulating an unbounded transcript payload in context.
8. For conversation-level RCA, extract evidence fields and preserve source conversation IDs. Avoid fetching every conversation again when the list response already contains the required events and evidence. For aggregate metrics or trends, hand off to `fini-api-agents` rather than deriving workspace-wide conclusions from a sample export.
9. For exact-event RCA, fetch metadata only for the selected Fini-authored event. Treat nullable trace sections as absent or not recorded, not as proof that a stage failed. Separate observed planning, knowledge selection, answer reasoning, tag selection, executed Attributes, and executed Rules before routing the issue to its owning workflow.
10. For QA/golden-set work, preserve expected vs actual answer fields, used articles, user attributes, feedback, failure type, and next action.
11. When the user wants AI-assisted diagnosis and a proposed correction for one exact bad response, preserve the exact conversation and `finibot` event, then hand execution to `fini-api-refine-with-ai` instead of manually creating drafts here.
12. Keep event rating, qualitative note, and resolution as three separate writes. Target the exact conversation and event, use `approved` only for the thumbs signal, use `feedback` only for the note, set `resolved: true` after handling a negatively rated event, and use `resolved: false` to mark it unresolved again.
13. For replay, rule evaluation, or regression testing, preserve the exact source conversation and Fini response event, then hand off to `fini-api-testing`. Do not treat a replay as a live configuration change or an automatically saved regression case.
14. For deletion, produce an exact deletion plan and wait for explicit confirmation before calling delete routes. Bulk-delete requests accept 1-50 unique conversation IDs; split larger exact sets into disclosed batches before confirmation.

## Defaults

- Use a narrow time window for exploration before large exports.
- Treat 90 days as the maximum single query window unless current docs say otherwise.
- Use at most 100 records per page. For larger exports, traverse cursors instead of raising `limit` above the documented maximum.
- Store or summarize cursor state when paging long exports, including the final cursor and last timestamp processed.
- Respect the documented 100-request-per-60-second throttle. On `429`, retry the same cursor with bounded backoff; after repeated failures, return an incomplete export with a resume cursor.
- Return counts, filters used, time window, and pagination status in summaries.
- Preserve event IDs, used articles, user attributes, status/resolution, and feedback fields when the export is for quality improvement.
- Preserve rating (`approved`), qualitative note (`feedback`), event resolution, conversation resolution, and CSAT as distinct signals. Do not collapse them into one "approval" field.
- Route evidence by failure type: content gaps to Knowledge/Sources; behavior config issues to Agent Configuration; configured Attribute, Data Step, or Action failures to Actions/Attributes; Action reachability to Agent Configuration Tool-node wiring; and caller-supplied runtime context issues to Generate Answer metadata.
- Prefer `fini-api-refine-with-ai` when the user wants a grounded proposed correction from a specific bad answer. It owns the iteration, generated draft lineage, and embedded replay; this skill retains evidence and QA state.
- Use `fini-api-draft-validation` for unpublished exact-ID candidate comparisons. Use `fini-api-testing` for Replays, rule evaluation, and persistent Test Suite runs against captured published or selected draft configuration; those runs simulate Actions using recorded/custom mocks and captured Attribute context.
- Use `fini-api-agents` for aggregate analytics and section-level performance reads.
- Do not bulk delete unless the user explicitly confirms exact IDs and, for more than 50 IDs, the disclosed batch boundaries.

## Gotchas

- List conversations defaults to a recent window, so "all conversations" needs explicit paging and windows.
- Results are sorted newest-first; `direction=next` moves older.
- A page containing 100 records is not proof that only 100 conversations matched. Check `hasMore` and continue with `nextCursor`.
- Do not create an N+1 read pattern by fetching every conversation again unless fields required by the task are absent from the list response.
- Event metadata accepts only a Fini-authored event with `role=finibot`; do not substitute the linked user event. The direct API route unusually omits the common `/public` suffix.
- Event metadata is a stored read-only trace. It does not replay the answer, rerun Rules, call Actions, or change configuration.
- Trace sections may be `null`. Do not invent execution evidence from an absent section.
- Attribute and external-step results may contain customer data. Report only task-relevant evidence and never expose secrets or unnecessary payloads.
- Avoid an N+1 metadata sweep across broad exports. Fetch metadata only for events selected for diagnosis.
- The endpoint returns conversations Fini touched, not necessarily every customer conversation in an external system.
- `userAttributes` can be workspace-defined and schema-less; do not build brittle assumptions around it.
- The wire field `approved` is the event's thumbs rating: `true` is thumbs up, `false` is thumbs down, and `null` clears it. It is not configuration approval.
- A feedback note does not set the thumbs rating, and marking feedback resolved does not erase the underlying rating or note.
- `resolved: false` explicitly reopens or marks a negatively rated event unresolved; it does not clear its rating or note.
- Conversation `resolved` is recalculated from negatively rated Fini events: no negatives yields `null`, all handled negatives yields `true`, and any unresolved negative yields `false`.
- Refine with AI creates persisted draft recommendations and a separate replay but does not publish them. Route the iteration lifecycle to `fini-api-refine-with-ai` and keep event rating, note, and resolution separate.
- Replays are separate conversations and require a Fini response event with a linked user event. Route their creation and comparison to `fini-api-testing`.
- Bulk delete accepts 1-50 unique IDs per request and can have partial-effect behavior. Plan larger exact sets as disclosed batches of at most 50 before authorization; after any error, reconcile completed and remaining IDs before continuing.

## Proof Of Completion

- Export: report filters, time range, count, pages processed, whether `hasMore` remains, final cursor/timestamp, and where data was written or how it was summarized. Label bounded or interrupted exports as incomplete.
- Single conversation lookup: include the conversation ID, event count, key timestamps, and whether referenced articles or user attributes were present.
- Exact-event RCA: report the conversation ID, exact Fini event ID, confirmed `finibot` role, returned and absent trace sections, evidence-based diagnosis, and owning follow-up skill. Do not imply that any replay, test, or mutation occurred.
- QA/golden-set export: include question/evidence, actual answer, used articles, feedback/resolution signal, and recommended next action.
- Feedback write: re-read the target event and report its rating, note, event-level resolution, and recalculated conversation resolution separately.
- Refine handoff: include the conversation ID, exact `finibot` event ID, linked-user-event eligibility when known, evidence/feedback goal, and QA state. The Refine skill owns session, iteration, replay, and generated-draft proof.
- Replay/testing handoff: include source conversation ID, target Fini response event ID, linked user event availability, intended replay mode or regression goal, and the testing workflow receiving it.
- Analytics handoff: report the agent, time window, filters, requested metric/section, and which conversation evidence should be inspected afterward.
- Deletion plan: include exact IDs, total count, route type, batch count and boundaries when above 50, expected irreversibility, and confirmation status.
- Completed deletion: re-check or report API response evidence from current docs.

## References

Read [conversation-export-playbook.md](references/conversation-export-playbook.md) for export pagination, analysis outputs, and deletion planning templates.

Read [qa-and-golden-set.md](references/qa-and-golden-set.md) when using conversations to build a golden set, find KB gaps, evaluate user attributes, or plan knowledge improvements.

Read [feedback-replay-and-analysis.md](references/feedback-replay-and-analysis.md) for exact-event metadata RCA, separate rating/note/resolution writes, and replay, regression-testing, rule-evaluation, and analytics handoffs.
