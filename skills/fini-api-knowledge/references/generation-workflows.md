# Knowledge Generation Workflows

Use this reference for generating Fini knowledge from candidate text, inbox/conversation evidence, or processed sources. Fetch current docs first for exact fields, job status values, and route names.

## Knowledge Lifecycle

Knowledge is the factual retrieval layer. A complete behavior-changing workflow usually needs:

1. Candidate content or processed source IDs.
2. A usable folder tree when creating source-backed knowledge.
3. Draft generation job.
4. Job polling and result inspection.
5. Review/publish decision.
6. Folder placement.
7. Agent folder assignment.
8. Generate Answer proof.

Drafts, unassigned folders, and unprocessed sources are not enough to change what an agent can answer.

## Choose Generation Type

| Input | Preferred path |
| --- | --- |
| Processed source IDs | Bulk source-backed generation |
| Raw text or extracted PDF/file content | Single Generate knowledge job with `candidateKnowledge`, `origin: "generated"`, `isDraft: true` |
| Conversation or inbox evidence | Single generation with evidence linkage |
| Golden-set failure caused by missing/stale KB | Draft improvement loop |
| Manual final article text | Direct article management, not generation |

## Source-Backed Generation

1. Confirm sources finished ingestion/refresh.
2. Confirm there is a usable folder tree with a likely target folder. If the workspace is empty, create or initialize/persist folders first.
3. Use only source IDs that should become knowledge.
4. Default to draft.
5. Use restricted operations when the caller wants to limit add/update/no-op behavior.
6. Queue bulk generation.
7. Poll jobs.
8. Require article/draft IDs before treating generation as successful.
9. Review result state before publish.

## Single Generation

1. Confirm candidate text is the content to transform.
2. For raw text or extracted PDF/file content, use `origin: "generated"` with no source/event linkage ID.
3. Set `isDraft: true` explicitly, even if current docs default it.
4. Use restricted operations if the user wants constraints.
5. Queue job.
6. Poll.
7. Inspect generated article/draft/no-op result.

## Draft vs Live

Default to draft when:

- The source is new.
- The input is customer feedback or conversation evidence.
- The user has not approved final wording.
- Assignment/visibility is not yet understood.
- The input is a golden-set failure or QA finding.

Live generation is appropriate only when:

- The user explicitly wants immediate live change.
- The caller accepts retrieval behavior changing.
- Folder placement and assignment are known or will be verified immediately.

## Job Polling Output

```text
Knowledge job status
- Job IDs:
- Source/candidate linkage:
- Terminal status:
- Article/draft IDs:
- No-op or duplicate result:
- Errors:
- Next action:
```

If a job is `COMPLETED` but has no article/draft ID, do not proceed to publish or assignment. Treat it as blocked/no-op, inspect the source and tree state, and report the missing persistence proof.

## Validation

Generation is not complete until one of these is true:

- Draft exists and is ready for review.
- Live article exists and is in the intended folder.
- No-op/duplicate result is confirmed and acceptable.
- Failure is reported with enough context to retry or debug.

For source-backed generation, also verify one of:

- The job status includes an article/draft ID and List articles can fetch it.
- The source record now links to knowledge when the workflow is expected to sync source linkage.
- A no-op is explicitly acceptable to the user.
