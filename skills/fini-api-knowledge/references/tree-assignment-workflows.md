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

For fresh onboarding, do this before source-backed generation. Magic Articles/source-backed generation needs a real tree/folder target; an empty or stale tree can lead to completed jobs with no article created.

Before persisting:

- Review the generated or edited tree file.
- Confirm folder names and intended hierarchy.
- Confirm whether existing structure will be changed.
- Plan article placement and assignments after import.

If the current tool surface cannot call tree initialize/persist because it requires file download/upload, create a minimal starter tree through folder-create operations instead. Verify the created folders before generating knowledge.

## Snapshot Sanity

The public folder read returns a knowledge-tree snapshot. Before using it for onboarding, compare it with current agents, article counts, source state, and scoped `botId` visibility.

Block instead of proceeding when:

- The snapshot shows mature folders but List articles returns no articles.
- The snapshot references bot IDs that do not appear in List agents.
- The unscoped tree and `botId`-scoped tree tell incompatible stories.
- Recent folder creation does not appear after re-fetching.

## Output Template

```text
Knowledge visibility check
- Agent:
- Folder:
- Article IDs:
- Full tree contains folder:
- Scoped tree contains folder:
- Snapshot sanity:
- Assignment action:
- Verification:
- Remaining risk:
```
