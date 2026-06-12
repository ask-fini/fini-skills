# Tree And Assignment Workflows

Use this reference for knowledge folders, tree organization, article moves, folder moves, tree import, and folder assignment to agents. Fetch current docs first for exact route names and payloads.

## Visibility Model

Knowledge being live is not always enough. If a customer expects an agent to use knowledge, verify:

- Article exists.
- Article is in the intended folder.
- Folder is active/available according to current docs.
- Folder is assigned to the target agent.
- Scoped tree for the agent includes the folder.

## Assign Folder To Agent

1. Resolve agent name to `botId`.
2. Fetch full tree and identify the folder ID.
3. If a scoped tree exists for the agent, inspect current visibility first.
4. Plan assignment or unassignment.
5. Execute only after confirmation when changing production visibility.
6. Re-fetch scoped tree for the `botId`.
7. Optionally test Generate Answer for behavior.

## Move Article Or Folder

1. Fetch current tree.
2. Confirm current parent and target parent.
3. Explain likely retrieval/organization impact.
4. Execute move.
5. Re-fetch tree and verify placement.
6. Re-check agent scoped tree if visibility matters.

## Initialize Or Persist Tree

Use tree initialization/persist routes when setting up a workspace structure, not for ordinary small article moves.

Before persisting:

- Review the generated or edited tree file.
- Confirm folder names and intended hierarchy.
- Confirm whether existing structure will be changed.
- Plan article placement and assignments after import.

## Output Template

```text
Knowledge visibility check
- Agent:
- Folder:
- Article IDs:
- Full tree contains folder:
- Scoped tree contains folder:
- Assignment action:
- Verification:
- Remaining risk:
```
