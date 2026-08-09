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

Mental shortcut: knowledge fixes facts and source truth. It does not fix routing, escalation policy, tone, prompt instructions, or missing runtime metadata.

Route accordingly:

- Missing KB -> generate draft knowledge.
- Stale source -> refresh source first, then generate draft updates.
- Visibility issue -> folder/article assignment workflow.
- Metadata issue -> Generate Answer runtime pattern.
- Prompt/rule/tag behavior issue -> Agent Configuration workflow.
- Configured Attribute, Data Step, or Action failure -> `fini-api-actions-attributes`.
- Action not reached -> Agent Configuration Tool-node and rule assignment, then `fini-api-actions-attributes`.
- Requested business operation with no configured/documented Action path -> do not create KB that promises the operation; assess it through `fini-api-actions-attributes`.

## Draft Fix Workflow

1. Preserve the failing question and expected answer/reference.
2. If it comes from one exact eligible bad Fini response and the user wants AI-assisted correction, use `fini-api-refine-with-ai` first.
3. When Refine returns a `draftArticleId`, fetch and review that exact draft here; do not generate a duplicate.
4. Verify the article's factual support and folder placement. The improved Refine replay is behavioral evidence, not automatic factual proof.
5. Use the manual generation path only when no eligible response exists or the user explicitly requests direct authoring.
6. Use `fini-api-draft-validation` for additional fresh cases or combined candidates when needed.
7. Publish only after explicit approval or prior authorization and satisfactory exact-draft review.
8. Verify the article is in a folder visible to the intended agent.
9. Rerun the same question without draft artifacts to prove active behavior.

## Output Template

```text
Knowledge improvement
- Failing question:
- Failure type:
- Evidence source:
- Draft/article IDs:
- Draft-artifact retest:
- Folder:
- Agent visibility:
- Active retest result:
- Publish decision:
- Remaining risk:
```

## Safety Rules

- Do not publish directly from ambiguous evidence.
- Do not update live knowledge when the expected answer is not confirmed.
- Do not create KB entries that claim a customer-owned business operation will run unless a configured and tested Fini Action/Rulebook workflow supports it.
- Do not treat a passing retest as proof that every related question is fixed; summarize the scope tested.
