# Conversation Export Playbook

Use this reference for conversation export, analysis, and deletion planning. Fetch current docs first for exact parameters and response fields.

## Export Procedure

1. Define purpose: QA, analytics, BI, compliance, support review, billing, or RCA.
2. Define scope: workspace, agent, source/channel, customer attributes, and time window.
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

## Pagination Rules To Remember

- Results are newest-first.
- `direction=next` moves older.
- `direction=previous` moves newer.
- A cursor should be treated as opaque.
- The documented throttle is 100 requests per 60 seconds. If a request returns `429`, retry the same cursor with bounded backoff rather than skipping a page.
- Avoid per-conversation follow-up reads when the list payload already contains the evidence required by the task.
- For long exports, record the final cursor and the last conversation timestamp.

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
- Resolution/feedback:
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
- Current docs warning for partial/atomic behavior.
- Expected irreversibility.
- Confirmation phrase requested from user.

Execution summary should include:

- Route used.
- IDs attempted.
- IDs confirmed deleted or response evidence.
- Any IDs not found, inaccessible, or ambiguous.
