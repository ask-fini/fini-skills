# Existing Agent Configuration Sequence

Use this reference when the user asks to configure or onboard an existing Slack/support agent using the Fini public APIs.

## Boundary

This sequence configures an existing Fini agent. Route agent creation to `fini-api-agents`. Workspace creation, Slack installation, and provider credentials remain separate unless `https://docs.usefini.com/llms.txt` exposes current routes for them.

## Recommended Sequence

1. Resolve the target agent with List agents.
2. Design tags:
   - routing/rulebook tags first
   - output/reporting tags second
   - verify Rulebook availability for routing tags
3. Draft rulebooks/rules:
   - create draft from behavior instructions
   - refine until reviewable
   - wire Actions through exact Tool-node references when needed
   - inspect test fields and evaluate with controlled context
   - publish only the explicit exact reviewed `draftVersionId` to explicit target agents; never infer a current/latest draft
   - keep Business Rules on their immediate-live, non-versioned lifecycle instead of using intent draft/refine/publish routes
4. Inspect and adjust prompts:
   - fetch current prompts and history
   - create a full-array draft
   - fetch the exact draft version and require a complete, non-truncated payload before review
   - publish the exact reviewed version; rebase instead of retrying if stale
   - use direct update only for an explicitly chosen immediate-live change
5. Add or refresh knowledge:
   - use `fini-api-sources` for raw/provider content
   - use `fini-api-knowledge` to create/verify a tree first if the KB is empty
   - use `fini-api-knowledge` for drafts, article-ID verification, publish, folders, and assignment
6. Run runtime tests:
   - use Generate Answer with explicit test metadata
   - use artifact overrides to test exact prompt and knowledge drafts before publication when appropriate
   - use a rule-version override only when its owning rule is already published and assigned to the target agent; a brand-new unassigned rule draft has no supported runtime override
   - account for Attributes eligible for the message's agent/channel assignment and Action Tool-node side effects
   - preserve returned event IDs and evidence fields
7. Inspect conversations:
   - use conversation exports for QA, golden-set candidates, and rollout evidence
8. Use `fini-api-draft-validation` for unpublished exact-ID candidates, `fini-api-testing` Replay for historical diagnosis, and Test Sets for persistent regression against current published live-agent behavior. Preflight Test Set Actions and external calls.

## Configuration Mental Model

Agent configuration changes how an existing agent behaves before or around knowledge retrieval:

- Prompts shape global instructions, tone, planning, and channel behavior.
- Rules/rulebooks shape routing, escalation, workflow branches, and structured behavior.
- Actions perform typed work only when a Rulebook Tool node invokes them.
- Attributes load context automatically for eligible messages on their assigned agents and configured channels.
- Tags/tag groups provide categories that rules, reporting, and output classification can depend on.
- Knowledge provides factual content. Do not use knowledge generation to fix prompt, rule, or tag problems.

If the agent gives a bad answer, decide whether the failure is behavior, content, visibility, or runtime context before changing configuration.

## Defaults

- Keep configuration writes reviewable: draft rules, prompt draft versions, tag design, draft knowledge.
- Use exact reviewed version IDs for prompt and intent-rule publication.
- Treat rule and runtime tests that can reach Actions as effectful.
- Separate behavior config failures from knowledge failures. Missing content usually belongs in Sources/Knowledge, while wrong routing or handling may belong in rules/prompts/tags.
- Use conversation evidence to improve either configuration or knowledge, not as a reason to blindly update both.

## Proof For Customer-Facing Summary

```markdown
Agent: [name/botId]
Tags: [not changed / designed / created / verified]
Rulebooks: [not changed / drafted / published / tested]
Action wiring: [not used / Tool nodes verified / effectful test performed]
Prompts: [not changed / read / drafted / published / directly updated / verified]
Knowledge: [not changed / sources processed / drafts generated / published / assigned]
Runtime test: [not run / pass / fail / needs reviewer]
Conversation QA: [not run / sample reviewed / golden set created]
Remaining approvals: [publish/delete/prompt/rule/tag/none]
```
