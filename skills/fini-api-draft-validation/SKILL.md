---
name: fini-api-draft-validation
description: Use when the user wants to validate unpublished Fini prompt, knowledge-article, knowledge-snapshot, or eligible intent-rule candidates through `fini_generate_answer`; compare current behavior with one or more exact draft artifact IDs; test each candidate dimension before a combined configuration; or prove a review draft without publishing it. Route AI diagnosis and draft generation for one exact bad response to `fini-api-refine-with-ai`, ordinary turns to `fini-api-generate-answer`, caller-selected historical candidates to Replay, rule-only logic checks to rule evaluation, and persistent live-agent regression to `fini-api-testing`.
---

# Fini API Draft Validation

Use this skill to produce a controlled, request-scoped behavioral comparison before publication. Persist candidates through their owning workflows, pass only their exact IDs through `fini_generate_answer.artifacts`, and preserve active configuration.

Before endpoint-specific work, fetch `https://docs.usefini.com/llms.txt` plus the current Generate Answer and owning prompt, knowledge, or rule docs.

## Choose The Surface

| Need | Use |
| --- | --- |
| Compare active behavior with exact unpublished candidates on a new controlled turn | This skill |
| Diagnose one bad historical response and have Fini propose drafts plus a replay | `fini-api-refine-with-ai` |
| Send or continue one ordinary turn with already selected configuration | `fini-api-generate-answer` |
| Regenerate a known historical response with its prior context | Replay through `fini-api-testing` |
| Inspect only a Rulebook path for supplied fields | Rule test fields and rule evaluation |
| Run durable regression cases against current published live-agent behavior | Test Sets through `fini-api-testing` |

A Test Set run evaluates its persistent cases against current live-agent behavior and may invoke Actions or external APIs. It does not accept the request-scoped unpublished artifact overrides used here. Validate drafts with Generate Answer artifacts; use Test Sets after publication for persistent live regression.

## Candidate Workflow

1. Resolve the exact workspace and `botId`.
2. Fix the test case before execution: user message, role, channel, metadata, user attributes, prior-context shape, expected result, and controlled customer/test identity.
3. Inspect configured Attributes and reachable Action Tool nodes. `markAsTest: true` labels synthetic traffic; it does not suppress integrations, prevent external effects, or make Generate Answer read-only.
4. Run an active baseline in a fresh conversation with `role: user`, `markAsTest: true`, and no `artifacts`.
5. Persist every candidate through its owning skill and fetch the exact stored object:
   - prompt draft/version through `fini-api-agent-configuration`;
   - knowledge drafts or an exact knowledge snapshot through `fini-api-knowledge`;
   - intent-rule draft/version through `fini-api-agent-configuration`.
6. Reject missing, inferred, truncated, cross-workspace, or non-persisted IDs. Do not put raw prompt arrays, article text, instructions, or rule flow JSON inside Generate Answer artifacts.
7. Run one fresh, like-for-like candidate conversation per changed dimension:
   - prompt only with `artifacts.promptVersionId`;
   - knowledge drafts only with `artifacts.knowledgeArticleDraftIds`;
   - exact snapshot only with `artifacts.knowledgeSnapshotId`;
   - eligible intent-rule versions only with `artifacts.ruleVersionIds`.
8. Compare each result with the active baseline before combining candidates.
9. If isolated candidates behave as intended, run one fresh combined candidate with the explicit reviewed ID set.
10. Capture interaction and event IDs, answer, evidence/used knowledge, rule path, Attribute/Action effects, errors, and the exact artifact set for every run.
11. Stop at a review result. Artifact overrides are request-scoped and imply no publish, assignment, or persistent configuration change.

For time-sensitive Attributes or customer state, refresh the baseline or report that the comparison is not fully controlled. For a multi-turn case, build equivalent independent baseline and candidate threads rather than continuing the baseline under a different configuration.

## Artifact Gates

- **Prompt:** create a draft, fetch that exact lifecycle version, require a complete non-truncated payload, then pass only its `versionId` as `artifacts.promptVersionId`.
- **Knowledge draft:** create or generate with draft intent, poll the job to a terminal state, require concrete draft/article IDs, fetch each exact draft, then pass the IDs as `artifacts.knowledgeArticleDraftIds`. `COMPLETED` with no persisted ID is a blocker.
- **Knowledge snapshot:** use `artifacts.knowledgeSnapshotId` only when an exact workspace-owned snapshot ID is available and verified. Never substitute “latest” or infer one from article state.
- **Intent rule:** verify the fetched rule is `intent`, persist or refine its draft, fetch the exact version ID, and pass it through `artifacts.ruleVersionIds` only when the owning rule already has a published version assigned to the target bot. A brand-new unassigned draft and a Business Rule have no supported rule-version override path.

Read [candidate-validation-playbook.md](references/candidate-validation-playbook.md) for the isolation ladder, acquisition proof, comparison record, and routing boundaries.

## Safety And Interpretation

- Generate Answer requires write scope and creates conversation/events even when nothing is published.
- Use a controlled workspace, agent, account, and inputs when configured or overridden Rulebooks can reach Actions.
- Treat configured Attributes as possible external calls and sources of changing context.
- Do not retry blindly; another send can duplicate events and external effects.
- Keep secrets and sensitive external responses out of comparison records.
- Do not claim a passing candidate is active. Prove active behavior separately after an explicitly requested publish.

## Proof Of Completion

Report:

- exact workspace, agent, case, and expected behavior;
- active-baseline interaction/event IDs and artifact-free configuration;
- each isolated candidate's exact persisted artifact IDs and interaction/event IDs;
- combined candidate IDs and result, when run;
- like-for-like controls and any dynamic-context differences;
- observed or uninspected Attribute/Action effects;
- pass/fail reasoning and remaining uncertainty;
- explicit statement that no publish or active configuration change was implied.
