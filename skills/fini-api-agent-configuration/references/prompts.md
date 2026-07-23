# Prompt Workflows

Use this reference when the user wants to inspect, compare, or update an existing Fini agent's prompts.

## Prompt Lifecycle

Prompts are versioned configuration for one existing agent:

1. Read the merged prompt view.
2. Check history to understand saved agent-specific versions.
3. Fetch an exact historical version when the user wants a precise comparison or rollback source.
4. Edit the full prompt arrays offline.
5. Save a new version after approval.
6. Re-fetch and test behavior with Generate Answer.

Use prompts for instruction, tone, planning, channel behavior, and general handling policy. Do not use prompts as a substitute for missing factual knowledge, source ingestion, or customer-owned action APIs.

## Read Current Prompts

1. Resolve `botId` with List agents if needed.
2. Fetch current prompts for that agent.
3. If the agent has no saved version, treat the returned content as a workspace-template fallback, not proof of an agent-specific saved prompt.
4. Fetch prompt history when the user asks what changed, wants rollback context, or asks whether the agent has custom prompts.
5. Fetch the exact version by prompt ID before using historical content as a restore or comparison source.

## Update Prompts

Use this sequence for a write:

1. Fetch current prompts.
2. Make a minimal diff against the prompt arrays.
3. Preserve all three full top-level arrays:
   - `hcPlanningPrompt`
   - `hcGuidelinePrompt`
   - `hcChannelPrompt`
4. Show a short plan with target agent, changed sections, and rollback/history note.
5. Ask for explicit approval unless the caller already authorized the exact write.
6. Save the new prompt version.
7. Fetch prompts again to verify the merged read shape.
8. Test with Generate Answer if behavior should change. Reuse the same questions and relevant metadata from the baseline test so the comparison is meaningful.

## Diagnose, Edit, Retest

Use this loop when the user supplies questions/answers, failing conversations, or a behavioral test set:

1. Fetch the complete current prompt and history.
2. Run a small baseline set with Generate Answer and preserve interaction/event IDs.
3. Classify each failure as prompt behavior, rule behavior, missing knowledge, or runtime metadata before editing.
4. For prompt failures, make the smallest diff to the full arrays and show it for approval.
5. Save the complete arrays, re-fetch the stored version, and rerun the same cases.
6. Report before/after behavior and any regressions; do not claim success from a saved version alone.

## Gotchas

- Prompt update saves a new version; it does not patch the current row in place.
- Send full section arrays, not only the changed section.
- Never reconstruct a write payload from a compact preview. Fetch prompt content in full before editing.
- Subsection helper fields from merged reads are not required in the write payload.
- The write response is the stored version row. Fetch prompts after writing when the merged view matters.
- Current docs say the write route uses `POST /v2/bots/{id}/hc-prompt` without `/public`. Verify before calling.
- Read scope is enough for get/history; write scope is required for update.

## Output Shape

For a prompt change plan, report:

```markdown
Target agent: [name or botId]
Current source: [saved version or workspace template fallback]
Sections changing: [planning/guideline/channel]
Write status: [not run / saved version ID / failed]
Verification: [re-fetched / not checked]
Runtime test: [not requested / passed / failed / needs review]
Rollback context: [history checked / history not checked]
```
