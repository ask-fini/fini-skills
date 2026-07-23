# Rulebook And Rule Workflows

Use this reference when the user says rulebook, rule, business rule, intent rule, escalation workflow, routing behavior, or asks to generate behavior from instructions.

## Rulebook Mental Model

Rules/rulebooks are behavior and workflow controls for existing agents. Use them when the desired change is about what the agent should do in a class of situations, not when the issue is simply missing documentation.

Common rulebook jobs:

- route specific intents
- trigger escalation or handoff behavior
- guide structured workflow steps
- use tags or input schema as conditions
- constrain behavior around sensitive or high-risk situations

Intent-rule drafts are versioned review artifacts. Published rules assigned to `botIds` can affect live behavior. Business Rules use a separate, non-versioned lifecycle.

## Choose The Safe Path

| Intent | Default |
| --- | --- |
| Natural-language behavior instructions | Generate a preview, inspect it, then persist it as an intent `DRAFT` |
| Revise generated behavior before launch | Refine the same intent rule into a new draft version |
| Launch approved draft to agents | Publish the exact reviewed `draftVersionId` with explicit `botIds` |
| Inspect current behavior | List rules, then Get rule by ID |
| Inspect or recover history | List/get versions; restore the selected version as a new draft |
| Clone existing behavior intentionally | Duplicate rule, then inspect the independent copy |
| Manually edit exact tree or assignments | Update rule after re-fetch and approval |
| Remove behavior | Delete rule after exact-ID confirmation |

## Draft-First Workflow

1. Read the intent-rule field context before authoring the flow.
2. Capture behavior instructions in customer language.
3. Generate a preview and inspect its name, description, and flow tree. A preview is not persisted.
4. Persist the approved preview as an intent rule with status `DRAFT`.
5. Refine by regenerating from the current flow and updating the same rule as `DRAFT`. Do not duplicate first.
6. Re-fetch the draft and its versions; record the exact `draftVersionId` that was reviewed.
7. Resolve target `botIds`.
8. Publish only that reviewed version, after explicit approval and with non-empty target agents.
9. Re-fetch the published rule and test with Generate Answer.

## Existing Rule Workflow

1. List rules with explicit `type` and intent `versionStatus` when relevant.
2. Fetch the full rule by ID before explaining, updating, or duplicating.
3. Preserve `flowConfig` and `inputSchema` unless the user intends to change them.
4. For intent rules, inspect version history before restore, rollback, or publish. Restore creates a new draft; it does not silently replace live behavior.
5. For Business Rules, use the current rule directly or start from a documented default template; do not apply intent-version semantics.
6. Treat assignment changes in `botIds` as live-impacting.
7. Re-fetch after write.

## Gotchas

- Product language may say "Rulebook"; current docs use "rule" and "rule draft".
- AI generation returns an unpersisted preview. Persist it explicitly before reporting a created draft.
- `Create rule` creates a published rule directly. Use it only for exact, approved live creation.
- `List rules` defaults to `intent`; send `type=business` explicitly for Business Rules.
- Use `versionStatus` for intent draft/published/archived reads; do not rely on a stale `isDraft` convention.
- List summaries omit `flowConfig` and do not expand assignments. Fetch by ID for full tree.
- Draft intent rules return `botIds: []`.
- Publish requires at least one target `botId` and the exact reviewed `draftVersionId`.
- Refinement updates the same intent rule as a draft version. Duplication is not part of refinement.
- Business Rules are not versioned and do not have the intent draft/publish flow.
- Duplicating an `intent` rule creates a draft copy; duplicating a `business` rule creates a non-draft copy.
- Template-based rules may not duplicate.
- Delete is destructive.

## Proof

For any rulebook operation, report:

```markdown
Rule/rulebook: [name and ID]
Type: [intent/business/unknown]
State: [draft/published/archived/deleted]
Version: [version ID/number or not versioned]
Agents: [botIds or not assigned]
Operation: [listed/fetched/drafted/refined/published/updated/deleted]
Verification: [re-fetched/tested/not requested]
Next action: [review/refine/publish/test/none]
```
