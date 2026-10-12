---
name: fini-api-refine-with-ai
description: Use when the user explicitly asks for Fini Refine with AI or fix review; wants Fini to diagnose, fix, or improve a bad Fini-authored response; asks how that improvement workflow works before supplying a response ID; wants Fini to propose prompt, intent-Rulebook, or Knowledge article drafts and replay the answer; or wants those exact generated drafts reviewed and routed toward publication. Execution requires one exact eligible response. Route read-only conversation explanation to fini-api-conversations, standalone article work to fini-api-knowledge, caller-selected historical candidates to Replay in fini-api-testing, and fresh exact-ID comparisons to fini-api-draft-validation.
---

# Fini Refine with AI

Use this skill to plan or execute one conversation-grounded improvement loop: diagnose a bad Fini response, let Fini create reviewable draft recommendations, inspect its improved replay, and hand each draft to its owning workflow. For a planning or hypothetical request, explain the workflow and required evidence without creating anything. Execution requires one exact eligible Fini response. Refine creates persisted drafts and replay evidence; it does not publish configuration.

Before endpoint-specific work, fetch `https://docs.usefini.com/llms.txt` and the current Refine with AI pages. Treat those docs as the direct HTTP contract and the returned structured response as observed session/result evidence.

## Choose The Path

| User intent | Path |
| --- | --- |
| Plan or execute AI-assisted diagnosis and improvement of a bad Fini answer | This skill; resolve one exact eligible response before creating an iteration |
| Create or improve an article because one exact answer lacked or misstated knowledge | This skill first, then `fini-api-knowledge` for exact draft review/publish |
| Explain an answer read-only, without generating drafts or a replay | `fini-api-conversations` |
| Create or edit a standalone article from supplied text, files, or sources | `fini-api-knowledge` |
| Replay a historical answer with a caller-selected exact candidate | `fini-api-testing` |
| Compare persisted unpublished IDs on fresh controlled turns | `fini-api-draft-validation` |
| Run broad post-publish regression | `fini-api-testing` Test Suite cases |

Do not trigger on the bare word "refine." For example, "refine this Rulebook" belongs to `fini-api-agent-configuration` unless the request is explicitly about Refine with AI for a bad response.

## Workflow

1. Resolve the exact conversation and Fini response event through `fini-api-conversations`. Preserve the conversation's `botId`, verify `role=finibot`, and confirm Refine has a linked user event.
2. Read the active Refine session first with `fini_get_active_fix_review_session` or the documented direct API route. Inspect `latestIteration.status`. If it is `queued`, `generating_changes`, or `replaying`, poll it instead of creating a duplicate. If it is already `ready`, inspect it before deciding whether more feedback is needed.
3. Treat iteration creation as a write. Require clear user intent to generate drafts and a replay, but do not ask again when that intent is already explicit. Preflight whether the replay can reach Rulebook Actions or customer APIs. For the MCP connector, pass the tool-required literal `confirmationPhrase: "CONFIRM"`; for the direct API, follow the documented write-scoped request.
4. Create one iteration with `fini_create_fix_review_iteration` or the documented direct API route and a concise, evidence-based `feedbackNote` of at most 2,000 characters. Do not mix it up with the event's teammate feedback note, thumbs rating, or resolution state.
5. Preserve the returned `sessionId`, `iterationId`, and `backgroundJobId`. After an ambiguous create timeout, read the active session before retrying because creation is not documented as idempotent.
6. Poll the exact session with `fini_get_fix_review_session` or the documented direct API route and evaluate `latestIteration.status`. Stop on `ready`, `no_change`, `failed`, `published`, or `superseded`; the latter two are historical lifecycle states, not reasons to create a duplicate. Do not invent a manual workaround for `no_change` or `failed` unless the user separately asks for one.
7. On `ready`, compare the old and new answer snapshots, retain the replay interaction/event IDs, and inspect the root cause, confidence, summary, and every returned change.
8. Route each generated artifact by its exact returned ID:
   - `create_article` or `update_article` -> `draftArticleId` -> `fini-api-knowledge`;
   - `update_prompt` -> preserved `botId` plus `draftPromptVersionId` -> `fini-api-agent-configuration`;
   - `update_rule` -> `draftRuleVersionId` plus its parent `ruleId` -> `fini-api-agent-configuration`.
9. Fetch and review each exact generated artifact. Do not recreate a second manual draft when Refine already returned one. Missing draft IDs are missing artifacts, not successful draft creation.
10. Treat the Refine replay as the primary behavioral proof for the original response. Verify generated facts against trusted conversation, source, and current-artifact evidence; an improved answer is not by itself proof of factual accuracy.
11. Publish only the separately reviewed exact artifact through its owning skill after explicit live-change authorization. Multiple recommendations may be approved or rejected independently.
12. Use Draft Validation only for additional fresh cases, isolated artifact checks, or a combined candidate. After publication, use active Generate Answer proof and a stable Test Suite when broader regression evidence is required. Mark conversation feedback resolved only after the live fix is proven.

## Safety And Interpretation

- A read-only analysis request does not authorize iteration creation.
- Refine writes drafts and a separate replay conversation, but it does not change live prompt, rule, or Knowledge state.
- Automatic replay may rerun Rulebook paths and reach Actions or external APIs, depending on the generated artifacts and runtime path. Preflight conservatively and inspect returned replay evidence instead of assuming which Attribute or Rule results were reused or recomputed.
- Only one iteration should run at a time. Reuse the active session and use focused follow-up feedback for another iteration.
- Preserve the original conversation, event, session, iteration, replay, and artifact IDs as one lineage record.
- Keep prompt and rule version IDs distinct from their parent resources. Intent Rulebook publication still needs the parent `ruleId`, exact reviewed draft version, and explicit target agents.
- Do not infer which artifact Fini created from the feedback request. Trust the returned `changeType` and exact draft pointer.

## Proof Of Completion

Report the exact conversation/event and `botId`, session/iteration/background-job IDs, terminal status, old versus replayed answer, replay IDs, root-cause summary, every proposed change and exact draft ID, factual-review result, external effects observed or uninspected, publication state, and remaining validation or regression work.
