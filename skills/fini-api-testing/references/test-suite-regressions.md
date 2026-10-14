# Test Suite Cases, Criteria Groups, And Regression Runs

Read the current [Test Suite overview](https://docs.usefini.com/en/api-reference/test-suite), [fields context](https://docs.usefini.com/en/api-reference/get-test-suite-fields-context), [start run](https://docs.usefini.com/en/api-reference/start-test-suite-run), and [get run](https://docs.usefini.com/en/api-reference/get-test-suite-run) contracts before execution. The workflow below was checked against Collector `19832b3a` and MCP `ac6ee33` on 2026-10-14; fetch current contracts rather than assuming those versions are still serving.

Use `/v2/test-suite` public routes with the intended workspace's Bearer key, loaded without printing it. Read scope covers list/get and preview; write scope covers saved-resource changes and execution. Use current MCP tools when their schemas support cases/groups. Retired `fini_*test_set*` tools and `/v2/test-sets` routes cannot substitute for the new contract. Missing tools do not prevent authorized REST execution; do not switch tenant keys after authentication failure.

## Save Cases And Shared Criteria

1. Capture a conversation belonging to the selected bot with a source other than `replay`/`testsuite`, or save authored scenario turns directly. Saving a scenario does not send a live message. Read capture inputs or Action fixtures before constructing mocks; never invent rule/node/tool identities.
2. Preserve saved inputs, Attribute overrides, Action mocks, local/shared criteria, and coverage artifacts. Coverage `artifacts` identify rules/articles; they are not execution overrides. Case PATCH requires `name`; preserve it for setup-only edits. Duplicate saved setup using the source case and its updated timestamp instead of recapturing a changed conversation.
3. Read fields context. Case/group criteria can use `defaultCriterionId` with an optional `blocking` override and no custom definition fields, or a new custom definition. Creating a standalone library criterion requires a new custom definition, never a default reference. Deterministic criteria require one condition predicate and no judge prompts; judges require nonblank `judgePrompt`, `passPrompt`, and `failPrompt`, with no condition. Keep only release-critical criteria blocking.
4. Before replacing group criteria or unlinking cases, read its exact detail and inspect linked-case impact. Preserve the current `impactToken`; on a stale-token conflict, re-read and reassess instead of retrying blindly. Group PATCH replaces name and criteria rather than applying a partial criterion edit.
5. Organize cases/groups in folders when useful. Verify memberships after changes; a folder can include cases through linked groups. Collection reads require `botId`. Cases paginate in `{items,total,limit,offset}`; group/folder lists return arrays and runs return a paginated array. Use only each endpoint's consumed filters, not every field accepted by the shared DTO.

Avoid criteria that encode the candidate wording, combine unrelated claims, or cannot be traced to available evidence. Treat generated tests as suggestions, not accepted criteria.

## Run And Poll

Resolve exact selection through `caseIds`, `folderIds`, `criteriaGroupId`, or `collectionId`; omission can expand to every case for the bot. At most 200 cases can execute. A selected group supplies that group's criteria; an ordinary case run combines local and linked-group criteria.

Runs accept top-level `promptVersionId`, `ruleVersionIds`, and `knowledgeArticleDraftIds`. Verify eligible choices through run configuration: selected rules must be drafts, at most one per rule; articles must belong to the bot's knowledge tree, at most one per original article. There is no run-body `knowledgeSnapshotId` or inline draft content. Run configuration reads can initialize a knowledge snapshot internally. Starting a run captures configuration without publishing drafts.

1. Preview the same selection and resolve `incompleteCases`; every case needs effective criteria. Preview ignores draft selectors and does not validate draft eligibility or runtime Action mocks.
2. Start one authorized run (`202`) and retain its `id`, captured environment and case `items`. MCP confirmation gates belong to the connector; do not invent a REST `confirmationPhrase`.
3. Poll Get run with bounded intervals until every item is `completed` or `failed`; pending items are `queued`/`running`. Resume the same ID after interruption. Do not assume the legacy top-level run status or create another run to check progress.
4. Inspect each item's `result.verdict`, `criteriaResults`, generated messages, Action sources, replay IDs, errors and optional `failureAnalysis`. Separate completed criterion failures from execution errors and missing mocks.

Test Suite simulates Actions with recorded/custom responses and reuses captured Attributes or overrides. A missing Action mock fails without calling the live Action. For an authorized missing-mock recovery, use the **case-run item ID** with `actionIndex`, `success` and `output`, re-fetch saved case setup, then start a separately requested rerun. Scenario mocks belong in scenario setup. Runs create replay/results and consume model execution; they do not prove real provider HTTP behavior.

## Compare Runs

Keep cases, saved inputs, mocks, Attributes and effective criteria stable. Record configuration and timestamps; inspect `sameSetup`, `previousResult` and `previousCreatedAt` where returned. A changed setup prevents attribution to configuration alone. Draft passes do not prove publication; a go-live gate must evaluate the intended published configuration. Real Generate Answer, Replay, rule evaluation and integration tests retain their own external-effect rules.

Report:

```text
Baseline run ID / Candidate run ID
Test Suite case/group IDs and conversation count
Criteria IDs and blocking state
Pass, fail, and error deltas
New failures / fixed failures / unchanged failures
Criterion reasoning and evidence for every delta
Captured configuration per run
Action mock sources and saved Attribute context per run
```

If inputs, criteria, or configuration changed in uncontrolled ways, label the result inconclusive instead of attributing the delta.

## Update And Cleanup

Preserve reconstruction evidence before explicitly requested deletion. Delete only the exact supported case, folder or membership, then verify with list/get. Do not infer standalone criterion-update/delete, group-delete or run-delete routes. Never delete source conversations as implicit suite cleanup.
