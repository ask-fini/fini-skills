# Runtime Conversation Patterns

Use this reference when starting or continuing conversations, replaying tests, passing user attributes, or handling multi-turn context. Fetch current Generate Answer docs before relying on exact request fields.

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

## Role Rules

- Use `role=user` only for the customer/user turn that should trigger an AI reply.
- Use non-user roles only when intentionally storing events without triggering a new reply.
- Do not flatten a long transcript into one user message unless the caller explicitly wants a summarization-style test.
- Do not treat human-agent or third-party-bot messages as the latest customer request.

## User Attributes

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
- If an attribute is needed only to fetch data from the customer's own API, that is outside this public bot/KB workflow unless current Fini docs expose a supported route.

## Golden-Set Tests

For each row:

1. Resolve the target `botId`.
2. Send one clear `role=user` question.
3. Include only the metadata needed for the scenario.
4. Capture actual answer, event IDs, used articles, and whether the result should pass.
5. Fetch the full conversation only when persistence or audit evidence is needed.

## Attachments

Verify current docs before relying on public attachment support. If the docs say Generate Answer is text-only, do not promise screenshot, receipt, image, or file handling through this route.

## Output Template

```text
Runtime test
- Agent:
- New or continued conversation:
- User attributes sent:
- Test marker:
- Actual answer:
- Used articles:
- Pass/fail or manual review:
- Next action:
```
