# Replays

Use a replay to regenerate one known Fini response or a sequence ending at that response with explicit candidate controls. Do not describe it as a complete current-configuration rerun. Read the current [Replay overview](https://docs.usefini.com/en/api-reference/replays), [Create replay](https://docs.usefini.com/en/api-reference/create-replay), [List replays](https://docs.usefini.com/en/api-reference/list-replays), [Get replay](https://docs.usefini.com/en/api-reference/get-replay), and [Get replay events](https://docs.usefini.com/en/api-reference/get-replay-events) pages before calling the API.

## Selection

Choose Replay when the source conversation and target Fini response already exist and the caller already selected a specific supported candidate or only wants regeneration. Choose `fini-api-refine-with-ai` when the user wants Fini to diagnose that response, generate reviewable draft fixes, and provide its own improved replay. Choose Generate Answer for a new scenario, an intentional multi-turn probe, or Attribute runtime proof. Choose a Test Suite for persistent multi-case regression against captured published or selected draft configuration; its runs simulate Actions using recorded or custom mocks.

A replay creates a separate conversation and does not overwrite the original or publish configuration changes. By default, it regenerates an answer while reusing the original response's executed Rulebook and Attribute evidence; it is not a complete current-configuration rerun.

Use exact candidate artifacts deliberately:

- pass `promptVersionId` for an exact prompt draft/version;
- pass `ruleVersionIds` for exact rule candidates that must be re-evaluated, but only when each owning rule is already published and assigned to the responding agent. A brand-new unassigned rule draft is not executable through Replay;
- pass supported knowledge artifacts or `mlModels` only for the candidate being tested.

Artifact-backed rule evaluation can reach Tool nodes and invoke Actions. In `until` mode, every regenerated turn can reach those nodes, so preflight the whole included sequence rather than only the target response. Replay does not refresh Attribute execution results; use a fresh Generate Answer conversation to prove an Attribute change.

## Procedure

1. Fetch the original conversation and identify the exact Fini response event to test.
2. Record the original conversation ID, response event ID, answer, timestamps, and visible prompt, knowledge, rule, or model artifacts.
3. Choose the mode deliberately:
   - `until` walks user events through the one linked to the target response, skips copying their original Fini replies, and generates a replacement reply after every included user event. It can therefore produce multiple generations and multiple external effects when candidate rules invoke Actions.
   - `single` copies the prior conversation history and regenerates only the selected target response. Prefer it when one response is the intended proof.
4. Bind the run to exact candidate IDs. A request to test an unpublished prompt requires its `promptVersionId`. A draft version of an already published and assigned rule requires its `ruleVersionIds`; a brand-new unassigned rule draft has no supported Replay execution path. Without a supported exact artifact, do not claim the draft was tested.
5. Create one replay and retain its replay conversation ID and `parentInteractionId`.
6. Fetch replay metadata and events. Align the candidate response with the original through `originalEventId`; do not compare unrelated last messages.
7. Compare:
   - response meaning and required facts;
   - escalation or reply type;
   - knowledge and rule evidence when returned;
   - reused versus re-evaluated Rulebook evidence;
   - Action/tool execution and external effects from any re-evaluated candidate path;
   - errors, latency, and replay status.
8. List replays for the original conversation to establish the audit trail and avoid presenting an older replay as the latest candidate.

## Evidence Template

```text
Original: interactionId, responseEventId, answer, artifact versions
Candidate: replayId, originalEventId, mode, regenerated turn count, overrides, answer
Comparison: expected behavior, matched facts, regressions, evidence
Effects: newly executed Actions/external calls, reused original execution evidence, or unavailable
Conclusion: improved / unchanged / regressed / inconclusive
```

Do not infer that no Action ran merely because the answer text looks unchanged. Report only returned execution evidence, distinguish reused original evidence from newly executed evidence, and explicitly label unavailable evidence.

## Cleanup

The current replay docs expose create/list/get/events but no dedicated replay-deletion endpoint. Do not invent one or claim the replay was deleted. If cleanup is required, inspect the latest public Conversations deletion docs and use that workflow only when it explicitly supports the target replay conversation and the user names the exact deletion intent.
