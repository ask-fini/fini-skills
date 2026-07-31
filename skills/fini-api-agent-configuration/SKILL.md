---
name: fini-api-agent-configuration
description: Use when the user wants to configure an existing Fini agent through public APIs; inspect prompt history or exact versions; create, duplicate, refine, publish, restore, test, update, or delete Rulebooks/rules with the correct intent-versus-Business Rule lifecycle; create, review, publish, rebase, or directly save prompt versions; wire Actions into Rulebook Tool nodes; manage tags and tag groups; or validate configuration before runtime testing.
---

# Fini API Agent Configuration

Use this skill for existing-agent behavior configuration: prompts, rulebooks/rules, and tags/tag groups. Route agent creation to `fini-api-agents`, Action/Attribute construction to `fini-api-actions-attributes`, unpublished exact-ID behavior comparisons to `fini-api-draft-validation`, and live regression suites or Replays to `fini-api-testing`.

Before endpoint-specific work, fetch `https://docs.usefini.com/llms.txt` and the relevant Prompts, Rules, Tags, or Tag Groups docs page.

## Choose The Layer

| User intent | Path |
| --- | --- |
| "Show the bot prompt or history" | Current merged prompt -> history -> exact version when needed |
| "Change how the agent behaves globally" | Current prompt -> draft -> exact review -> exact publish -> Generate Answer |
| "Make this exact prompt live immediately" | Direct prompt update only when the caller explicitly chooses the live path |
| "Test answers, diagnose failures, and change the guidelines" | Active baseline -> prompt diff -> exact draft validation -> review/publish -> marked active proof |
| "Create a rulebook/rule from instructions" | Generate preview -> persist intent draft |
| "Refine this rulebook before launch" | Refine the same intent rule as a new draft version |
| "Publish this rulebook to one bot" | Require the explicit exact reviewed `draftVersionId`, then publish it with explicit agent IDs |
| "Inspect or restore an older rule version" | Rule version history -> restore as draft |
| "Duplicate this rule" | Re-fetch the exact source and actual rule kind -> disclose possible live/assigned copy -> exact confirmation -> duplicate -> inspect the copy |
| "Test a published saved rule" | Saved test fields -> inspect inputs -> evaluate with explicit context |
| "Inspect an unsaved flow or brand-new draft" | Preview test fields -> structural review; persist/review before a supported execution path |
| "Make this Action available to an agent" | Put its ID in a Rulebook Tool node -> publish that rule to the agent |
| "Create routing or outcome tags" | Tag group and tag workflow |
| "Configure a Slack/support agent" | Tags -> rule drafts -> prompts -> tests, then Sources/Knowledge if content is missing |

## Workflow Gates

1. Resolve the target agent with List agents when the user gives a name.
2. Read current state before writing: prompts/history, rule list/get, tag groups/tags. For a rule mutation, re-fetch the exact rule and use its returned kind; never choose draft or live policy from a caller-supplied/default `type`.
3. Prefer reviewable, versioned paths. For prompt changes, create a draft, fetch that exact version, require a complete non-truncated payload, compare it with the current prompt, and publish only the reviewed `versionId`. For intent rules, generate a preview, persist it as `DRAFT`, refine the same rule, and publish the exact reviewed draft version.
4. Preserve and submit all three prompt arrays even when only one section changes.
5. Write only to workspace-owned custom tag groups/tags; treat Fini-managed defaults as read-only.
6. Re-fetch after writes and report changed IDs, exact version, draft/live state, assigned `botIds`, and remaining review steps.
7. Inspect rule test fields before evaluating a published saved rule. Preview fields for an unsaved flow are structural guidance, not execution. Treat supported evaluation as potentially effectful when a flow reaches an Action Tool node.
8. Inspect conversations when the user needs QA, rollout, or golden-set evidence.
9. Route unpublished prompt/knowledge/rule candidate comparisons to `fini-api-draft-validation`; test final active behavior with marked Generate Answer; route broad persistent live regression to `fini-api-testing`.

## Defaults

- Prefer intent-rule preview -> persisted `DRAFT` for natural-language rulebook requests.
- Read rule field context before authoring a flow tree. Use default Business Rule templates when the request matches one.
- Prefer prompt read -> draft -> exact fetch/review -> exact publish. Use direct Update prompts only for an explicitly authorized immediate live update.
- If a prompt draft is stale, perform a three-way rebase: fetch the stale draft, fetch its exact `parentVersionId` version, identify and fetch the latest active lifecycle version, derive the draft-vs-parent diff, detect conflicts with active-vs-parent changes, then apply only the reviewed non-conflicting diff to a new full-array draft. Read the current merged view separately when workspace-template changes also matter. Do not retry publishing the stale ID.
- For prompt improvement work, run the same small set of cases before and after the change so the behavioral effect is attributable.
- Prefer custom tag groups for customer-specific classification.
- Leave tag groups available to Rulebooks unless the user explicitly wants output-only reporting tags.
- Wire Actions through Rulebook Tool nodes. Action assignment records are not runtime gates; active runtime reachability comes from rule publication and rule-to-agent assignment. An exact draft override can execute only when its owning rule is already published and assigned to the target bot.
- Assign Attributes through the Attribute workflow; they run automatically only for eligible messages on their configured agent/channel assignments.
- Treat prompt publish/direct update, rule evaluation with Tool nodes, published rule create/update/delete, every rule duplicate, rule publish, and tag deletion as live-impacting.

## Gotchas

