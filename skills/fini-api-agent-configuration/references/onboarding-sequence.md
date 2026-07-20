# Existing Agent Configuration Sequence

Use this reference when the user asks to configure or onboard an existing Slack/support agent using the Fini public APIs.

## Boundary

This sequence configures an existing Fini agent. Do not claim the public APIs can create a new workspace, install Slack, or provision a brand-new agent unless `https://docs.usefini.com/llms.txt` exposes current routes for that.

## Recommended Sequence

1. Resolve the target agent with List agents.
2. Design tags:
   - routing/rulebook tags first
   - output/reporting tags second
   - verify Rulebook availability for routing tags
3. Draft rulebooks/rules:
   - create draft from behavior instructions
   - refine until reviewable
   - publish only to explicit target agents
4. Inspect and adjust prompts:
   - fetch current prompts and history
   - update full prompt arrays only after approval
   - re-fetch after saving
5. Add or refresh knowledge:
   - use `fini-api-sources` for raw/provider content
   - use `fini-api-knowledge` to create/verify a tree first if the KB is empty
   - use `fini-api-knowledge` for drafts, article-ID verification, publish, folders, and assignment
6. Run runtime tests:
   - use Generate Answer with explicit test metadata
   - preserve returned event IDs and evidence fields
7. Inspect conversations:
   - use conversation exports for QA, golden-set candidates, and rollout evidence

## Configuration Mental Model

Agent configuration changes how an existing agent behaves before or around knowledge retrieval:

- Prompts shape global instructions, tone, planning, and channel behavior.
- Rules/rulebooks shape routing, escalation, workflow branches, and structured behavior.
- Tags/tag groups provide categories that rules, reporting, and output classification can depend on.
- Knowledge provides factual content. Do not use knowledge generation to fix prompt, rule, or tag problems.

If the agent gives a bad answer, decide whether the failure is behavior, content, visibility, or runtime context before changing configuration.

## Defaults

- Keep configuration writes reviewable: draft rules, prompt plan, tag design, draft knowledge.
- Publish or delete only after explicit approval.
- Separate behavior config failures from knowledge failures. Missing content usually belongs in Sources/Knowledge, while wrong routing or handling may belong in rules/prompts/tags.
- Use conversation evidence to improve either configuration or knowledge, not as a reason to blindly update both.

## Proof For Customer-Facing Summary

```markdown
Agent: [name/botId]
Tags: [not changed / designed / created / verified]
Rulebooks: [not changed / drafted / published / tested]
Prompts: [not changed / read / updated / verified]
Knowledge: [not changed / sources processed / drafts generated / published / assigned]
Runtime test: [not run / pass / fail / needs reviewer]
Conversation QA: [not run / sample reviewed / golden set created]
Remaining approvals: [publish/delete/prompt/rule/tag/none]
```
