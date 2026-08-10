---
name: fini-api-slack-onboarding
description: "Use only when the user explicitly wants full Slack-led onboarding inside an existing authenticated Fini workspace: resolve or create a bare agent, configure behavior, ingest and assign knowledge, optionally add Actions and Attributes, validate review drafts, publish approved configuration, run a persistent Test Set against live-agent behavior as the go-live gate, and report readiness. First inspect workspace state; route one-off agent, source, knowledge, prompt, Rulebook, integration, analytics, conversation, or testing requests to the narrower Fini skill."
---

# Fini API Slack Onboarding

Conduct a complete Slack-guided onboarding by coordinating the narrower Fini API skills. Do not replace their domain workflows or duplicate endpoint schemas here.

Before endpoint-specific work, fetch `https://docs.usefini.com/llms.txt` and the relevant linked docs. Treat the current public docs as route, field, enum, limit, and response-shape truth.

## Boundary

- Work inside an existing authenticated Fini workspace.
- Create a bare agent through `fini-api-agents` when the onboarding needs one and no suitable agent exists.
- Keep bare-agent creation separate from workspace creation, Slack/channel installation, provider OAuth, prompts, knowledge, and integrations. Those are not implied by the returned agent ID.
- Use dashboard or user-owned setup for Slack installation and provider OAuth when the live public docs expose no route.
- Route targeted changes such as one source import, prompt edit, analytics query, Action update, or replay to the focused skill instead.

## Classify Workspace State

For an ambiguous request such as "set up my Fini bot and KB," inspect before interviewing:

1. List agents and identify likely targets.
2. Inspect articles/drafts, knowledge folders, and sources.
3. Inspect current prompts, rules, tags, Actions, Attributes, and assignments when relevant to the requested use case.
4. Sanity-check whether agent, article, source, folder, and assignment state agree.

Classify:

| Classification | Meaning | Next step |
| --- | --- | --- |
| Fresh onboarding | No suitable agent or minimal content/configuration | Continue this skill; create a bare agent if needed |
| Existing-client expansion | Meaningful setup exists but the launch scope is broad | Confirm the target and continue only with missing stages |
| Targeted update | One focused outcome | Route to the owning skill |

Treat a stale or contradictory folder snapshot as a blocker to knowledge work, not as proof that the target agent can use the KB.

## Collect Intake

Offer:

- All at once: accept a Markdown/PDF/doc bundle, help center, SOPs, behavior notes, automation requirements, required customer context, test questions, examples, and launch constraints.
- Step by step: collect one short section at a time and maintain onboarding state.

Read [intake-and-state.md](references/intake-and-state.md) for the intake order, state record, and completion format.

## Core Workflow

1. **Agent**
   - Use `fini-api-agents` to resolve an existing target.
   - If none exists, create a bare agent with the exact requested name and preserve its returned ID.
   - Record Slack/channel-installation status separately.
2. **Prerequisites**
   - Confirm provider OAuth, source access, customer API ownership, and other user-managed prerequisites.
3. **Knowledge tree**
   - Use `fini-api-knowledge` to verify a usable tree and create or initialize one when absent before source-backed knowledge generation.
4. **Sources**
   - Use `fini-api-sources` to crawl, register, ingest, refresh, inspect, and poll source records.
5. **Knowledge**
   - Generate drafts, require persisted article/draft IDs, review, publish, organize, and assign folders to the target agent.
6. **Behavior**
   - Use `fini-api-agent-configuration` for tags, prompt drafts, and intent Rulebook drafts; publish only explicitly reviewed exact version IDs. Never infer a current/latest rule draft during publish. Treat Business Rules as immediate-live and non-versioned.
7. **Optional customer context and operations**
   - Use `fini-api-actions-attributes` only when requirements need them.
   - Use Attributes for context fetched automatically before eligible message generations for assigned agents/channels.
   - Use Actions for operations invoked through a published Rulebook Tool node.
   - Configure ordered external API calls and keep stored secrets masked.
8. **Draft validation and publish**
   - Use `fini-api-draft-validation` for an artifact-free baseline, one-dimension-at-a-time exact draft candidates, and an intentional combined candidate when pre-publish proof is required.
   - Publish only separately approved exact prompt, knowledge, and intent-rule versions through their owning skills.
9. **Targeted active smoke**
   - Use `fini-api-generate-answer` for a small controlled proof of active behavior after publication. Through MCP, set its `markAsTest: true`; direct REST exposes no top-level field by that name.
10. **Go-live regression gate**
   - Use `fini-api-testing` to create or update a persistent Test Set, attach criteria, run it against current published live-agent behavior, poll to a terminal result, and classify every blocking failure.
   - Preflight Action, Attribute, and external-API effects because the Test Set run can invoke them.
11. **Rollout evidence**
   - Use `fini-api-conversations` for exact transcript/feedback evidence and `fini-api-agents` for aggregate analytics.

