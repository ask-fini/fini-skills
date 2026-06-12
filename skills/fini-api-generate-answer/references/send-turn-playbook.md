# Send Turn Playbook

Use this reference for starting or continuing a Fini conversation through Generate Answer. Fetch current docs first for exact request and response fields.

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

## Safety Checks

Ask before sending when:

- The target agent may be production.
- The user supplied an agent name but not a confirmed ID.
- The request is a load test or repeated send.
- The message contains sensitive customer data and the workspace is unclear.

## Response Interpretation

Do not assume the first returned item is the final answer. Read all created events and summarize:

- Which events were user vs assistant/system/tool.
- Which event has the answer text.
- Whether evidence/citations/used knowledge appeared.
- Whether the response indicates handoff, fallback, or failure.

## Output Template

```text
Generate Answer result
- Target agent:
- New or existing conversation:
- Sent role:
- Events created:
- Assistant reply:
- Evidence/used articles:
- Full conversation fetched:
- Caveats:
```
