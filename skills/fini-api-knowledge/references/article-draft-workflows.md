# Article And Draft Workflows

Use this reference for direct article management, drafts, publishing, updates, and deletion. Fetch current docs first for exact route names and fields.

## When To Use Direct Article Management

Use direct article APIs when:

- The user already has final article text.
- The user wants to create or update a specific article manually.
- The task is lifecycle control: draft, publish, update, move, or delete.
- The caller needs exact deterministic article content.

Do not use direct article create as the default way to convert sources or messy evidence into knowledge; use generation first.

## Draft Workflow

1. Fetch or list relevant articles.
2. Resolve the target folder. A draft created from a live article should inherit that article's `parentFolderId` unless the user explicitly wants to move it.
3. Create a draft from the existing live article, or create a new draft article with an explicit target folder.
4. Make changes.
5. Re-read/list to verify draft state and folder.
6. Present diff or summary for approval.
7. Publish only after explicit confirmation.
8. Verify live article after publish.

## Update Workflow

1. Fetch the target article by ID using current docs.
2. Confirm whether the change should happen live or in a draft.
3. Apply update.
4. Re-fetch/list to verify changed fields.
5. If live, consider testing retrieval through the target agent.

## Delete Workflow

Deletion is destructive. Before deleting:

- Confirm exact article ID.
- Confirm whether it is live or draft.
- Explain whether linked sources remain.
- Explain likely impact on agent answers.
- Wait for explicit confirmation.

After deleting, report response evidence and whether follow-up source cleanup or assignment checks are needed.

## Gotchas

- `parentFolderId` is required when creating an article draft. Do not call the route with an empty folder.
- Preserve the source article's folder by default. Moving knowledge is a separate decision because folder assignment controls agent visibility.
- A persisted draft does not affect live answers until it is published and visible to the target agent.

## Publish Confirmation Template

```text
Publish plan
- Draft ID:
- Original live article:
- Intended live title:
- Folder:
- Agents expected to see it:
- Verification after publish:
- Confirmation required:
```
