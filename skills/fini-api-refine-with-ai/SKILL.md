---
name: fini-api-refine-with-ai
description: Use when the user explicitly asks for Fini Refine with AI or fix review; wants AI-assisted diagnosis of one exact bad Fini-authored response; wants Fini to propose prompt, intent-Rulebook, or Knowledge article drafts and replay the answer; or wants those exact generated drafts reviewed and routed toward publication. Route read-only conversation explanation to fini-api-conversations, standalone article work to fini-api-knowledge, caller-selected historical candidates to Replay in fini-api-testing, and fresh exact-ID comparisons to fini-api-draft-validation.
---

# Fini Refine with AI

Use this skill for one conversation-grounded improvement loop: diagnose an exact bad Fini response, let Fini create reviewable draft recommendations, inspect its improved replay, and hand each draft to its owning workflow. Refine creates persisted drafts and replay evidence; it does not publish configuration.

Before endpoint-specific work, fetch `https://docs.usefini.com/llms.txt` and the current Refine with AI pages. Treat those docs and the structured response as endpoint truth.

## Choose The Path

| User intent | Path |
| --- | --- |
| Diagnose or improve one exact bad Fini answer and propose the fix | This skill |
| Create or improve an article because one exact answer lacked or misstated knowledge | This skill first, then `fini-api-knowledge` for exact draft review/publish |
| Explain an answer read-only, without generating drafts or a replay | `fini-api-conversations` |
| Create or edit a standalone article from supplied text, files, or sources | `fini-api-knowledge` |
| Replay a historical answer with a caller-selected exact candidate | `fini-api-testing` |
| Compare persisted unpublished IDs on fresh controlled turns | `fini-api-draft-validation` |
| Run broad post-publish regression | `fini-api-testing` Test Sets |

Do not trigger on the bare word "refine." For example, "refine this Rulebook" belongs to `fini-api-agent-configuration` unless the request is explicitly about Refine with AI for a bad response.

## Workflow

1. Resolve the exact conversation and Fini response event through `fini-api-conversations`. Preserve the conversation's `botId`, verify `role=finibot`, and confirm Refine has a linked user event.
2. Read the active Refine session first with `fini_get_active_fix_review_session` or the documented direct API route. If its latest iteration is `queued`, `generating_changes`, or `replaying`, poll it instead of creating a duplicate. If it is already ready, inspect it before deciding whether more feedback is needed.
3. Treat iteration creation as a write. Require clear user intent to generate drafts and a replay, but do not ask again when that intent is already explicit. Preflight whether the replay can reach Rulebook Actions or customer APIs. For the MCP connector, pass the tool-required literal `confirmationPhrase: "CONFIRM"`; for the direct API, follow the documented write-scoped request.
4. Create one iteration with `fini_create_fix_review_iteration` or the documented direct API route and a concise, evidence-based `feedbackNote`. Do not mix it up with the event's teammate feedback note, thumbs rating, or resolution state.
5. Preserve the returned `sessionId`, `iterationId`, and `backgroundJobId`. After an ambiguous create timeout, read the active session before retrying because creation is not documented as idempotent.
6. Poll the exact session with `fini_get_fix_review_session` or the documented direct API route. Stop on `ready`, `no_change`, `failed`, `published`, or `superseded`; the latter two are historical lifecycle states, not reasons to create a duplicate. Do not invent a manual workaround for `no_change` or `failed` unless the user separately asks for one.
7. On `ready`, compare the old and new answer snapshots, retain the replay interaction/event IDs, and inspect the root cause, confidence, summary, and every returned change.
8. Route each generated artifact by its exact returned ID:
   - `create_article` or `update_article` -> `draftArticleId` -> `fini-api-knowledge`;
   - `update_prompt` -> preserved `botId` plus `draftPromptVersionId` -> `fini-api-agent-configuration`;
   - `update_rule` -> `draftRuleVersionId` plus its parent `ruleId` -> `fini-api-agent-configuration`.
9. Fetch and review each exact generated artifact. Do not recreate a second manual draft when Refine already returned one. Missing draft IDs are missing artifacts, not successful draft creation.
10. Treat the Refine replay as the primary behavioral proof for the original response. Verify generated facts against trusted conversation, source, and current-artifact evidence; an improved answer is not by itself proof of factual accuracy.
11. Publish only the separately reviewed exact artifact through its owning skill after explicit live-change authorization. Multiple recommendations may be approved or rejected independently.
12. Use Draft Validation only for additional fresh cases, isolated artifact checks, or a combined candidate. After publication, use active Generate Answer proof and a stable Test Set when broader regression evidence is required. Mark conversation feedback resolved only after the live fix is proven.

## Safety And Interpretation

- A read-only analysis request does not authorize iteration creation.
- Refine writes drafts and a separate replay conversation, but it does not change live prompt, rule, or Knowledge state.
- Automatic replay can execute reachable Attributes, Rulebooks, Actions, or external APIs. Use controlled identities and acceptable targets.
- Only one iteration should run at a time. Reuse the active session and use focused follow-up feedback for another iteration.
- Preserve the original conversation, event, session, iteration, replay, and artifact IDs as one lineage record.
- Keep prompt and rule version IDs distinct from their parent resources. Intent Rulebook publication still needs the parent `ruleId`, exact reviewed draft version, and explicit target agents.
- Do not infer which artifact Fini created from the feedback request. Trust the returned `changeType` and exact draft pointer.

## Proof Of Completion

Report the exact conversation/event and `botId`, session/iteration/background-job IDs, terminal status, old versus replayed answer, replay IDs, root-cause summary, every proposed change and exact draft ID, factual-review result, external effects observed or uninspected, publication state, and remaining validation or regression work.
