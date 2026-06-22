# QA And Golden Set Workflows

Use this reference when conversations are being exported for QA, golden-set creation, regression testing, or knowledge improvement. Fetch current conversation docs first for exact response fields.

## Build A Golden Set From Conversations

1. Export a bounded sample with explicit time window and filters.
2. Select conversations with clear user questions, answer quality issues, missing knowledge, incorrect used articles, negative feedback, low CSAT, unresolved status, or escalation.
3. For each candidate, preserve:
   - question
   - expected answer
   - expected knowledge reference when known
   - actual answer
   - used articles
   - user attributes needed for the scenario
   - pass/fail reason
   - next action
4. Remove unnecessary personal data before sharing the golden set.

## Conversation Evidence To Knowledge Improvement

Use this when the conversation shows a KB gap or stale answer:

1. Extract the smallest useful evidence from the conversation.
2. Decide whether this is missing knowledge, stale source content, bad assignment, prompt/rule/tag behavior, or runtime metadata.
3. Route missing/stale KB issues to `fini-api-knowledge`.
4. Route stale source issues to `fini-api-sources` first.
5. Route prompt, rulebook, routing, escalation, or tag classification issues to `fini-api-agent-configuration`.
6. Retest with `fini-api-generate-answer` before publishing if the change affects live behavior.

## Golden Set Output Template

```text
Golden-set candidate
- Conversation ID:
- User question:
- Expected answer:
- Expected knowledge reference:
- Actual answer:
- Used articles:
- User attributes needed:
- Pass/fail:
- Failure type:
- Recommended workflow:
```

## QA Summary Template

```text
Conversation QA summary
- Window:
- Filters:
- Conversations reviewed:
- High-confidence passes:
- KB gaps:
- Stale source candidates:
- Assignment/visibility issues:
- Metadata/user-attribute issues:
- Prompt/rule/tag config issues:
- Escalation/handoff signals:
- Next workflow:
```
