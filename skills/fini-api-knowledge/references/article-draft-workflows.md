# Article And Draft Workflows

Use this reference for direct article management, drafts, publishing, updates, history inspection, exact-version retrieval, rollback, and deletion. Fetch current docs first for exact route names and fields.

## Contents

- [When To Use Direct Article Management](#when-to-use-direct-article-management)
- [Draft Workflow](#draft-workflow)
- [Update Workflow](#update-workflow)
- [History Inspection](#history-inspection)
- [Revert Workflow](#revert-workflow)
- [Delete Workflow](#delete-workflow)
- [Gotchas](#gotchas)
- [Templates](#publish-confirmation-template)

## When To Use Direct Article Management

Use direct article APIs when:

- The user already has final article text.
- The user wants to create or update a specific article manually.
- The task is lifecycle control: draft, publish, update, move, or delete.
- The user wants to inspect or restore a saved article version.
- The caller needs exact deterministic article content.

Do not use direct article create as the default way to convert sources or messy evidence into knowledge; use generation first.

## Draft Workflow

1. List relevant articles for discovery, then fetch every selected live or draft article by exact ID. Reconcile requested IDs against the returned IDs because exact lookup returns only the articles it finds.
2. Resolve the target folder. A draft created from a live article should inherit that article's `parentFolderId` unless the user explicitly wants to move it.
3. Create a draft from the existing live article, or create a new draft article with an explicit target folder.
4. Make changes.
5. Re-fetch the exact draft ID to verify current existence, draft state, and folder.
6. Present diff or summary for approval.
7. When the intended behavior needs proof, use `fini-api-draft-validation` for an artifact-free baseline and a controlled marked candidate with the exact draft ID in `artifacts.knowledgeArticleDraftIds`.
8. Immediately before publishing, fetch the exact target again when read scope is available and reconcile the requested/found ID plus draft provenance. Do not require `isDraft: true`; valid drafts derived from live articles can report `isDraft: false`. Use the dedicated write-only publish endpoint as the authoritative target check.
9. Publish only after explicit confirmation, using the complete required article payload and a valid `parentFolderId`.
10. Treat the returned live article/state as authoritative success proof. Then perform a secondary exact lookup for the old draft ID: expect it to be absent after consumption, and flag unexpected persistence for investigation.
11. Rerun the same case without draft artifacts to prove active knowledge.

## Update Workflow

1. Fetch the target article by ID using current docs.
2. Confirm whether the change should happen live or in a draft.
3. Apply update.
4. Re-fetch/list to verify changed fields.
5. If live, consider testing retrieval through the target agent.

## History Inspection

1. Fetch the current live article and preserve its content, folder, and relevant metadata.
2. List the article's prior saved history snapshots. This history list does not include the current live head.
3. Select the exact numeric `version`; do not infer a version from a timestamp, `versionId`, or phrase such as "the previous one."
4. Fetch that exact version and compare its content with the current live article.
5. Report the current live version, latest prior history version, selected version number, creation provenance/timestamp when present, and the material field differences. Current version `N` with history ending at `N-1` is expected.
6. Stop without writing when the user only asked to inspect history.

The exact-version read is read-scoped even though the current route uses `POST` to carry `version` in its body.

## Revert Workflow

Revert writes immediately to live knowledge. If the user wants a review step, copy the exact historical content into a draft and use the normal draft workflow instead.

1. Capture the current live article so the pre-revert state remains available.
2. List history and fetch the exact numeric target version.
3. Compare current vs target fields, including title, main knowledge, agent instruction, questions, keywords, escalation, and folder placement when present.
4. Confirm the exact article ID and version when the request has not already authorized that rollback.
5. Revert the exact numeric version.
6. Re-fetch the current live article and compare it field-by-field with the target snapshot.
7. Verify folder placement and target-agent visibility.
8. Use controlled Generate Answer for a separately requested active smoke. For persistent coverage, use Test Suite with saved cases, local/shared criteria, and selected published/draft configuration; Actions are simulated from saved mocks.

Do not treat the revert response alone as proof. The completion record must include the pre-revert snapshot identity, target version, returned article, verification read, and any field that did not match.

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
- Article lists are discovery surfaces. If a draft ID is known, exact lookup is authoritative for whether that ID currently exists.
- Exact lookup may return a successful response containing only the IDs it found. Compare requested and returned IDs before validating, publishing, reverting, updating, or deleting.
- Do not use `isDraft: true` as a mandatory publish gate. Drafts derived from live articles can carry draft provenance while reporting `isDraft: false`; the dedicated publish endpoint is the authoritative target check and remains write-scope-only.
- Do not infer automatic expiry from a missing draft. A missing ID may have been deleted, already published and consumed, or otherwise no longer persisted.
- Generate Answer can test an exact draft through `artifacts.knowledgeArticleDraftIds`; that request-scoped proof does not publish the draft. After publication, remove the override and rerun the same case.
- History entries are prior saved snapshots; they do not include the current live article. A current version one higher than the latest history version is normal.
- Publishing consumes the draft and returns the live article that remains. Treat that returned live article/state as primary proof; check old-draft absence secondarily and flag unexpected persistence rather than making absence the only success signal.
- A version read does not change the article.
- Revert expects the numeric `version` and changes the knowledge graph immediately. Do not send `versionId` unless the current docs explicitly require it.
- A rollback can restore content while still leaving an assignment problem. Verify folder and scoped-agent visibility after reverting.

## Publish Confirmation Template

```text
Publish plan
- Draft ID:
- Exact draft review evidence (requested/found ID, provenance, and lifecycle fields; do not gate solely on `isDraft`):
- Original live article:
- Intended live title:
- Folder:
- Agents expected to see it:
- Resulting live article:
- Secondary consumed-draft absence check:
- Verification after publish:
- Confirmation required:
```

## Revert Evidence Template

```text
Article revert
- Article ID:
- Pre-revert snapshot:
- Available history versions:
- Exact target version:
- Current vs target differences:
- Live-write authorization:
- Revert response:
- Verification read:
- Folder / agent visibility:
- Behavioral retest:
- Remaining mismatch:
```
