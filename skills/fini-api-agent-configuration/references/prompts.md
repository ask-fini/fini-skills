# Prompt Version Workflows

Use this reference when the user wants to inspect, compare, stage, publish, or directly update an existing Fini agent's prompts.

## Contents

- [Prompt Lifecycle](#prompt-lifecycle)
- [Read Current Prompts](#read-current-prompts)
- [Draft, Review, And Publish](#draft-review-and-publish)
- [Recover From A Stale Draft](#recover-from-a-stale-draft)
- [Direct Update Alternative](#direct-update-alternative)
- [Diagnose, Edit, Retest](#diagnose-edit-retest)
- [Gotchas](#gotchas)
- [Output Shape](#output-shape)

## Prompt Lifecycle

Prompts are versioned configuration for one existing agent:

1. Read the merged prompt view.
2. Check history to understand saved agent-specific versions.
3. Fetch an exact historical version when the user wants a precise comparison or rollback source.
4. Edit the full prompt arrays offline.
5. Create a draft lifecycle version.
6. Fetch and review that exact draft version.
7. Publish that exact version when it is ready.
8. Re-fetch and test behavior with Generate Answer.

Use prompts for instruction, tone, planning, channel behavior, and general handling policy. Do not use prompts as a substitute for missing factual knowledge, source ingestion, or Action implementation.

## Read Current Prompts

1. Resolve `botId` with List agents if needed.
2. Fetch current prompts for that agent.
3. If the agent has no saved version, treat the returned content as a workspace-template fallback, not proof of an agent-specific saved prompt.
4. Fetch prompt history when the user asks what changed, wants rollback context, or asks whether the agent has custom prompts.
5. Fetch the exact saved version by its prompt-version ID before using historical content as a restore or comparison source.

## Draft, Review, And Publish

Use this default sequence for a behavior change:

1. Fetch current prompts.
2. Make a minimal diff against the prompt arrays.
3. Preserve all three full top-level arrays:
   - `hcPlanningPrompt`
   - `hcGuidelinePrompt`
   - `hcChannelPrompt`
4. Create a draft version and record its `id`, `versionNumber`, `parentVersionId`, and `status`.
5. Fetch that exact lifecycle version by `versionId`; do not treat the creation response or a summary as the review artifact. When using MCP, require `truncated: false`; otherwise retrieve the complete arrays before review.
6. Compare the exact draft with the latest current prompt and summarize the complete intended diff.
7. Publish only that reviewed `versionId`.
8. Fetch current prompts and the published lifecycle version to verify the active state.
9. Test with Generate Answer. Reuse the same questions, metadata, and artifact choices from the baseline so the comparison is attributable.

Creating a draft is a write but does not change active agent behavior. Publishing changes live behavior.

## Recover From A Stale Draft

Publishing rejects a draft when the active prompt changed after the draft was based on its `parentVersionId`.

1. Stop; do not retry the same stale draft ID.
2. Fetch the stale draft and record its exact `parentVersionId`.
3. Fetch the exact parent lifecycle version that the stale draft was based on.
4. Fetch history, identify the latest active lifecycle version, and fetch that exact complete version. Read the current merged prompt separately when workspace-template changes also need consideration.
5. Derive the intended change as stale draft versus exact parent, then compare exact parent versus exact latest active to detect overlapping edits.
6. Surface conflicts instead of overwriting newer work. Apply only the reviewed non-conflicting diff to the latest complete arrays.
7. Create a new draft.
8. Fetch the new exact `versionId`, require a complete non-truncated payload, and review all three arrays.
9. Publish only the newly reviewed version.

Report the stale draft ID, exact parent ID, exact latest active version ID, conflicts found, and replacement draft ID. Treat the replacement as a new review target; authorization naming the stale ID does not identify the replacement. Never silently discard intervening changes.

## Direct Update Alternative

Use Update prompts only when the caller explicitly wants the complete payload to become live immediately and does not need a reviewable draft gate.

1. Fetch the complete current prompts.
2. Apply the minimal diff to all three arrays.
3. Confirm the exact target and immediate-live intent if it was not already clear.
4. Call the documented `POST /v2/bots/{id}/hc-prompt/public` route.
5. Fetch current prompts and history to verify the saved version.
6. Run the same behavioral cases.

This route still saves a new version; it is "direct" because it skips the draft/publish lifecycle, not because it patches a row.

## Diagnose, Edit, Retest

Use this loop when the user supplies questions/answers, failing conversations, or behavioral test cases:

1. Fetch the complete current prompt and history.
2. Run a small active baseline through `fini-api-draft-validation` and preserve interaction/event IDs.
3. Classify each failure as prompt behavior, rule behavior, missing knowledge, or runtime metadata before editing.
4. For prompt failures, make the smallest diff to the full arrays.
5. Create and fetch an exact draft. Require complete non-truncated arrays. Test it before publication through `fini-api-draft-validation` by sending only its `promptVersionId` through Generate Answer artifacts. Through MCP, set the connector's `markAsTest: true`; direct REST exposes no top-level field by that name.
6. Publish the exact reviewed version when requested, then rerun the same cases without the draft override to prove active behavior.
7. Report before/after behavior and any regressions; do not claim success from a saved version alone.

## Gotchas

- The current prompt read is merged with workspace templates. The exact lifecycle-version read is stored content and includes lifecycle metadata.
- Draft creation needs a current prompt record and stores `parentVersionId` for stale detection.
- Draft creation does not make the draft active.
- Publish requires the exact lifecycle `versionId` and fails on a stale draft.
- Direct prompt update saves a new version; it does not patch the current row in place.
- Send full section arrays, not only the changed section.
- Never reconstruct a write payload from a compact preview. Fetch prompt content in full before editing.
- An MCP response with `truncated: true` is not complete enough for exact review, three-way comparison, or publication.
- Subsection helper fields from merged reads are not required in the write payload.
- The write response is the stored version row. Fetch prompts after writing when the merged view matters.
- Current docs use `/public` for draft, exact-draft fetch, publish, and direct update routes. Verify the live docs before calling.
- Read scope is enough for current/history/exact-version reads; write scope is required for draft creation, publish, and direct update.

## Output Shape

For a prompt change plan, report:

```markdown
Target agent: [name or botId]
Current source: [saved version or workspace template fallback]
Sections changing: [planning/guideline/channel]
Path: [draft-review-publish / direct update]
Draft: [not created / version ID, parent ID, status / stale]
Publish: [not requested / exact version ID published / failed]
Verification: [current and exact version re-fetched / not checked]
Runtime test: [not requested / draft artifact passed / active passed / failed / needs review]
Rollback context: [history checked / history not checked]
```
