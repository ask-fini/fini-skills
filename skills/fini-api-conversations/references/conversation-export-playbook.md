# Conversation Export Playbook

Use this reference for conversation export, analysis, and deletion planning. Fetch current docs first for exact parameters and response fields.

## Contents

- [Export Procedure](#export-procedure)
- [Pagination Rules To Remember](#pagination-rules-to-remember)
- [Choose Export Or Analytics](#choose-export-or-analytics)
- [Analysis Output Template](#analysis-output-template)
- [RCA Evidence Template](#rca-evidence-template)
- [Deletion Planning](#deletion-planning)

## Export Procedure

1. Define purpose: QA, analytics, BI, compliance, support review, billing, or RCA.
2. Define scope: workspace, agent, source/channel, CSAT filter, customer attributes, and time window.
3. Resolve `botId` if the scope uses agent names.
4. Start with a small sample query to verify filters and fields.
5. Request at most 100 records per page. Treat that as a page-size ceiling, not a total-result ceiling.
6. Page until `hasMore=false` or an explicit user bound is reached. Use `nextCursor` with `direction=next` to move older.
7. Process or persist each page incrementally. Do not hold an unbounded transcript export in agent context.
8. Preserve source IDs, page count, final cursor, and last timestamp in any durable or resumable export.
9. Summarize what was included and what was intentionally excluded. Mark any bounded, interrupted, or repeatedly rate-limited run as incomplete.

For QA exports, also preserve:

- Conversation ID and event IDs.
- Status, resolved flag, feedback, approval, and CSAT fields when present.
- User attributes used for context.
- Used articles and used folders/subfolders.
- Event role, type, message, and timestamp.
- Source/channel filters and bot ID.

Keep these event quality signals separate:

- `approved`: thumbs up (`true`), thumbs down (`false`), or unrated (`null`).
- `feedback`: qualitative teammate note, which may be empty or absent independently of the rating.
- event `resolved`: whether that event's negative feedback has been handled.
- conversation `resolved`: aggregate state recalculated from negatively rated Fini events.
- `csatRating`: channel-provided numeric CSAT; do not assume `0` means missing.

## Pagination Rules To Remember

- Results are newest-first.
- `direction=next` moves older.
- `direction=previous` moves newer.
- A cursor should be treated as opaque.
- The documented throttle is 100 requests per 60 seconds. If a request returns `429`, retry the same cursor with bounded backoff rather than skipping a page.
- Avoid per-conversation follow-up reads when the list payload already contains the evidence required by the task.
- For long exports, record the final cursor and the last conversation timestamp.

## Choose Export Or Analytics

Use conversation export when the task needs transcripts, event evidence, user attributes, article usage, or a review cohort.

Use `fini-api-agents` analytics when the task asks for aggregate performance summaries, trends, knowledge usage, rule performance, escalation reasons, response time, resolution rate, or CSAT over a time window. Preserve the analytics filters and then return to conversation export only for representative or diagnostic evidence.

## Analysis Output Template

```text
Conversation export
- Purpose:
- Filters:
- Time window:
- Agent/source/channel:
- Conversations returned:
- Pages processed:
- Pagination status:
- Final cursor / last timestamp:
- Complete or incomplete:
- Fields used for analysis:
- Notable schema caveats:
- Follow-up data needed:
```

## RCA Evidence Template

```text
Conversation evidence
- Conversation ID:
- Timestamp:
- Agent:
- User issue:
- Relevant user turns:
- Relevant assistant turns:
- Used articles or cited knowledge:
- User attributes:
- Rating / note / event resolution / conversation resolution:
- CSAT:
- Tags/outcome:
- Interpretation:
- Confidence:
```

## Deletion Planning

Deletion is destructive. Do not execute from a vague request like "clean these up."

Before deleting, produce:

- Exact conversation IDs.
- Count.
- How IDs were selected.
- Whether this is single delete or bulk delete.
- For bulk delete, the disclosed batch count and exact boundaries; each request accepts 1-50 unique IDs.
- Current docs warning for partial/atomic behavior.
- Expected irreversibility.
- Confirmation phrase requested from user.

Execution summary should include:

- Route used.
- IDs attempted.
- Batch number and boundaries when the plan required more than one request.
- IDs confirmed deleted or response evidence.
- Any IDs not found, inaccessible, or ambiguous.

If one batch fails after earlier batches succeeded, stop and reconcile completed versus remaining IDs before retrying or continuing. Do not silently recompute the target set after authorization.
