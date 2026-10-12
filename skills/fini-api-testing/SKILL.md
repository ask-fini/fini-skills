---
name: fini-api-testing
description: Manage Fini Test Suite cases, scenarios, folders, reusable criteria groups, Action mocks, and asynchronous regression runs through public APIs; compare published or exact draft configurations; inspect Replay, Rulebook tests, and quality evidence. Use for test-case setup, suite results, missing mocks, or baseline/candidate regression. Route ordinary message generation to fini-api-generate-answer and one-turn artifact comparison to fini-api-draft-validation.
---

# Fini API Testing

Choose the surface that answers the request and preserve exact inputs, configuration IDs, and result evidence.

## Choose The Surface

| Need | Surface |
| --- | --- |
| Persistent cases, criteria groups, folders, or multi-case draft/published regression | Test Suite; read [test-suite-regressions.md](references/test-suite-regressions.md) |
| One new ordinary turn or proof of real Attribute/Action integration behavior | `fini-api-generate-answer`; external effects remain possible |
| One new controlled turn compared with exact unpublished artifacts | `fini-api-draft-validation`; external effects remain possible |
| Regenerate one historical response under caller-selected configuration | Replay; read [replays.md](references/replays.md) |
| Diagnose one bad response and generate draft fixes with replay evidence | `fini-api-refine-with-ai`, when installed |
| Discover rule fields, generate suggestions, or evaluate one rule | Read [rule-testing.md](references/rule-testing.md) |
| Aggregate usage or quality over time | `fini-api-agents`; analytics is observational evidence |

## Test Suite Contract

Test Suite uses saved cases, folders/collections, reusable criteria groups, and case-level run results. A case captures an eligible conversation or defines a synthetic multi-turn scenario. Runs regenerate answers from saved inputs, capture their execution configuration, simulate Actions using recorded/custom responses, and reuse captured Attributes. A missing Action mock fails with diagnostics; it does not fall back to a live Action call.

Runs can select exact prompt versions, rule drafts, and knowledge article drafts without publishing. These run-level selectors differ from Generate Answer `artifacts`. Linked case `artifacts` identify coverage; they are not draft execution overrides. Simulated results do not prove a customer integration's real HTTP behavior.

Use the public REST contract in the regression reference with the intended tenant-scoped key through the available HTTP/shell runtime. An absent named MCP tool does not disable REST access. In an MCP environment, use only tools whose current schema explicitly supports the new cases/groups contract; do not substitute legacy `fini_*test_set*` operations. The old `/v2/test-sets/...` runtime has been replaced.

## Workflow

1. Resolve one workspace and `botId`; inspect cases, folders, groups, and existing runs before changing anything.
2. For a conversation case, verify the source belongs to this bot and is neither `replay` nor `testsuite`. Capture its inputs once. For a new scenario, save authored turns directly; do not generate a live conversation merely to seed a case.
3. Read fields context before constructing deterministic conditions. Prefer a bounded semantic judge for meaning and deterministic criteria for observable state. Keep only release-critical assertions blocking.
4. Reuse criteria groups for shared assertions; inspect linked-case impact before replacing a group's criteria or removing membership. Read and preserve the current `impactToken` for those writes.
5. Preserve saved inputs, Attribute overrides, Action mocks, local/group criteria, and configuration selections for baseline/candidate comparison. Change one intended dimension at a time.
6. Preview the selected run. Address `incompleteCases`; every selected case needs effective criteria. Confirm exact scope and configuration before starting a requested run.
7. Start once. Retain its ID and inspect the same run's items until none remain `queued` or `running`. Use bounded waits when the user requests completion; otherwise return the run ID as the resume point. Do not create another run to check progress.
8. Report completed pass/fail verdicts separately from execution failures. Inspect missing mocks, judge errors, criterion reasoning, generated replies, configuration metadata, and `sameSetup` before attributing a change.

## Intent And Evidence

- List/get, fields context, and run preview do not start execution. Case/group/folder changes, mock recovery, and starting a run require the user's scoped intent; existing clear intent does not need a second approval or an invented REST `confirmationPhrase`.
- Runs create persisted replay/results and consume model execution even though customer Actions are simulated. Keep real Generate Answer, standalone Replay, rule evaluation, and Action/Attribute tests under their own effect rules.
- Delete only explicitly requested cases, folders, or membership. Preserve reconstruction evidence and verify the result. Do not delete source conversations or infer group/run deletion endpoints.
- Show human-readable names and verified dashboard links. Keep opaque IDs in links or structured evidence unless requested for debugging. Use only dashboard routes verified for the current frontend.
- Never expose keys, credential headers, customer secrets, or unnecessary conversation content.
- Report workspace/agent, case/group/run IDs, saved setup, selected configuration, pending/pass/fail/error totals, blocking results, mock gaps, comparison controls, and remaining uncertainty. A simulated pass does not mean drafts were published.
