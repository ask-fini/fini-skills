---
name: fini-api-generate-answer
description: Use when the user wants to send a message turn into a Fini agent through the public API, start or continue a Fini conversation, test bot behavior or golden-set questions, pass channel or user metadata/user attributes with a turn, preserve multi-turn runtime context, interpret the created public events returned by Generate Answer, or decide when to fetch the full conversation after sending a message.
---

# Fini API Generate Answer

Use this skill when a caller wants Fini to process a message event. This is a write workflow, even for tests.

Before endpoint-specific work, fetch `https://docs.usefini.com/llms.txt` and the Generate Answer docs page.

## Workflow

1. Identify whether this starts a new conversation or continues an existing one.
2. If starting new and the user gave an agent name, resolve `botId` through List agents.
3. If continuing, require the intended `interactionId`.
4. Use `role=user` only when the caller expects a new AI reply.
5. Include channel and user metadata intentionally; do not invent CRM fields.
6. Send the event only after the target agent/workspace is unambiguous.
7. Interpret the returned created events.
8. Fetch the full conversation afterward if the user needs wrapper state, historical context, or persisted evidence.

## Defaults

- Ask before sending test traffic to a production agent if the target is ambiguous.
- Treat non-user roles as event storage, not answer generation.
- Keep metadata minimal and explicit.
- Use stable `metadata.user_attributes` keys for golden-set and personalization tests.
- Mark synthetic traffic in metadata when the caller is testing rather than serving a real user.
- Summarize returned events by role, type, message text, and evidence fields rather than dumping raw payloads.

## Gotchas

- The response is an array of created public events, not the full conversation object.
- Non-user roles do not trigger a new AI reply.
- Current public docs describe text-message behavior; verify attachment support before promising it.
- This requires write scope.
- Retrying a send can create duplicate events unless the caller has an idempotency strategy from the current docs or surrounding system.
- Generate Answer creates conversation events. It is not, by itself, a customer helpdesk ticket-sync or customer-owned action API.

## Proof Of Completion

- New conversation: report the created interaction/conversation identifier if returned, the agent used, and the reply event evidence.
- Continued conversation: report the existing `interactionId`, new event count, and whether an AI reply was created.
- Test run: state whether the output is suitable for automated testing or only manual inspection.
- Full-state requirement: fetch the conversation and verify the event appears in the thread.
- Golden-set run: preserve question, expected answer/reference, actual answer, used articles, pass/fail, and next action.

## References

Read [send-turn-playbook.md](references/send-turn-playbook.md) for start-vs-continue decisions, production safety checks, metadata handling, and response interpretation.

Read [runtime-conversation-patterns.md](references/runtime-conversation-patterns.md) when the user is continuing a conversation, replaying a test, passing user attributes, or trying to avoid role/context mistakes.
