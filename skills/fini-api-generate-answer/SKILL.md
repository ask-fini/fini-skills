---
name: fini-api-generate-answer
description: Use when the user wants to send one already-defined message turn into a Fini agent through the public API; start or continue a conversation; execute one already-selected exact prompt, rule, knowledge-snapshot, or article-draft artifact set; validate configured Attributes or Action-backed Rulebooks; pass channel or caller-supplied user attributes; interpret created events; or fetch the resulting conversation. Route systematic active-baseline versus unpublished-candidate comparisons to `fini-api-draft-validation`.
---

# Fini API Generate Answer

Use this skill when a caller wants Fini to process one message event through the normal agent runtime. This is a write and may be effectful even when the caller calls it a test.

Before endpoint-specific work, fetch `https://docs.usefini.com/llms.txt` and the Generate Answer docs page.

## Choose The Test Surface

| Need | Use |
| --- | --- |
| Send one new or continued runtime turn now | Generate Answer |
| Compare active behavior with unpublished artifacts one dimension at a time | `fini-api-draft-validation`, which executes through Generate Answer |
| Execute one already-selected exact artifact set | Generate Answer with explicit persisted IDs in `artifacts` |
| Rerun a turn from an existing conversation for diagnosis | Replay through `fini-api-testing` |
| Run persistent cases against current published live-agent behavior | Test Set through `fini-api-testing` |

Generate Answer creates real conversation events and runs the agent's normal reply flow for `role=user`. It is not an asynchronous regression suite and does not reproduce a historical turn automatically.

## Workflow

1. Identify whether this starts a new conversation or continues an existing one.
2. If starting new and the user gave an agent name, resolve `botId` through List agents.
3. If continuing, require the intended `interactionId`.
4. Decide whether to use active configuration or one already-selected exact `artifacts` set. Route baseline/isolation/combined-candidate methodology to `fini-api-draft-validation`. Resolve every artifact ID; do not mix workspaces or silently combine draft and active versions. A rule-version override is eligible only when its owning rule is already published and assigned to the target bot.
5. Inspect configured Attributes and reachable Action Tool nodes when their external effects matter. Caller-supplied `metadata.user_attributes` are not the same as configured Attributes.
6. Use `role=user` only when the caller expects the full reply runtime.
7. Include channel and user metadata intentionally; do not invent CRM fields.
8. Set `markAsTest: true` for synthetic traffic, then send after the target, runtime configuration, and acceptable external effects are unambiguous.
9. Interpret all returned created events.
10. Classify failures as configuration, knowledge, runtime metadata, Attribute/Data Step, or Action/Rulebook behavior before recommending follow-up work.
11. Fetch the full conversation afterward when the user needs wrapper state, historical context, or persisted evidence.

## Defaults

- Ask before sending test traffic to a production agent if the target is ambiguous.
- Treat non-user roles as event storage, not answer generation.
- Keep metadata minimal and explicit.
- Use stable `metadata.user_attributes` keys for golden-set and personalization tests.
- Set top-level `markAsTest: true` for synthetic traffic. Do not substitute an ad hoc metadata marker for the documented field.
- Use exact artifact overrides to test prompt and knowledge review drafts without changing active configuration. For rules, use a draft override only when its owning rule is already published and assigned to the target bot.
- Use this as the one-turn proof after prompt, rulebook, Action/Attribute, source, or knowledge changes; do not make those changes from this skill.
- Use a controlled workspace, test account, and inputs whenever reachable Actions can mutate an external system.
- Summarize returned events by role, type, message text, and evidence fields rather than dumping raw payloads.

## Artifact Overrides

Send only the fields needed for the exact proof:

- `promptVersionId`: run with one exact prompt lifecycle version, including a review draft.
- `knowledgeSnapshotId`: use one exact knowledge snapshot for retrieval.
- `knowledgeArticleDraftIds`: include exact draft article IDs in retrieval.
- `ruleVersionIds`: evaluate exact Rulebook versions for this turn, but only for rules already available to the target bot through a published version and bot assignment.

Persist and fetch candidate content through its owning prompt, knowledge, or intent-rule workflow first. Send only exact stored IDs in `artifacts`; never inline prompt arrays, article text, candidate instructions, or rule flow JSON. Artifact overrides affect answer generation for the submission; they do not publish or persistently replace active configuration. Record every override ID in the result so the proof is reproducible.

## Gotchas

- The response is an array of created public events, not the full conversation object.
- Non-user roles do not trigger a new AI reply.
- Public file attachments are not accepted by Generate Answer. Response events can still contain attachments already produced by the runtime.
- This requires write scope.
- `markAsTest: true` labels synthetic traffic but does not prevent event creation or suppress configured Attributes and Actions.
- Retrying a send can create duplicate events unless the caller has an idempotency strategy from the current docs or surrounding system.
- `metadata.user_attributes` are values supplied by the caller. Configured Attributes are outbound data lookups that run automatically only for eligible messages on their configured agent/channel assignments.
- A `role=user` turn may run configured Attributes and any Action Tool node reached by a published or overridden Rulebook. Those calls may read or mutate customer systems.
- A draft prompt or rule override does not make the runtime side-effect-free.
- A brand-new unassigned Rulebook draft cannot be executed merely by passing its version ID. Preview its fields and flow structurally; publish and assign its owning rule before using a runtime override.
- Generate Answer exposes no documented dry-run or Action-suppression flag.
- Do not assume an artifact array replaces or merges with every active artifact not named in the request. Verify the current runtime semantics when isolation matters and report what is known.
- Action assignment junctions do not control Action reachability; a Rulebook Tool node and the rule's agent assignment do.
- Generate Answer is not a Replay and does not provide aggregate criteria results like a Test Set.

## Proof Of Completion

- New conversation: report the created `interactionId`, agent used, configuration mode, and reply event evidence.
- Continued conversation: report the existing `interactionId`, new event count, and whether an AI reply was created.
- Artifact proof: report every exact persisted artifact ID, `markAsTest` state, rule-override eligibility, actual output, and whether active configuration was unchanged.
- Runtime integrations: report configured Attribute/Action effects that were observed or state that they were not inspected.
- Test run: state whether the output proves one runtime case, whether a historical case needs separate Replay diagnosis, and whether current published behavior needs a persistent live Test Set run.
- Full-state requirement: fetch the conversation and verify the event appears in the thread.
- Golden-set run: preserve question, expected answer/reference, actual answer, used articles, pass/fail, and next action.

## References

Read [send-turn-playbook.md](references/send-turn-playbook.md) for active-vs-artifact decisions, runtime integration effects, Replay/Test Set routing, and response interpretation.

Read [runtime-conversation-patterns.md](references/runtime-conversation-patterns.md) when continuing a conversation, passing user attributes, testing exact artifacts, or avoiding role/context mistakes.

Use `fini-api-draft-validation` when the request needs an active baseline, isolated prompt/knowledge/snapshot/rule candidates, a like-for-like comparison, or a combined unpublished candidate before publication.
