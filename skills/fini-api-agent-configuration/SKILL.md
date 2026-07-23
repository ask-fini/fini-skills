---
name: fini-api-agent-configuration
description: Use when the user wants to configure existing Fini agents through public APIs; inspect or update prompts and prompt history; create, list, duplicate, refine, publish, update, or delete rulebooks/rules/rule drafts; create, list, update, or delete tags and tag groups; design routing/output classification for a Slack or support agent; or plan safe agent behavior changes before testing with Generate Answer.
---

# Fini API Agent Configuration

Use this skill for existing-agent behavior configuration: prompts, rulebooks/rules, and tags/tag groups. It is not for creating a new Fini workspace, installing Slack, or provisioning a new agent unless the current public docs expose those routes.

Before endpoint-specific work, fetch `https://docs.usefini.com/llms.txt` and the relevant Prompts, Rules, Tags, or Tag Groups docs page.

## Choose The Layer

| User intent | Path |
| --- | --- |
| "Show/update the bot prompt" | Prompt read/history/update workflow |
| "Change how the agent behaves globally" | Prompt workflow, then Generate Answer test |
| "Test answers, diagnose failures, and change the guidelines" | Baseline Generate Answer tests -> prompt diff -> update -> re-fetch -> repeat the same tests |
| "Create a rulebook/rule from instructions" | Generate preview -> persist intent draft |
| "Refine this rulebook before launch" | Refine the same intent rule as a new draft version |
| "Publish this rulebook to one bot" | Publish the exact reviewed draft version with explicit agent IDs |
| "Inspect or restore an older rule version" | Rule version history -> restore as draft |
| "Create routing or outcome tags" | Tag group and tag workflow |
| "Configure a Slack/support agent" | Tags -> rule drafts -> prompts -> tests, then Sources/Knowledge if content is missing |

## Workflow Gates

1. Resolve the target agent with List agents when the user gives a name.
2. Read current state before writing: prompts/history, rule list/get, tag groups/tags.
3. Produce a short plan before live-impacting writes.
4. Prefer draft or versioned paths. For intent rules, generate a preview, persist it as `DRAFT`, refine the same rule, and publish the exact reviewed draft version. For prompt changes, preserve and submit all three arrays even when only one section changes.
5. Write only to workspace-owned custom tag groups/tags; treat Fini-managed defaults as read-only.
6. Re-fetch after writes and report changed IDs, draft/live state, assigned `botIds`, and remaining review steps.
7. Test with Generate Answer when the user expects behavior change.
8. Inspect conversations when the user needs QA, rollout, or golden-set evidence.

## Defaults

- Prefer intent-rule preview -> persisted `DRAFT` for natural-language rulebook requests.
- Read rule field context before authoring a flow tree. Use default Business Rule templates when the request matches one.
- Prefer prompt read/history before prompt update; preserve full prompt section arrays.
- For prompt improvement work, run the same small test set before and after the change so the behavioral effect is attributable.
- Prefer custom tag groups for customer-specific classification.
- Leave tag groups available to Rulebooks unless the user explicitly wants output-only reporting tags.
- Treat prompt updates, published rule create/update/delete, rule publish, tag deletion, and tag-group deletion as live-impacting.
- Do not claim full onboarding automation beyond existing-agent configuration unless current docs show the needed public routes.

## Gotchas

- `Update prompts` creates a new saved version, not an in-place patch.
- Prompt writes require full `hcPlanningPrompt`, `hcGuidelinePrompt`, and `hcChannelPrompt` arrays.
- Do not fabricate a partial prompt payload from a summary or truncated preview. Fetch the complete prompt, apply the smallest intended diff, and submit the complete arrays.
- The prompt write route currently does not use the `/public` suffix even though read routes do; verify the docs before calling it.
- `Create rule` creates a published rule directly; do not use it as the default for natural-language setup.
- `List rules` defaults to intent rules when `type` is omitted; ask for or send `type=business` when Business Rules are intended.
- Intent rules are versioned. Filter by version status, refine the same rule in place, and publish the exact reviewed `draftVersionId`.
- Generating an intent-rule preview does not persist a rule. Persist the preview as `DRAFT` before claiming a draft exists.
- Do not duplicate a rule as a prerequisite for refinement. Duplication is only for an intentional independent copy.
- Business Rules are not versioned and do not use the intent draft/publish lifecycle.
- Rule list summaries omit `flowConfig`; use Get rule before explaining or editing the tree.
- Draft intent rules return empty `botIds`; publishing requires non-empty target agent IDs.
- `isOutputTagGroup=true` makes a tag group unavailable in Rulebooks, including intent rules.
- Tag update/delete path IDs are tag IDs, not tag group IDs.
- Some invalid tag/group IDs may return `500` or `null`; verify by listing known groups/tags first.

## Proof Of Completion

- Prompts: prompt version saved, prompt re-fetched, changed sections summarized, rollback/history path identified, and the same runtime cases rerun when behavior validation was requested.
- Rule draft: persisted rule ID and draft version ID exist, status is `DRAFT`, no live assignment is assumed, and the next review/refine/publish step is clear.
- Published rule: exact reviewed draft version and target `botIds` are confirmed, the rule is re-fetched, and behavior is tested if requested.
- Tags: custom group/tag IDs returned and re-listed; output-vs-rulebook availability is explicit.
- Onboarding/configuration: tags, rules, prompts, knowledge gaps, runtime test, and QA evidence are separated.

## References

- Read [prompts.md](references/prompts.md) for prompt read/history/update sequencing and approval rules.
- Read [rulebooks.md](references/rulebooks.md) for draft, refine, publish, duplicate, update, and delete rule workflows.
- Read [tags.md](references/tags.md) for tag group design, output-vs-rulebook behavior, and custom-group mutation guardrails.
- Read [onboarding-sequence.md](references/onboarding-sequence.md) when configuring an existing Slack/support agent end to end.
