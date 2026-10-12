# Rulebook And Rule Workflows

Use this reference when the user says rulebook, rule, business rule, intent rule, escalation workflow, routing behavior, or asks to generate behavior from instructions.

## Contents

- [Rulebook Mental Model](#rulebook-mental-model)
- [Reply Rules Are Not Rulebooks](#reply-rules-are-not-rulebooks)
- [Choose The Safe Path](#choose-the-safe-path)
- [Draft-First Workflow](#draft-first-workflow)
- [Duplication Workflow](#duplication-workflow)
- [Wire An Action Into A Rule](#wire-an-action-into-a-rule)
- [Rule Testing Workflow](#rule-testing-workflow)
- [Existing Rule Workflow](#existing-rule-workflow)
- [Gotchas](#gotchas)
- [Proof](#proof)

## Rulebook Mental Model

Rules/rulebooks are behavior and workflow controls for existing agents. Use them when the desired change is about what the agent should do in a class of situations, not when the issue is simply missing documentation.

Common rulebook jobs:

- route specific intents
- trigger escalation or handoff behavior
- guide structured workflow steps
- invoke Actions through Tool nodes
- use tags or input schema as conditions
- constrain behavior around sensitive or high-risk situations

Intent-rule drafts are versioned review artifacts. Keep the parent rule resource ID (`ruleId`) separate from each lifecycle snapshot ID (`versionId` or publish-specific `draftVersionId`). Published rules assigned to `botIds` can affect live behavior. Business Rules use a separate, non-versioned lifecycle.

## Reply Rules Are Not Rulebooks

Reply Rules are three fixed workspace-level slots for no reply, internal comment, and direct reply behavior. They do not use intent-rule drafts, Business Rule lifecycle, bot assignment, versions, publishing, or Rulebook evaluation. The current public surface is read-only.

Read Reply Rules together with Reply Rule fields context. Resolve condition paths, operators, quantifiers, installed sources, Attributes, and output-tag values from the returned workspace context. Never infer paths from display labels or hardcode workspace resources.

Report every slot separately. `isEnabled=false` means the slot is inactive; a slot with no saved ID and empty conditions is unconfigured, not an active rule or a failed read.

Use Conversations event metadata for evidence about one actual Fini response. Current Reply Rules describe current configuration and may not prove what was configured when an older event occurred.

## Choose The Safe Path

| Intent | Default |
| --- | --- |
| Natural-language behavior instructions | Generate a preview, inspect it, then persist it as an intent `DRAFT` |
| Revise generated behavior before launch | Refine the same intent rule into a new draft version |
| Launch approved draft to agents | Require and publish the explicitly supplied exact reviewed `draftVersionId` with explicit `botIds` |
| Inspect current draft or published behavior | List rules, then Get rule with the parent `ruleId`; optionally select `DRAFT` or `PUBLISHED` with `versionStatus` |
| Inspect or recover one exact version | List versions with the parent `ruleId`; get or restore with both parent `ruleId` and exact `versionId` |
| Clone existing behavior intentionally | Re-fetch exact source and kind, disclose possible live/assigned copy, require exact confirmation, duplicate, then inspect the independent copy |
| Manually edit an exact intent-rule tree for review | Re-fetch, update with explicit `status: DRAFT`, then verify the new draft; omit `botIds` from the draft update |
| Manually edit an exact live intent rule | Re-fetch, review the complete live-impacting diff, obtain immediate-live authorization, then update with explicit `status: PUBLISHED` |
| Validate a published saved rule | Get rule test fields, supply explicit context, then evaluate |
| Inspect an unsaved flow | Preview test fields from `flowConfig`, then review or persist; this does not execute the flow |
| Test a draft version of an existing assigned rule | Use its exact `ruleVersionId` through Generate Answer; the owning rule must already be published and assigned to that bot |
| Expand regression coverage | Generate suggestions, run accepted cases into controlled conversations, translate assertions into criteria, then add through the testing workflow |
| Remove behavior | Delete rule after exact-ID confirmation |

## Draft-First Workflow

1. Read the intent-rule field context before authoring the flow.
2. Capture behavior instructions in customer language.
3. Generate a preview and inspect its name, description, and flow tree. A preview is not persisted.
4. Persist the approved preview as an intent rule with status `DRAFT`.
5. Refine or manually edit by updating the same intent rule with explicit `status: DRAFT`. Omit `botIds` from the draft update, and do not duplicate first.
6. Re-fetch the draft with its parent `ruleId`, list versions under that parent, and record the exact `draftVersionId` that was reviewed. Do not substitute one identifier for the other.
7. Resolve target `botIds`.
8. Require that exact reviewed `draftVersionId` as an explicit publish input. Never omit it or resolve a current/latest draft after approval.
9. Publish only that reviewed version, after explicit approval bound to the version ID and with non-empty target agents.
10. Re-fetch the published rule and test with Generate Answer.

If no exact `draftVersionId` was reviewed, do not publish. List versions, fetch the complete candidate version, present it for review, and obtain approval for that identifier first.

## Duplication Workflow

Treat Duplicate rule as live-impacting for every rule kind. Do not assume an intent duplicate will be a harmless draft or that assignments will be cleared.

1. Fetch the exact source rule and record its returned `type`, state, `botIds`, full flow, and Action Tool nodes.
2. Reject a caller-supplied/default `type` that conflicts with the fetched rule. Never use the requested workflow to infer resource kind.
3. Explain that duplication creates an independent rule and can return a published or assigned copy.
4. Require exact confirmation bound to the source rule ID and the duplication effect. If the user already supplied that exact authorization, do not ask twice; pass the required confirmation phrase.
5. Duplicate the exact source rule.
6. Re-fetch the returned copy and report its actual ID, kind, state, assignments, and Action reachability. Never infer these from the source.

Duplication is not refinement. To revise an intent rule, refine the same rule into another draft version. Business Rules have no draft/refine path; changing one is an immediate-live update.

## Wire An Action Into A Rule

In active runtime, an Action is reachable only when a published rule assigned to the responding agent contains a Tool node that points at the Action ID. Generate Answer can execute a draft override only when that version belongs to a rule already published and assigned to the target bot; a brand-new unassigned draft is not available for this override.

1. Fetch the Action and verify its exact ID, required inputs, declared outputs, and Data Steps through `fini-api-actions-attributes`.
2. Fetch the rule field context and the full rule flow.
3. Add upstream Read or context bindings for every required Action input.
4. Add a Tool node referencing the exact Action ID.
5. Bind declared Action outputs to downstream Check, Reply, or Tool nodes.
6. Persist and review the intent rule as a draft.
7. Preview required fields and inspect the draft structurally. If the draft belongs to a rule already published and assigned to the target bot, route its exact-version runtime comparison through `fini-api-draft-validation`; otherwise no pre-publication execution path is available. Any eligible Tool-node run can have side effects.
8. Publish the reviewed rule version to explicit `botIds`.
9. Run a controlled Generate Answer proof against active configuration and inspect the resulting conversation.

Do not rely on Action assignment junctions. The backend stores those records but does not use them as a runtime gate. Rule assignment plus the Tool-node reference controls Action reachability. Attributes are different: assign them to agents through the Attribute workflow, configure their channels, and they run automatically for eligible messages.

## Rule Testing Workflow

1. For a published saved rule, fetch its complete flow and Get rule test fields.
2. For an unsaved flow, Preview rule test fields from the candidate `flowConfig`. This derives required context but does not execute the flow.
3. Build explicit test context for every required field. Do not invent customer identifiers.
4. Inspect the flow for Tool nodes before evaluation.
5. Evaluate the published saved rule with controlled inputs.
6. Capture node-level path/results, outputs, Action calls, and errors.
7. Generate suggested Rulebook tests when broader coverage is needed; treat the suggestions as review material, not persisted tests.
8. For each accepted suggestion, establish an eligible persistent case, translate the relevant assertions into deterministic or judge criteria, then add the conversation/case and criteria through `fini-api-testing`.
9. Use `fini-api-draft-validation` for one eligible Generate Answer candidate, controlled Generate Answer for real active behavior, and Test Suite for persistent published/draft regression using saved Action mocks. Real message generation retains its own external-effect rules.

Rule evaluation requires write scope and may execute external Actions. It is not a read-only preview when the flow can reach a Tool node.

## Existing Rule Workflow

1. List rules with explicit `type` and intent `versionStatus` when relevant. Preserve each parent `ruleId` separately from any returned `versionId`.
2. Fetch the full rule with its parent `ruleId` before explaining or mutating it. Bind all policy and lifecycle decisions to the returned kind, not a caller-supplied/default `type`.
3. Preserve `flowConfig` and `inputSchema` unless the user intends to change them.
4. For every intent-rule update, send status explicitly. Use `status: DRAFT` and omit `botIds` for review-first work; use `status: PUBLISHED` only after immediate-live authorization. Never rely on omission.
5. For intent rules only, inspect version history before restore, rollback, or publish. Restore creates a new draft; it does not silently replace live behavior.
6. For Business Rules, use the current rule directly or start from a documented default template. Create/update/delete/duplicate are immediate-live operations; do not apply intent draft, refine, restore, version-history, or publish semantics.
7. If the fetched kind conflicts with the requested workflow, stop and explain the correct lifecycle instead of coercing the rule through a cross-kind route.
8. Treat assignment changes in `botIds` and every duplication as live-impacting.
9. Re-fetch after write.

## Gotchas

- Product language may say "Rulebook"; current docs use "rule" and "rule draft".
- AI generation returns an unpersisted preview. Persist it explicitly before reporting a created draft.
- Intent-rule creation defaults to published unless the request explicitly sends `status: DRAFT`. Intent-rule update can also publish when status is omitted and a published version already exists. Never omit update status: send `DRAFT` for review-first work or explicit `PUBLISHED` only after immediate-live authorization. Business Rule creation and update are immediate and non-versioned.
- `List rules` defaults to `intent`; send `type=business` explicitly for Business Rules.
- Use `versionStatus` for intent draft/published/archived reads; do not rely on a stale `isDraft` convention.
- Get rule requires the parent `ruleId`; it does not accept a lifecycle `versionId` in that path position. `versionStatus` is optional and only selects `DRAFT` or `PUBLISHED`. A `404` after swapping those IDs is an identifier error, not proof that the version disappeared.
- Get one exact intent-rule version with both its parent `ruleId` and exact `versionId`. The exact-version route rejects a version that does not belong to the supplied parent.
- Preserve both identifiers when creating, refining, listing, fetching, publishing, restoring, or handing off a rule workflow.
- List summaries omit `flowConfig` and do not expand assignments. Fetch by ID for full tree.
- Draft intent rules return `botIds: []`.
- Publish requires at least one target `botId` and an explicitly supplied exact reviewed `draftVersionId`. Never omit the version ID or resolve a current/latest draft after approval.
- Refinement updates the same intent rule as a draft version. Duplication is not part of refinement.
- Intent rules alone have version history, drafts, refinement, restore, and publish.
- Business Rules are immediate-live and non-versioned; they do not have the intent draft/publish flow.
- Every duplicate is a live-impacting operation requiring exact-source confirmation. Re-fetch the returned copy because its state and assignments must not be inferred from rule kind.
- Template-based rules may not duplicate.
- An Action's name or description does not trigger it; only the Rulebook's flow decides when a Tool node runs.
- Action outputs must be declared and correctly mapped before downstream rule nodes can use them.
- Get/Preview rule test fields only derives required context. It does not prove behavior.
- Evaluate rule loads the published saved rule and uses synthetic test identifiers. It cannot execute an unsaved flow or a brand-new unassigned draft. Use conversation-rule evaluation when testing a real conversation.
- Rule evaluation can invoke Action Tool nodes and external APIs.
- Generated Rulebook tests are suggestions. Save reviewed conversation captures or authored scenarios as Test Suite cases, attach local/shared criteria, and preview the intended run with verified Action mocks.
- Delete is destructive.

## Proof

For any rulebook operation, report:

```markdown
Rule/rulebook: [name and ID]
Parent ruleId: [stable rule resource ID]
Type: [intent/business/unknown]
State: [draft/published/archived/deleted]
Version: [exact versionId and number / not versioned]
Agents: [botIds or not assigned]
Operation: [listed/fetched/drafted/refined/published/duplicated/updated/deleted]
Action wiring: [none / exact Action ID and Tool node verified]
Verification: [re-fetched/tested/generated tests/not requested]
Effects: [none observed / Action calls and results / unknown]
Next action: [review/refine/publish/test/none]
```
