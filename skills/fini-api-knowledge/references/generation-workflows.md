# Knowledge Generation Workflows

Use this reference for generating Fini knowledge from candidate text, inbox/conversation evidence, or processed sources. Fetch current docs first for exact fields, job status values, and route names.

## Choose Generation Type

| Input | Preferred path |
| --- | --- |
| Processed source IDs | Bulk source-backed generation |
| Raw text supplied by user/system | Single generation from candidate text |
| Conversation or inbox evidence | Single generation with evidence linkage |
| Golden-set failure caused by missing/stale KB | Draft improvement loop |
| Manual final article text | Direct article management, not generation |

## Source-Backed Generation

1. Confirm sources finished ingestion/refresh.
2. Use only source IDs that should become knowledge.
3. Default to draft.
4. Use restricted operations when the caller wants to limit add/update/no-op behavior.
5. Queue bulk generation.
6. Poll jobs.
7. Review result state before publish.

## Single Generation

1. Confirm candidate text is the content to transform.
2. Choose origin and required linkage IDs from current docs.
3. Default to draft.
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

## Validation

Generation is not complete until one of these is true:

- Draft exists and is ready for review.
- Live article exists and is in the intended folder.
- No-op/duplicate result is confirmed and acceptable.
- Failure is reported with enough context to retry or debug.
