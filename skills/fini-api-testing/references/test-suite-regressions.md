# Test Suite Cases, Criteria Groups, And Regression Runs

## Public REST Contract

Contract checked against the serving Collector commit `6ebf4ba6` on 2026-10-12: [controller](https://bitbucket.org/finitechnologies/collector-api-v2/src/6ebf4ba6df53b1df747627c52bd7dccdbba046da/src/test-suite/test-suite.controller.ts), [request DTOs](https://bitbucket.org/finitechnologies/collector-api-v2/src/6ebf4ba6df53b1df747627c52bd7dccdbba046da/src/test-suite/dto/test-suite.dto.ts), and [runner](https://bitbucket.org/finitechnologies/collector-api-v2/src/6ebf4ba6df53b1df747627c52bd7dccdbba046da/src/test-suite/test-case-runner.service.ts).

Base: `https://api-prod.usefini.com/v2/test-suite`. Authenticate with the intended workspace's public API key as a Bearer header, loaded from its environment without printing it. Read scope covers reads and preview; write scope covers saved-resource changes and execution. Do not substitute another tenant's key after an authentication failure.

Use current public docs when accessible. If the docs index is blocked or still describes Test Sets, this versioned contract provides the new routes. A read-only endpoint response can verify availability, but does not verify a write schema. Stop on an unsupported field or changed contract; do not guess routes or switch to removed `/test-sets` endpoints.

| Operation | Method and path relative to the base |
| --- | --- |
| List cases / save case | `GET` / `POST /cases/public` |
| Read / update / delete case | `GET` / `PATCH` / `DELETE /cases/{id}/public` |
| Inspect capture input | `GET /conversation-input/public?botId={botId}&conversationId={conversationId}` |
| Field definitions and criterion defaults | `GET /fields-context/public` |
| Create library criterion | `POST /criteria/public` |
| List / create criteria groups | `GET` / `POST /criteria-groups/public` |
| Read / replace group | `GET` / `PATCH /criteria-groups/{id}/public` |
| Link / unlink group cases | `POST` / `DELETE /criteria-groups/{id}/cases/public` |
| Capture conversations or attach existing cases to group | `POST /criteria-groups/{id}/conversations/public` |
| List / create folders | `GET` / `POST /folders/public` |
| Link / unlink folder cases | `POST` / `DELETE /folders/{id}/cases/public` |
| Link / unlink folder groups | `POST` / `DELETE /folders/{id}/groups/public` |
| Delete folder | `DELETE /folders/{id}/public` |
| Available run configuration / Action identities | `GET /run-configuration/public` / `GET /action-fixtures/public` |
| Run readiness without execution | `POST /runs/preview/public` |
| List / start runs | `GET` / `POST /runs/public` |
| Read run and its case items | `GET /runs/{id}/public` |
| Recover missing Action mock | `POST /case-runs/{id}/action-mocks/public` |
| Aggregate resource counts | `GET /summary/public` |

Collection reads require `botId`, except fields context and exact-ID reads. Supported list filters include `search`, `folderId`, `criteriaGroupId`, `artifactId`, `ruleId`, and `articleId`; use only filters relevant to that endpoint. Cases paginate with `limit` (1–100, default 50) and `offset`, returning `{items,total,limit,offset}`. Group/folder/run lists return arrays; do not assume a cursor or that every list consumes every query field.

## Save Cases And Shared Criteria

- A case requires `botId`, nonblank `name`, and `sourceType`: `conversation` with `conversationId`, or `scenario` with `scenario.turns`. Supply exactly the corresponding source. Select source conversations through `fini-api-conversations`; a capture must belong to this bot and have a source other than `replay`/`testsuite`.
- A scenario has 1–50 turns of `{message, actions?}` and optional `userAttributes`. Each mock Action specifies verified `ruleId`, `nodeId`, `toolId`, `output`, and optional `success`. Read Action fixtures for identities rather than inventing nodes. Saving a scenario does not send a live message.
- Cases can carry local `criteria`, `criteriaGroupIds`, `collectionId`, `attributeOverrides`, and coverage `artifacts: [{kind: "rule" | "article", id}]`. Conversation `actionResponses` edits require exact returned `fingerprint`, `eventId`, and `resultIndex`, with `success`/`output` or `reset`; re-read after a stale fingerprint.
- Case `PATCH` requires `name`; preserve the current name for setup-only edits. Omit fields that should remain unchanged; re-fetch saved setup. Duplication uses `duplicateOfCaseId` and `duplicateSourceUpdatedAt` to copy the saved input and mocks, rather than recapturing a changed Inbox conversation.
- Group creation requires `botId`, `name`, and 1–50 criteria; optional `conversationIds`, `caseIds`, and `collectionId` attach members. At most 50 conversation/case selections can be added together. Group case membership accepts `caseIds`; the conversations route requires `conversationIds` (use `[]` for existing cases) and supports `caseIds` plus `requestId` for repeat-safe additions.
- Read the exact group to obtain `linkedCases`, `caseCount`, `revision`, and `impactToken`. Group `PATCH` requires `botId`, `name`, replacement `criteria`, and the current `impactToken`; unlinking cases also requires that token. Review the affected members. On a stale token, re-read and reassess; do not blindly retry a shared change.
- Criterion definitions use `defaultCriterionId` without custom definition fields, or `name` and `type` (`deterministic`, `basic_judge`, `complex_judge`), with optional `blocking`. Deterministic criteria require `condition` and forbid judge prompts; both judge types require nonblank `judgePrompt`, `passPrompt`, and `failPrompt` and forbid `condition`. Read returned `defaultCriteria`, `contextFields`, `operatorTypes`, and `quantifierTypes`; deterministic conditions have one `predicate` object, not a predicate array.
- Folders use `{botId,name}`. Case membership uses `{caseIds}`; group membership uses `{groupIds}`. A collection/folder can include cases through linked groups. Folder deletion is organizational cleanup; source-conversation deletion is never implicit. No public group-delete, run-delete, or standalone criterion-update/delete route is exposed here.

## Preview And Execute

The run body has `botId` and selection via `caseIds`, `folderIds`, `criteriaGroupId`, or `collectionId`; at most 200 cases can execute. A group run applies that selected group's criteria; an ordinary case run combines its local and linked-group criteria. An omitted selection can expand to the agent's cases, so resolve scope explicitly instead of accidentally running everything.

Select optional `promptVersionId`, `ruleVersionIds`, and `knowledgeArticleDraftIds` at the run body's top level. Configuration options identify eligible versions. Selected rule versions must be drafts, at most one per rule; article drafts must be available in the bot's knowledge tree, at most one per original article. There is no run-body `knowledgeSnapshotId`, inline draft content, or `confirmationPhrase`. Configuration is captured when the run starts and does not publish drafts. Configuration-options reads can initialize a knowledge snapshot internally; do not describe them as guaranteed zero-write diagnostics.

1. Preserve the exact case inputs, mocks, attributes, criteria/group revisions, and selected configuration for comparison.
2. Preview the same selection: inspect `total`, `readyCaseIds`, and `incompleteCases`. Every selected case needs effective criteria. Readiness does not guarantee that every runtime Action branch has a mock.
3. Start one authorized run (`202`) and retain its `id`. The response contains an environment and `items`; do not expect the old `status: running, result: null` shape.
4. Get the same run using bounded waits. Case items move through `queued`, `running`, `completed`, or `failed`. Completion is determined from those items, not an assumed top-level run status. Resume the same ID after a timeout.
5. Completed items contain `result.verdict`, `criteriaResults`, generated `messages`, `actions`, `replayInteractionId`, and configuration evidence. Failed items can still retain diagnostics; inspect `error` and each Action's `source` (`recorded`, `custom`, `missing`). Count pending, completed passes, completed criterion failures, and execution errors separately.

Test Suite execution uses saved inputs and simulated Action responses, and does not call live Attribute providers. A newly reached Action without a matching mock fails rather than executing it. To recover a missing Action on a failed conversation case, use the **case-run item ID** with `{actionIndex,success,output}`; this updates saved case setup and requires explicit write intent. Review the request identity, re-fetch the case, and start a separately requested rerun. Scenario mock changes belong in the scenario setup. These simulations still create replay events and use model execution; they do not verify real provider HTTP calls.

## Compare And Clean Up

Keep cases, inputs, mocks, attributes, and effective criteria stable across baseline/candidate runs. Record each run's environment and timestamps; inspect returned `sameSetup`, `previousResult`, and `previousCreatedAt` where present. A changed group, input, mock, or external assumption prevents attribution to configuration alone. A pass based on simulated context does not prove that configuration was published or customer integrations are healthy.

Before explicitly requested deletion, preserve the case or folder definition, membership, and relevant run evidence. Delete only that target and verify its absence by get/list. Do not delete source conversations or infer legacy cleanup rules or unavailable group/run deletion routes.
