---
name: fini-api-knowledge
description: Use when the user wants to generate Fini knowledge from raw text, inbox evidence, conversations, golden-set failures, or sources; bulk-generate from source IDs; poll knowledge jobs; manage live articles or draft articles; publish, update, delete, or fetch articles; inspect or organize the knowledge tree; move articles or folders; initialize or persist a tree; assign folders/knowledge to agents; or run a draft-review-to-publish improvement loop through the public API.
---

# Fini API Knowledge

Use this skill for the live knowledge layer and review workflow. It covers generation, article/draft management, tree organization, and assignment because those steps are usually one customer workflow.

Before endpoint-specific work, fetch `https://docs.usefini.com/llms.txt` and the relevant Knowledge, Manage Knowledge, or Organize Knowledge docs page.

## Choose The Path

| User intent | Path |
| --- | --- |
| Turn imported/changed source content into knowledge | Source-backed generation |
| Turn raw text, feedback, or inbox evidence into knowledge | Single generation job |
| Fix a golden-set failure caused by missing or stale KB content | Draft improvement loop |
| Write exact article text manually | Direct article management |
| Review before changing answers | Draft workflow |
| Make existing knowledge visible to an agent | Tree/folder assignment |
| Change where knowledge lives | Move article or folder |

## Workflow Gates

1. Resolve the target: source IDs, candidate text, article IDs, folder IDs, or `botId`.
2. Choose draft vs live. Default to draft when quality or assignment is uncertain.
3. Queue generation or article operation.
4. Poll async jobs when the route queues work.
5. Inspect result state: draft, live, no-op, failed, or duplicate.
6. Publish only after explicit confirmation if the request did not already authorize live change.
7. Verify folder placement and agent assignment when the user expects answers to change.
8. Test with Generate Answer only if the user wants behavioral proof.

## Defaults

- Prefer source-backed generation for imported source content.
- Prefer generation over direct article writes when the agent is transforming evidence into a KB entry.
- Prefer drafts for newly generated or refreshed knowledge.
- Prefer a draft improvement loop for golden-set failures and conversation evidence.
- Use `restrictedOps` when the user wants to prevent unexpected creates, updates, or duplicates.
- Treat publish, delete, folder delete, move, and assignment as live-impacting.

## Gotchas

- Generation defaults can leave content as draft; draft content does not change live answers.
- Source-backed generation depends on processed sources, not raw queued ingestion.
- `DO_NOTHING` can still produce reviewable output in draft mode.
- Direct article management is the manual/advanced path, not the default for source content.
- There is no need to memorize direct lookup paths; verify current article lookup docs before fetching by IDs.
- Knowledge can be live but invisible to an agent until assigned through the folder tree.
- A scoped tree for a `botId` can be empty because no folders are assigned, not because the workspace has no knowledge.

## Proof Of Completion

- Generation: job completed and produced the expected article/draft/no-op state.
- Draft review: draft exists, links back to original article or source evidence when applicable, and is not assumed live.
- Publish: live article exists after publish and draft state is resolved.
- Direct update: article fields changed as intended after re-fetch/list.
- Assignment: scoped tree for the target `botId` shows the intended folder.
- Behavior: Generate Answer or another runtime test uses the updated/assigned knowledge.
- Improvement loop: failed question has a draft/live fix, retest evidence, and a clear publish decision.

## References

- Read [generation-workflows.md](references/generation-workflows.md) for single and bulk generation, origins, restricted operations, polling, and review.
- Read [article-draft-workflows.md](references/article-draft-workflows.md) for direct article create/update/delete, draft, publish, and confirmation rules.
- Read [tree-assignment-workflows.md](references/tree-assignment-workflows.md) for folders, moves, tree import, and assignment to agents.
- Read [improvement-loop.md](references/improvement-loop.md) for golden-set failures, conversation evidence, draft fixes, retesting, publish decisions, and assignment verification.
