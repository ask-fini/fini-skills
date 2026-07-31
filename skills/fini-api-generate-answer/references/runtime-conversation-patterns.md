# Runtime Conversation Patterns

Use this reference when starting or continuing conversations, passing caller attributes, executing an already-selected exact artifact set, or handling multi-turn context. Route systematic active-baseline versus unpublished-candidate comparisons to `fini-api-draft-validation`. Fetch current Generate Answer docs before relying on exact request fields.

## Contents

- [Start vs Continue](#start-vs-continue)
- [Active Configuration vs Exact Artifacts](#active-configuration-vs-exact-artifacts)
- [Role Rules](#role-rules)
- [Caller Attributes vs Configured Attributes](#caller-attributes-vs-configured-attributes)
- [Actions During Runtime](#actions-during-runtime)
- [Golden-Set Tests](#golden-set-tests)
- [Attachments](#attachments)
- [Output Template](#output-template)

## Start vs Continue

Start a new conversation when:

- The user wants a standalone bot test.
- There is no known `interactionId`.
- The user is testing a golden-set row independently.

Continue an existing conversation when:

- The user provides an `interactionId`.
- The test needs the actual prior thread.
- The user wants the new event to appear in the existing conversation.

If context matters and the caller provides an existing conversation ID, fetch the conversation first so you know what has already happened.

## Active Configuration vs Exact Artifacts

Omit `artifacts` when testing the agent exactly as currently published.

Use explicit artifact overrides when the proof must target review state:

- `promptVersionId` for one exact prompt version or draft
- `knowledgeSnapshotId` for one exact retrieval snapshot
- `knowledgeArticleDraftIds` for exact draft articles
- `ruleVersionIds` for exact Rulebook versions whose owning rules are already published and assigned to the target bot

Resolve and record every ID. Keep the combination intentional: testing one prompt draft against unrelated draft rules can make the result impossible to interpret. Overrides apply to this generation request; they do not publish those versions.

Persist and fetch each candidate through its owning workflow, then pass IDs only. Do not put raw prompt, article, or rule content inside `artifacts`. Use `fini-api-draft-validation` to establish an artifact-free baseline, test one dimension at a time, and run a combined candidate only after isolated results are understood.

A rule-version override does not make a new rule available to a bot. Collector accepts it only when the owning rule already has a published version assigned to that bot. A draft of that existing rule can be overridden; a brand-new unassigned rule draft cannot. For the latter, use structural field/flow review until a supported execution path exists.

## Role Rules

- Use `role=user` only for the customer/user turn that should trigger an AI reply.
- Use non-user roles only when intentionally storing events without triggering a new reply.
- Do not flatten a long transcript into one user message unless the caller explicitly wants a summarization-style test.
- Do not treat human-agent or third-party-bot messages as the latest customer request.

## Caller Attributes vs Configured Attributes

Use `metadata.user_attributes` for facts the caller already knows and wants Fini to consider.

Good examples:

- plan or subscription tier
- locale or region
- account status
- customer segment
- channel
- test-run marker

Rules:

- Keep keys stable.
- Keep values minimal.
- Do not invent fields.
- Do not include secrets or unnecessary sensitive data.
- Do not confuse these metadata values with configured Fini Attributes. Configured Attributes run their external API chains automatically for eligible messages on their assigned agents and configured channels, and can add fresh customer context before a reply.

When a configured Attribute matters:

1. Inspect the agent's Attribute assignments, including channel eligibility.
2. Verify the Data Step environment and test/customer identity.
3. Know that an eligible `role=user` turn can call those external systems.
4. Compare the returned behavior with the Attribute's expected visible-to-AI outputs.

## Actions During Runtime

Generate Answer executes the normal Rulebook path. If that path reaches a Tool node, its Action can call external APIs and mutate customer systems.

- Inspect reachable Tool nodes when the scenario may trigger an Action.
- Use controlled test accounts and values for effectful scenarios.
- Do not rely on Action assignment junctions; Rulebook wiring and rule-to-agent assignment control reachability.
- Do not describe a draft-artifact test as side-effect-free. An eligible overridden Rulebook version can also contain Tool nodes.
- Record observed Action results or state that external effects were not inspected.

## Golden-Set Tests

For each row:

1. Resolve the target `botId`.
2. Send one clear `role=user` question.
3. Include only the metadata needed for the scenario.
4. Set `markAsTest: true`.
5. Use the same artifact set for comparable cases and record it with the result.
6. Capture actual answer, event IDs, used articles, configured-integration effects, and whether the result should pass.
6. Fetch the full conversation only when persistence or audit evidence is needed.
7. Route failures by type: content gaps to Knowledge/Sources, wrong behavior config to Agent Configuration, missing caller context to user attributes, and external lookup/action failures to Actions/Attributes.

For broader regression coverage after publication, use a Test Set through `fini-api-testing`. Its asynchronous run evaluates persistent cases against current live-agent behavior and may invoke Actions or external APIs; it does not accept the unpublished artifact overrides used by Draft Validation.

## Attachments

Generate Answer does not accept public file attachments. Response events may include runtime-created attachment metadata, including expiring signed URLs; do not promise inbound screenshot, receipt, image, or file handling through this route.

## Output Template

```text
Runtime test
- Agent:
- New or continued conversation:
- Configuration: active / exact artifact IDs
- Caller user attributes sent:
- Configured Attributes expected/observed:
- Reachable Actions and effects:
- Test marker:
- Actual answer:
- Used articles:
- Pass/fail or manual review:
- Next action:
```