## Mutation Guidance

- Read exact current state and dependencies before changing or deleting resources.
- Prefer reviewable prompt, Rulebook, and knowledge drafts when a draft path exists.
- If the user already provided explicit execution intent and exact targets, proceed without asking for duplicate approval.
- If the target or intended live effect remains ambiguous, state the effect and ask one concise question.
- Treat Action/external-call tests, rule evaluation, Replay, Generate Answer, and Test Set runs as executions when they can reach customer APIs. Use inputs and targets where the resulting real effect is acceptable.
- For historical replays, remember that `until` can regenerate multiple user turns through the target event, while `single` regenerates only the target response against preserved prior history.
- By default, replay reuses original Attribute outputs and seeded Rulebook results unless exact prompt or rule artifacts cause the Rulebook to rerun.
- Never expose, reconstruct, or log stored Action/Attribute secrets. Preserve a masked value only when the public contract defines it as retaining the stored secret.

## Hard Gates

- **Agent gate:** do not continue with agent-scoped configuration until the exact `botId` is resolved or created.
- **Usable-tree gate:** as an operational empty-workspace safeguard, verify usable folders and create or initialize them when absent before source-backed knowledge generation. Raw source import may happen first.
- **Job-result gate:** require an article/draft ID; `COMPLETED` without one is a no-op or blocker.
- **Snapshot gate:** stop when the folder snapshot contradicts current agent/article/source state.
- **Visibility gate:** require exact folder-agent assignment and agent-scoped visibility proof.
- **Action reachability gate:** require a reviewed published Rulebook Tool node and the explicit exact `draftVersionId` used to publish it; Action assignment alone is insufficient.
- **Attribute assignment gate:** verify the exact agent/channel and eligible-message behavior before relying on the Attribute.
- **Draft-validation gate:** when review drafts need behavioral proof, require persisted exact IDs, an artifact-free baseline, isolated candidates, and an explicit no-publish conclusion.
- **Test Set gate:** do not mark a full onboarding ready without a terminal persistent regression run against current published live-agent behavior and a disposition for blocking failures.
- **Integration-effect gate:** before Generate Answer, rule evaluation, Replay, integration tests, or Test Set runs, verify that real external effects and repeat counts are acceptable.

## Build The Go-Live Test Set

Define 4-8 representative cases:

- common support questions
- source-specific factual questions
- out-of-scope or escalation behavior
- tone/personality
- caller metadata or Attribute-driven personalization when applicable
- Action success, denial, failure, and escalation cases when applicable
- edge cases from historical bad answers

After the intended prompt, Rulebook, knowledge, Attribute, and Action configuration is published:

1. Select eligible representative conversations/cases and verify the target agent, inputs, and expected behavior.
2. Create or update the persistent Test Set and add deterministic and/or judge criteria that express launch requirements.
3. Re-fetch current published configuration and inspect every case for reachable Actions, configured Attributes, external calls, and repeated-effect risk.
4. Use controlled accounts and acceptable targets for Action-bearing cases.
5. Start one live-agent run and poll until completed or failed.
6. Inspect per-case, per-criterion, generated-behavior, and Action/external-effect evidence.
7. Route each failure to the owning skill, make the smallest draft change, validate it before publish when needed, publish exactly, then rerun the same stable Test Set.

The Test Set evaluates current published live-agent behavior and may invoke Actions or customer APIs. It does not accept the request-scoped unpublished artifact overrides used by `fini-api-draft-validation`.

## Failure Routing

| Failure | Fix owner |
| --- | --- |
| Missing/stale facts | Sources -> Knowledge |
| Correct facts, wrong style/instructions | Agent Configuration prompt draft |
| Wrong routing/escalation | Agent Configuration Rulebook |
| Action never reached | Agent Configuration Tool-node wiring, then Actions/Attributes |
| Action/Attribute input, external call, or mapping failure | Actions/Attributes |
| Missing eligible-message customer context | Actions/Attributes Attribute workflow |
| Missing caller-owned one-turn context | Generate Answer metadata |
| Wrong category/reporting output | Agent Configuration tags and rules |

## Completion Proof

End with:

```markdown
Onboarding classification: [fresh / expansion / targeted]
Workspace/auth: [ready / blocked]
Agent: [existing / created] [name / botId]
Slack/channel installation: [ready / user action / blocked]
Sources: [status and IDs]
Knowledge: [draft/live IDs, folders, assignment]
Prompts: [draft/reviewed/published version]
Rules/tags: [draft/published/verified IDs]
Attributes: [not needed / IDs, assignments, proof]
Actions: [not needed / IDs, Rulebook reachability, proof]
Test Set: [ID, run ID, terminal status, blocking results]
Analytics/rollout evidence: [not checked / summary]
Go-live blockers: [list / none]
Next action: [one concrete step]
```
