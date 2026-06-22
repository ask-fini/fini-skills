# Improvement Loop

Use this reference when a conversation, golden-set row, support-review finding, or QA export suggests that Fini knowledge needs to change. Fetch current Knowledge docs first for exact generation, draft, publish, and assignment routes.

## Classify The Failure

Before changing knowledge, decide which failure type applies:

- Missing KB content.
- Stale source content.
- Article exists but is not assigned to the target agent.
- Article exists but is in the wrong folder.
- Runtime metadata/user attributes were missing.
- Prompt, rulebook, routing, escalation, or tag classification was wrong.
- The expected answer should be handled by product, policy, or human escalation rather than KB.

Route accordingly:

- Missing KB -> generate draft knowledge.
- Stale source -> refresh source first, then generate draft updates.
- Visibility issue -> folder/article assignment workflow.
- Metadata issue -> Generate Answer runtime pattern.
- Prompt/rule/tag behavior issue -> Agent Configuration workflow.
- Out-of-scope business action -> do not create KB that promises unsupported actions.

## Draft Fix Workflow

1. Preserve the failing question and expected answer/reference.
2. Extract the smallest useful evidence from the conversation or review note.
3. Generate draft knowledge by default.
4. Poll the job and inspect draft/no-op/failure state.
5. Review the draft content and folder placement.
6. Publish only after explicit approval or prior authorization.
7. Verify the article is in a folder visible to the intended agent.
8. Retest the failed question with Generate Answer.

## Output Template

```text
Knowledge improvement
- Failing question:
- Failure type:
- Evidence source:
- Draft/article IDs:
- Folder:
- Agent visibility:
- Retest result:
- Publish decision:
- Remaining risk:
```

## Safety Rules

- Do not publish directly from ambiguous evidence.
- Do not update live knowledge when the expected answer is not confirmed.
- Do not create KB entries that claim Fini can perform customer-owned business actions unless the customer and current public API support that workflow.
- Do not treat a passing retest as proof that every related question is fixed; summarize the scope tested.
