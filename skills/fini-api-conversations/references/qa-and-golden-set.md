# QA And Golden Set Workflows

Use this reference when conversations are being exported for QA, golden-set creation, regression testing, or knowledge improvement. Fetch current conversation docs first for exact response fields.

## Conversation Evidence Model

Conversations are evidence, not the fix. Use them to decide which workflow should happen next:

- Missing or stale facts -> Sources/Knowledge.
- Bad routing, escalation, or classification -> Agent Configuration.
- Missing user/account context -> Generate Answer metadata or caller integration.
- Good answer candidates -> golden-set regression rows.
- Delete requests -> destructive conversation cleanup only, not quality improvement.

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

## Fix-Review Iteration

Use fix review when one specific Fini answer needs a grounded proposed correction:

1. Fetch the conversation and identify the exact assistant event.
2. Create a fix-review iteration with concise feedback describing the observed failure and desired behavior.
3. Read the active or exact session and compare the original answer, replay answer, and recommendations.
4. Classify each recommendation as Knowledge, Sources, Prompt, Rule, Tag, or runtime metadata work.
5. Treat recommendations as proposals only. They do not update or publish Fini configuration.
6. Route the accepted recommendation to the owning skill, apply it through its normal approval gate, and rerun the same case.

Approving or rejecting the conversation event is a separate QA-state write. Require explicit confirmation and do not confuse event approval with applying the recommended fix.

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
