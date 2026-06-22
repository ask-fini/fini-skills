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

Drafts are review artifacts. Published rules assigned to `botIds` can affect live behavior.

## Choose The Safe Path

| Intent | Default |
| --- | --- |
| Natural-language behavior instructions | Create rule draft |
| Revise generated behavior before launch | Refine rule draft |
| Launch approved draft to agents | Publish rule draft with explicit `botIds` |
| Inspect current behavior | List rules, then Get rule by ID |
| Clone existing behavior | Duplicate rule, then inspect draft/live state |
| Manually edit exact tree or assignments | Update rule after re-fetch and approval |
| Remove behavior | Delete rule after exact-ID confirmation |

## Draft-First Workflow

1. Capture behavior instructions in customer language.
2. Create a draft rule with `behaviorInstructions`.
3. Inspect the returned draft. Do not assume it is assigned to any agent.
4. Refine the draft if the user wants changes.
5. Resolve target `botIds`.
6. Publish only after explicit approval and non-empty target agents.
7. Re-fetch the rule and test with Generate Answer.

## Existing Rule Workflow

1. List rules with explicit `type` when the user says intent or business.
2. Fetch the full rule by ID before explaining, updating, or duplicating.
3. Preserve `flowConfig` and `inputSchema` unless the user intends to change them.
4. Treat assignment changes in `botIds` as live-impacting.
5. Re-fetch after write.

## Gotchas

- Product language may say "Rulebook"; current docs use "rule" and "rule draft".
- `Create rule` creates a published rule directly. Use it only for exact, approved live creation.
- `List rules` defaults to `intent`; send `type=business` explicitly for Business Rules.
- List summaries omit `flowConfig` and do not expand assignments. Fetch by ID for full tree.
- Draft rules return `botIds: []`.
- Publish requires at least one target `botId`.
- Duplicating an `intent` rule creates a draft copy; duplicating a `business` rule creates a non-draft copy.
- Template-based rules may not duplicate.
- Delete is destructive.

## Proof

For any rulebook operation, report:

```markdown
Rule/rulebook: [name and ID]
Type: [intent/business/unknown]
State: [draft/published/deleted]
Agents: [botIds or not assigned]
Operation: [listed/fetched/drafted/refined/published/updated/deleted]
Verification: [re-fetched/tested/not requested]
Next action: [review/refine/publish/test/none]
```
