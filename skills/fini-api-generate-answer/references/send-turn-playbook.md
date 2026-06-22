# Send Turn Playbook

Use this reference for starting or continuing a Fini conversation through Generate Answer. Fetch current docs first for exact request and response fields.

## Runtime Mental Model

Generate Answer is the runtime proof harness. It creates conversation events and can trigger an AI reply for `role=user`, but it does not edit prompts, rules, tags, sources, or knowledge.

Use it to answer:

- Does this exact agent respond correctly now?
- Did the prompt/rule/tag/knowledge change affect behavior?
- Did the supplied user attributes or channel matter?
- Should the next fix go to configuration, knowledge, sources, or caller metadata?

## Start New Conversation

1. Resolve target agent.
2. Confirm environment: test, staging, or production.
3. Build one user message.
4. Add only intentional metadata.
5. Send with a user role when an AI response is desired.
6. Inspect returned events.
7. Fetch conversation if the user needs durable thread state.

## Continue Existing Conversation

1. Confirm `interactionId`.
2. Read the recent conversation first when context matters.
3. Send the new user event.
4. Inspect returned events.
5. Fetch the full conversation if persistence or audit evidence is needed.

## Metadata Guidance

Use metadata to represent facts the customer system already knows:

- Channel or surface.
- Customer/account identifiers.
- Plan, segment, locale, or other routing attributes.
- Test-run marker when generating synthetic traffic.

Do not invent metadata keys unless the user gives the downstream convention.

For user attributes:

- Keep field names stable across tests and production calls.
- Prefer small, business-relevant fields such as plan, locale, segment, account status, or channel when the caller supplies them.
- Avoid sensitive fields unless the customer explicitly confirms they are safe for Fini context.
- Do not use metadata as a place to hide instructions; it is context data.

## Safety Checks

Ask before sending when:

- The target agent may be production.
- The user supplied an agent name but not a confirmed ID.
- The request is a load test or repeated send.
- The message contains sensitive customer data and the workspace is unclear.

## Config Or Knowledge Change Proof

Use Generate Answer as a proof step after another workflow changes agent behavior:

- Prompt, rulebook, or tag changes -> route setup to `fini-api-agent-configuration`, then test here.
- Source or knowledge changes -> route setup to `fini-api-sources` or `fini-api-knowledge`, then test here.
- User attributes or channel behavior -> send only the minimal metadata needed for the scenario.

If the answer is wrong, classify the likely failure before proposing a fix:

- Missing/stale content -> Knowledge or Sources.
- Wrong routing, escalation, tone, or rule behavior -> Agent Configuration.
- Missing personalization context -> Generate Answer metadata/user attributes.
- Customer-owned action not supported by public Fini docs -> integration gap, not KB/config.

## Response Interpretation

Do not assume the first returned item is the final answer. Read all created events and summarize:

- Which events were user vs assistant/system/tool.
- Which event has the answer text.
- Whether evidence/citations/used knowledge appeared.
- Whether the response indicates handoff, fallback, or failure.
- Whether user attributes appear relevant to the answer when testing personalization.

## Output Template

```text
Generate Answer result
- Target agent:
- New or existing conversation:
- Sent role:
- Events created:
- Assistant reply:
- Evidence/used articles:
- Likely follow-up workflow:
- Full conversation fetched:
- Caveats:
```