- Draft prompt creation does not affect the active prompt. Publishing changes live behavior.
- Publish accepts one exact prompt lifecycle `versionId`, not a history `promptId`.
- A prompt draft becomes stale when another prompt is published after its `parentVersionId`; publish rejects it rather than overwriting newer configuration.
- `Update prompts` is the direct-live alternative. It creates a new saved version rather than patching in place.
- Prompt writes require full `hcPlanningPrompt`, `hcGuidelinePrompt`, and `hcChannelPrompt` arrays.
- Do not fabricate a partial prompt payload from a summary or truncated preview. Fetch the complete prompt, apply the smallest intended diff, and submit the complete arrays.
- When an MCP prompt read reports `truncated: true`, it is not an exact review artifact. Retrieve the complete arrays and require `truncated: false` before publication.
- Current prompt write routes use the `/public` suffix; verify the live docs before calling them.
- Intent-rule creation defaults to published unless `status: DRAFT` is supplied; always send `DRAFT` for the review-first natural-language workflow. Business Rule creation is immediate and non-versioned.
- `List rules` defaults to intent rules when `type` is omitted; ask for or send `type=business` when Business Rules are intended.
- Intent rules are the only rules with draft/version/refine/restore/publish semantics. Filter by version status, refine the same rule in place, and publish the explicitly supplied exact reviewed `draftVersionId`.
- Never publish by omitting `draftVersionId` or resolving a current/latest draft after approval. If the ID is missing, list versions, fetch the candidate version completely, and obtain approval bound to that exact ID.
- Generating an intent-rule preview does not persist a rule. Persist the preview as `DRAFT` before claiming a draft exists.
- Do not duplicate a rule as a prerequisite for refinement. Duplication is only for an intentional independent copy.
- Treat every duplicate as live-impacting: re-fetch the exact source rule, verify its actual kind and assignments, disclose that the independent copy can be published or assigned, and require exact-source confirmation before calling Duplicate. Never infer the copy's state from the source kind; re-fetch the returned copy.
- Business Rules are immediate-live and non-versioned. Do not send them through intent draft, refine, restore, version-history, or publish routes. Business Rule create, update, duplicate, and delete operations require live-write treatment.
- If the fetched rule kind conflicts with the requested workflow or caller-supplied `type`, stop instead of coercing the resource through the other kind's route.
- Rule list summaries omit `flowConfig`; use Get rule before explaining or editing the tree.
- Draft intent rules return empty `botIds`; publishing requires non-empty target agent IDs.
- `isOutputTagGroup=true` makes a tag group unavailable in Rulebooks, including intent rules.
- An Action does nothing merely because it exists. Active runtime needs a published Rulebook assigned to the responding agent with a Tool node pointing at it; an eligible exact draft override of that existing assigned rule can also execute the Tool node.
- Action assignment records are stored but not enforced at runtime. Do not use them as proof of Action availability.
- A rule evaluation can reach Tool nodes and call external APIs. Use controlled inputs and do not label it read-only.
- Generated Rulebook tests are suggestions, not persisted cases or behavioral proof.
- A generated Rulebook test is not directly a Test Set row. Review it, establish an eligible persistent case/conversation, translate its assertions into Test Set criteria, then run the suite against current published live-agent behavior. Preflight reachable Actions.
- Tag update/delete path IDs are tag IDs, not tag group IDs.
- Some invalid tag/group IDs may return `500` or `null`; verify by listing known groups/tags first.

## Proof Of Completion

- Prompt draft: draft ID, `parentVersionId`, status, `truncated: false` or equivalent complete-read proof, full-array review, and exact diff are recorded; active behavior is unchanged.
- Published prompt: the exact reviewed draft ID is `PUBLISHED`, current prompts are re-fetched, changed sections are summarized, and the same runtime cases are rerun when requested.
- Stale prompt: the rejected version is not retried; stale draft, exact parent, and latest active versions were compared; conflicts were surfaced; and a newly based draft plus its exact review state are reported.
- Direct prompt update: the caller's immediate-live intent is explicit, the returned saved version is re-fetched, and the history/rollback path is identified.
- Rule draft: the fetched rule is confirmed as `intent`; persisted rule ID and draft version ID exist, status is `DRAFT`, no live assignment is assumed, and the next review/refine/publish step is clear.
- Published rule: the explicit exact reviewed `draftVersionId` and target `botIds` are confirmed, the rule is re-fetched, Action Tool-node IDs are verified when used, and behavior is tested if requested.
- Duplicated rule: exact source ID/kind and confirmation are recorded; the returned copy is re-fetched; its actual state, assignments, and independent ID are reported without assuming it is a draft.
- Business Rule: the resource is reported as immediate-live and non-versioned; no intent draft/version/publish claims are made.
- Rule test: tested rule/version, exact input context, node results, possible external effects, and remaining regression coverage are reported.
- Tags: custom group/tag IDs returned and re-listed; output-vs-rulebook availability is explicit.
- Onboarding/configuration: tags, rules, prompts, knowledge gaps, runtime test, and QA evidence are separated.

## References

- Read [prompts.md](references/prompts.md) for prompt draft review, exact publishing, stale-draft recovery, and direct updates.
- Read [rulebooks.md](references/rulebooks.md) for draft, refine, publish, Action Tool-node wiring, testing, versioning, and deletion.
- Read [tags.md](references/tags.md) for tag group design, output-vs-rulebook behavior, and custom-group mutation guardrails.
- Read [onboarding-sequence.md](references/onboarding-sequence.md) when configuring an existing Slack/support agent end to end.
