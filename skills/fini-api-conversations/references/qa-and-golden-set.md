# QA And Golden Set Workflows

Use this reference when conversations are being exported for QA, golden-set creation, regression testing, or knowledge improvement. Fetch current conversation docs first for exact response fields.

## Contents

- [Conversation Evidence Model](#conversation-evidence-model)
- [Build A Golden Set From Conversations](#build-a-golden-set-from-conversations)
- [Conversation Evidence To Knowledge Improvement](#conversation-evidence-to-knowledge-improvement)
- [Refine With AI Handoff](#refine-with-ai-handoff)
- [Replay And Regression Handoff](#replay-and-regression-handoff)
- [Golden Set Output Template](#golden-set-output-template)
- [QA Summary Template](#qa-summary-template)

## Conversation Evidence Model

Conversations are evidence, not the fix. Use them to decide which workflow should happen next:

- Missing or stale facts -> Sources/Knowledge.
- Bad routing, escalation, or classification -> Agent Configuration.
- Missing user/account context -> Generate Answer metadata or caller integration.
- Configured Attribute or external Data Step missing/incorrect -> `fini-api-actions-attributes`.
- Action not reached -> Agent Configuration Tool-node and rule assignment, then `fini-api-actions-attributes`.
- Action reached but failed -> `fini-api-actions-attributes`.
- Good answer candidates -> golden-set regression rows.
- One-off answer comparison -> replay through `fini-api-testing`.
- Repeatable coverage -> Test Set and criteria through `fini-api-testing`.
- Aggregate quality or performance question -> agent analytics through `fini-api-agents`.
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
   - rating, feedback note, event resolution, conversation resolution, and CSAT as separate signals
   - pass/fail reason
   - next action
4. Remove unnecessary personal data before sharing the golden set.
5. Hand selected eligible conversation IDs to `fini-api-testing` as persistent cases. A run evaluates current published live-agent behavior and may invoke Actions; it says nothing about a later configuration until that version is published and the same stable set is rerun.

## Conversation Evidence To Knowledge Improvement

Use this when the conversation shows a KB gap or stale answer:

1. Extract the smallest useful evidence from the conversation.
2. Decide whether this is missing knowledge, stale source content, bad assignment, prompt/rule/tag behavior, or runtime metadata.
3. Route missing/stale KB issues to `fini-api-knowledge`.
4. Route stale source issues to `fini-api-sources` first.
5. Route prompt, rulebook, routing, escalation, or tag classification issues to `fini-api-agent-configuration`.
6. Route configured Attribute, Data Step, or Action implementation failures to `fini-api-actions-attributes`; keep missing Tool-node reachability with Agent Configuration.
7. For a manually created draft, or when the Refine result needs additional clean-case or combined-candidate proof, use `fini-api-draft-validation` before publishing. Do not duplicate Refine's embedded replay for the original response; after publish, use marked Generate Answer for targeted active proof.

## Refine With AI Handoff

Use `fini-api-refine-with-ai` when one specific Fini answer needs AI diagnosis, persisted draft recommendations, and an improved replay:

1. Fetch the conversation and identify the exact Fini response event (`role=finibot`).
2. Preserve the observed failure, desired behavior, and QA state, then hand off to the Refine skill.
3. Let that skill own active-session reuse, confirmed iteration creation, polling, generated-draft lineage, and embedded replay comparison.
4. Route each exact returned article/prompt/rule draft to its owning skill for review and separate publication approval.

The event's thumbs rating, qualitative note, resolution flag, and Refine `feedbackNote` are separate writes. Do not confuse any of them with publishing a recommended fix.

## Replay And Regression Handoff

1. Preserve the original conversation ID and exact Fini response event ID.
2. Confirm the response has a linked user event before requesting a replay.
3. Record whether the user wants `single` to preserve prior history and regenerate only the target response, or `until` to regenerate every user turn through the linked target-user event.
4. Record whether validated model/artifact overrides are in scope. Default replay reuses original Attribute outputs and Rulebook results; exact rule artifacts can trigger Rulebook rerun behavior.
5. Hand replay creation, replay-event comparison, rule evaluation, and Test Set work to `fini-api-testing`.
6. Keep the original and replay IDs distinct. A replay is a separate conversation and is not automatically a Test Set case.
7. Validate an unpublished fix through `fini-api-draft-validation`, publish it exactly, then rerun the stable persistent Test Set against current live-agent behavior before declaring the regression resolved. Preflight Action/external effects.

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
