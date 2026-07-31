---
name: fini-api-knowledge
description: Use when the user wants to generate Fini knowledge from raw text, PDFs/files converted to text, inbox evidence, conversations, golden-set failures, or sources; create draft KB articles through Generate knowledge with candidateKnowledge and isDraft; bulk-generate from source IDs; poll knowledge jobs; manage articles/drafts; inspect article history or one exact saved version; revert and verify an article version; or publish, update, delete, move, organize, and assign knowledge through the public API.
---

# Fini API Knowledge

Use this skill for the live knowledge layer and review workflow. It covers generation, article/draft management, tree organization, and assignment because those steps are usually one customer workflow.

Before endpoint-specific work, fetch `https://docs.usefini.com/llms.txt` and the relevant Knowledge, Manage Knowledge, or Organize Knowledge docs page.

## Choose The Path

| User intent | Path |
| --- | --- |
| Turn imported/changed source content into knowledge | Source-backed generation |
| Turn a PDF/file/raw text into a KB draft | Single Generate knowledge job with `candidateKnowledge`, `origin: "generated"`, `isDraft: true` |
| Turn feedback or inbox evidence into knowledge | Single generation job with evidence linkage |
| Fix a golden-set failure caused by missing or stale KB content | Draft improvement loop |
| Fix wrong prompt, rulebook, routing, escalation, or tag behavior | Use `fini-api-agent-configuration`, not Knowledge |
| First KB setup in an empty workspace | Create or initialize/persist a folder tree before source-backed generation |
| Write exact article text manually | Direct article management |
| Review before changing answers | Draft workflow |
| Inspect what an article contained at a prior version | History list -> exact version read |
| Restore a saved article version immediately | Exact version comparison -> live revert -> verification |
| Make existing knowledge visible to an agent | Tree/folder assignment |
| Change where knowledge lives | Move article or folder |

## Workflow Gates

1. Resolve the target: source IDs, extracted candidate text, article IDs, folder IDs, or `botId`.
2. For source-backed generation, verify a usable folder tree exists first. If articles/folders are empty or the snapshot looks stale, bootstrap folders before generating.
3. For raw PDF/file/text payloads, extract supportable facts and call Generate knowledge as a draft by default: `candidateKnowledge`, `origin: "generated"`, optional `botId`, `isDraft: true`.
4. Choose draft vs live. Default to draft when quality or assignment is uncertain.
5. Queue generation or article operation.
6. Poll async jobs when the route queues work.
7. Inspect result state: draft, live, no-op, failed, or duplicate. `COMPLETED` with no article/draft ID is blocked/no-op, not success.
8. For a review draft that is expected to change answers, route to `fini-api-draft-validation`: run an artifact-free baseline, fetch exact persisted draft IDs, test `artifacts.knowledgeArticleDraftIds` with `markAsTest: true`, and preserve the comparison.
9. For history work, list available versions, fetch the exact numeric version, and compare it with the current live article before choosing draft restoration or immediate revert.
10. Treat revert as an immediate live write. Capture the current version first, restore only the exact reviewed version, then re-fetch and compare the live article with the target snapshot.
11. Publish or revert only after explicit confirmation if the request did not already authorize the exact live change.
12. Verify folder placement and agent assignment when the user expects answers to change.
13. When behavioral proof is requested, after publish or revert run or rerun the case without draft artifacts to prove active behavior.

## Defaults

- Prefer source-backed generation for imported source content.
- Prefer Generate knowledge over direct article writes when transforming raw text, PDFs/files, feedback, or other evidence into a KB entry.
- Prefer drafts for newly generated or refreshed knowledge; set `isDraft: true` explicitly even if current docs default it.
- For fresh workspaces, create or initialize a folder tree before source-backed generation.
- Prefer a draft improvement loop for golden-set failures and conversation evidence.
- Use `restrictedOps` when the user wants to prevent unexpected creates, updates, or duplicates.
- Use history reads before rollback; fetch a numeric version's content rather than inferring it from a label or timestamp.
- If the user wants review before restoration, copy the exact historical content into a draft instead of calling revert.
- When behavior proof is part of the request, use `fini-api-draft-validation` with exact `knowledgeArticleDraftIds` before publishing, then rerun the same marked case against active configuration after publication.
- Treat publish, revert, delete, folder delete, move, and assignment as live-impacting.

## Gotchas

- Generation defaults can leave content as draft; draft content does not change live answers.
- Source-backed generation depends on processed sources, not raw queued ingestion.
- As an observed operational safeguard, verify a usable tree/folder target before source-backed generation; empty or stale trees can produce no-op jobs even though raw source ingestion itself does not require this ordering.
- A completed generation job with `hcArticleId: null` did not create or update knowledge.
- `DO_NOTHING` can still produce reviewable output in draft mode.
- Direct article management is the manual/advanced path, not the default for source content.
- There is no need to memorize direct lookup paths; verify current article lookup docs before fetching by IDs.
- Listing article history returns saved versions; fetching one exact history version uses a read-scoped `POST`. HTTP method alone does not make that read a live write.
- Revert accepts the numeric `version`, writes to the knowledge graph immediately, and returns the restored live article. Do not substitute `versionId` unless current docs explicitly change the contract.
- A successful revert response is not enough proof. Re-fetch the live article, compare the restored fields with the exact target snapshot, and verify folder/agent visibility.
- Knowledge can be live but invisible to an agent until assigned through the folder tree.
- A scoped tree for a `botId` can be empty because no folders are assigned, not because the workspace has no knowledge.
- Folder reads are snapshot-style reads in the public API; if snapshots disagree with current agents/articles/sources, stop and flag stale state.
- Wrong prompt/rule/tag behavior is an agent-configuration issue, not a reason to create more KB content by default.

## Proof Of Completion

- Generation: job reached terminal state and produced the expected article/draft ID, or a no-op/duplicate was explicitly accepted.
- Draft review: draft exists, links back to original article or source evidence when applicable, is not assumed live, and—when behavior proof is required—has an exact-artifact Generate Answer result.
- Publish: live article exists after publish, draft state is resolved, and the same case passes without draft artifacts when behavioral proof was requested.
- Direct update: article fields changed as intended after re-fetch/list.
- History inspection: report the article ID, available version numbers, exact version fetched, provenance/timestamp when present, and the differences from current live content.
- Revert: current pre-revert snapshot and target version were captured, the exact numeric version was restored, the live article matches the intended historical fields after re-fetch, and folder/agent visibility remains correct.
- Assignment: scoped tree for the target `botId` shows the intended folder.
- Behavior: a marked Generate Answer smoke uses the updated/assigned knowledge; any broader Test Set run evaluates current published live-agent behavior and may invoke Actions or external APIs.
- Improvement loop: failed question has a draft/live fix, retest evidence, and a clear publish decision.

## References

- Read [generation-workflows.md](references/generation-workflows.md) for single and bulk generation, origins, restricted operations, polling, and review.
- Read [article-draft-workflows.md](references/article-draft-workflows.md) for direct article lifecycle, history inspection, exact-version reads, revert verification, and confirmation rules.
- Read [tree-assignment-workflows.md](references/tree-assignment-workflows.md) for folders, moves, tree import, and assignment to agents.
- Read [improvement-loop.md](references/improvement-loop.md) for golden-set failures, conversation evidence, draft fixes, retesting, publish decisions, and assignment verification.
