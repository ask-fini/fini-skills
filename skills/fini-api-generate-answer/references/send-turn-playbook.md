# Send Turn Playbook

Use this reference for starting or continuing a Fini conversation through Generate Answer. Fetch current docs first for exact request and response fields.

## Contents

- [Runtime Mental Model](#runtime-mental-model)
- [Start New Conversation](#start-new-conversation)
- [Continue Existing Conversation](#continue-existing-conversation)
- [Artifact Proof](#artifact-proof)
- [Metadata Guidance](#metadata-guidance)
- [Runtime Effects](#runtime-effects)
- [Configuration Or Knowledge Proof](#configuration-or-knowledge-proof)
- [Response Interpretation](#response-interpretation)
- [Output Template](#output-template)

## Runtime Mental Model

Generate Answer is the one-turn runtime proof harness. It creates conversation events and can trigger the normal AI reply flow for `role=user`, but it does not publish prompts, rules, tags, sources, or knowledge.

Use it to answer:

- Does this exact agent respond correctly now?
- Does this exact set of prompt/rule/knowledge artifacts behave correctly before publication?
- Did supplied user attributes, configured Attributes, channel, or reachable Actions matter?
- Should the next fix go to configuration, Actions/Attributes, knowledge, sources, or caller metadata?

Do not use it for aggregate regression analysis. Route an active-baseline versus unpublished-candidate sequence to `fini-api-draft-validation`. Use a Replay for historical regeneration: `single` preserves prior history and regenerates only the target response, while `until` regenerates every user turn through the linked target-user event. Default replay reuses original Attribute outputs and Rulebook results unless exact artifacts trigger Rulebook rerun behavior. Use Test Suite cases and criteria groups for persistent published/draft regression with simulated Actions and captured Attribute context.

## Start New Conversation

1. Resolve target agent.
2. Confirm environment: test, staging, or production.
3. Build one user message.
4. Choose active configuration or resolve exact artifact overrides.
5. Add only intentional metadata.
6. Through MCP, set the connector's `markAsTest: true` for synthetic traffic. Do not send a top-level field by that name to direct REST.
7. Inspect configured Attribute assignments and Action-bearing Rulebooks when their effects matter.
8. Send with a user role when an AI response is desired.
9. Inspect all returned events.
10. Fetch the conversation if the user needs durable thread state.

## Continue Existing Conversation

1. Confirm `interactionId`.
2. Read the recent conversation first when context matters.
3. Choose active configuration or exact artifact overrides; do not assume the previous turn used the same versions.
4. Inspect runtime integration effects when relevant.
5. Send the new user event.
6. Inspect returned events.
7. Fetch the full conversation if persistence or audit evidence is needed.

## Artifact Proof

Use `artifacts` only for versions the caller intends to test:

```text
promptVersionId: one exact prompt version or draft
knowledgeSnapshotId: one exact knowledge snapshot
knowledgeArticleDraftIds: zero or more exact draft article IDs
ruleVersionIds: zero or more exact Rulebook versions whose owning rules are already available to the target bot
```

Before sending:

1. Resolve each ID from its owning workflow.
2. Fetch every persisted candidate exactly and reject missing, inferred, or cross-workspace IDs.
3. Confirm all IDs belong to the intended workspace and scenario.
4. Pass IDs only; do not inline prompt, knowledge, or rule content in `artifacts`.
5. Record the full artifact set as part of the test case.
6. Route baseline, isolated-dimension, and combined-candidate comparisons to `fini-api-draft-validation`.
7. Verify whether each override replaces or composes with active state when isolation matters; do not infer unspecified merge behavior.

The override is request-scoped and does not publish configuration. It still creates conversation/events and runs the normal runtime, including configured Attributes and Action Tool nodes. The MCP connector's `markAsTest: true` is a traffic marker, not a dry-run or Action-suppression flag.

`ruleVersionIds` is not a way to attach a brand-new rule to a bot. The owning rule must already have a published version assigned to the target bot. A draft version of that existing rule can be tested as an override; a brand-new unassigned draft is limited to structural preview/review until it has a supported executable path.

## Metadata Guidance

For direct REST, send `metadata.channel` as `chat` or `email`. The MCP tool exposes it as top-level `channel` and maps it to the REST field. Do not put channel in `metadata.user_attributes`.

Use user attributes to represent facts the customer system already knows:

- Customer/account identifiers.
- Plan, segment, locale, or other routing attributes.
- Business metadata needed by the test case. Through MCP, use its top-level `markAsTest: true` input for synthetic traffic; direct REST exposes no top-level field by that name.

Do not invent metadata keys unless the user gives the downstream convention.

For user attributes:

- Keep field names stable across tests and production calls.
- Prefer small, business-relevant fields such as plan, locale, segment, or account status when the caller supplies them.
- Avoid sensitive fields unless the customer explicitly confirms they are safe for Fini context.
- Do not use metadata as a place to hide instructions; it is context data.

Caller `metadata.user_attributes` are supplied values. Configured Attributes are API-backed lookups that can run automatically before eligible replies on their assigned agents and configured channels. Keep those two evidence sources distinct.

## Runtime Effects

Before a `role=user` proof:

- Resolve the exact production/test agent instead of relying on a name guess.
- Inspect assigned Attributes and their channel eligibility when they can call customer APIs or influence the expected answer.
- Inspect published or overridden Rulebooks for Tool nodes the message can reach.
- Use controlled accounts and inputs when an Action can mutate an external system.
- Remember that retrying can duplicate events and possibly repeat external effects.
- Record observed Attribute/Action outcomes or state that they were not inspected.

Action assignment junctions do not gate Action execution. The Rulebook Tool-node reference and that rule's agent assignments control reachability.

## Configuration Or Knowledge Proof

Use Generate Answer as a proof step after another workflow prepares or changes agent behavior:

- Prompt, Rulebook, or tag changes -> route setup to `fini-api-agent-configuration`; test exact prompt drafts and eligible rule-version drafts here before publication, then test active configuration afterward.
- Action or Attribute changes -> route setup to `fini-api-actions-attributes`, then use a controlled runtime proof here.
- Source or knowledge changes -> route setup to `fini-api-sources` or `fini-api-knowledge`, then test here.
- User attributes or channel behavior -> send only the minimal attributes plus the separate channel field needed for the scenario.

If the answer is wrong, classify the likely failure before proposing a fix:

- Missing/stale content -> Knowledge or Sources.
- Wrong routing, escalation, tone, or rule behavior -> Agent Configuration.
- Missing personalization context -> Generate Answer metadata/user attributes.
- Attribute lookup or Data Step failure -> Actions/Attributes configuration.
- Action not firing -> verify published Rulebook Tool-node wiring and rule assignment before changing the prompt.
- Action called but failed -> inspect its typed inputs, Data Steps, and output mappings.

## Response Interpretation

Do not assume the first returned item is the final answer. Read all created events and summarize:

- Which events were user versus the Fini response (`role=finibot`) or other returned event roles.
- Which event has the answer text.
- Whether evidence/citations/used knowledge appeared.
- Whether the response indicates handoff, fallback, or failure.
- Whether caller attributes or configured Attributes appear relevant to the answer.
- Whether any Action result or external effect was observed.

The response is event evidence for one submission, not a Test Suite criterion result. Route aggregate pass/fail evidence to `fini-api-testing`.

## Output Template

```text
Generate Answer result
- Target agent:
- New or existing conversation:
- Configuration: [active / exact artifact IDs]
- Marked as test:
- Sent role:
- Caller user attributes:
- Configured Attributes:
- Reachable Actions/effects:
- Events created:
- Fini reply:
- Evidence/used articles:
- Likely follow-up workflow:
- Full conversation fetched:
- Caveats:
```
