# Tag And Tag Group Workflows

Use this reference when the user wants classification, routing labels, output labels, analytics tags, or rulebook conditions.

## Tag Taxonomy Model

Tags are shared vocabulary for classification, routing, reporting, and downstream review. Design the taxonomy before mutating it:

- Routing tags should be stable and available to Rulebooks.
- Output/reporting tags can describe the final state or outcome after a conversation.
- Tag names should be short; descriptions and prompts carry the distinction rules.
- A tag group should answer one classification question. Do not mix unrelated dimensions in one group.

## Design First

Before creating tags, classify the group:

- Routing/rulebook condition: keep `isOutputTagGroup` false.
- Post-answer reporting/output classification: `isOutputTagGroup` can be true.
- Single-label taxonomy: `multiselect=false`.
- Multi-label taxonomy: `multiselect=true` only when multiple tags can legitimately apply.

Keep tag names short and stable. Put disambiguation in `tagDescription` or group `prompt`.

## Read Workflow

1. List tag groups.
2. Identify workspace-owned custom groups versus Fini defaults.
3. List tags in one group or across groups.
4. Regroup flat cross-group responses by `tagGroupId` when needed.

## Mutation Workflow

1. Verify the target is a workspace-owned custom group or tag.
2. For new routing tags, create/update the custom group with `isOutputTagGroup=false`.
3. Create tags inside that group.
4. Re-list the group and tags.
5. Check rulebook availability if the tags will be used by rules.
6. Require explicit confirmation before delete operations.

## Gotchas

- Fini-managed default groups may be visible but are not the supported write target.
- `List tags across groups` uses repeated `tagGroupIds` query params, not comma-separated values.
- `Create tag` uses the tag group ID in the path.
- `Update tag` and `Delete tag` use the tag ID in the path, not the group ID.
- Unknown tag/group IDs may return `500` or `null`; list known objects first rather than trusting a guessed ID.
- Deleting tag groups can affect routing, reporting, and rules that depend on those tags.

## Output Shape

For tag design or mutation, report:

```markdown
Tag group: [title and ID]
Purpose: [rulebook/routing/output/reporting]
Multiselect: [true/false]
Available to Rulebooks: [yes/no]
Tags: [created/updated/listed count]
Write status: [not run / completed / failed]
Verification: [re-listed / not checked]
Risk: [none / affects rules / affects reporting / destructive]
```
