# Agent Analytics

Use this reference to construct bounded agent analytics requests and interpret their results without overstating causality.

## Public Endpoints

| Operation | Method and path | Scope |
| --- | --- | --- |
| Full analytics | `GET /v2/bots/{id}/hc-analytics/public` | `read` |
| One section | `GET /v2/bots/{id}/hc-analytics/{section}/public` | `read` |

Both endpoints require the same agent ID, time window, and optional filters. They are not paginated.

## Build The Query

Require:

- `startEpoch`: window start as Unix epoch milliseconds;
- `endEpoch`: window end as Unix epoch milliseconds;
- `startEpoch < endEpoch`.

Optional filters can be scalar values or arrays:

| Filter | Meaning |
| --- | --- |
| `source` | Conversation source; `all` includes supported sources |
| `channel` | Conversation channel |
| `latestStatus` | Latest conversation status |
| `latestSentiment` | Latest sentiment |
| `usedSubfolderIds` | Knowledge folder IDs |
| `tagIds` | Tags |
| `ruleIds` | Rules |
| `escalationReasonTagIds` | Escalation-reason tags |
| `csatRatings` | Integer ratings from 1 through 5 |
| `timezone` | Timezone used for date grouping |

Use the smallest window that answers the question. State how human dates were converted to epoch milliseconds and which timezone governs grouping. Keep the exact window, timezone, and filters with the result so another run can reproduce it.

## Choose Full Or Sectioned Analytics

Use one section when possible:

- `summary`: current and comparison metrics, summary cards, status breakdown, and channel usage.
- `trends`: conversation volume, resolution rate, response time, hourly volume, and CSAT series.
- `knowledge`: knowledge-folder usage.
- `rules`: rule-level conversations, resolution, escalation, and CSAT metrics.
- `escalations`: escalation-reason counts.

Use full analytics when the task needs several of these together. Avoid fetching the full payload merely to answer one narrow metric question.

## Interpret Carefully

- Compare periods only when their duration, timezone, agent, and filters are equivalent.
- Distinguish counts, rates, durations, and ratings; do not compare unlike units.
- Treat absent or `null` CSAT as missing data, not a zero rating.
- Treat knowledge and rule analytics as associations. Inspect conversations before claiming a folder or rule caused an outcome.
- A filtered view is not the agent's global performance. Include all active filters in the summary.
- Preserve raw structured fields relevant to the question. Current documentation examples and field descriptions do not always show the same outer shape, so do not discard unfamiliar returned fields.
- For per-conversation evidence or root-cause analysis, route to `fini-api-conversations`.
- Use Draft Validation for one new-turn artifact comparison, or Test Suite for saved-case published/draft regression with stable inputs, criteria groups, and mocks. Analytics remains observational.

## Report

Include:

1. agent ID and name;
2. exact start and end timestamps plus timezone;
3. every active filter;
4. whether the result is full analytics or one section;
5. key metrics with units and denominators where available;
6. missing-data or comparability caveats;
7. the next evidence query when a causal conclusion needs conversation-level proof.
