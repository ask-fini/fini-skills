---
name: fini-api-testing
description: Manage Test Suite cases, scenarios, criteria groups, Action mocks, and asynchronous published/draft regression runs; inspect Replay, Rulebook tests, and quality evidence. Use for saved-case setup, results, missing mocks, or baseline/candidate regression. Route ordinary message generation to fini-api-generate-answer and new-turn artifact comparison to fini-api-draft-validation.
---

# Fini API Testing

Choose the smallest Fini testing surface that can answer the question, preserve exact baseline and candidate evidence, and distinguish request-scoped unpublished candidates from persistent published/draft Test Suite regression.

Before endpoint-specific work, fetch `https://docs.usefini.com/llms.txt` and the current docs pages for the selected workflow. Treat the live index and linked pages as the route and schema source of truth.

## Choose The Test Surface

| Question | Use | Important effect |
| --- | --- | --- |
| "What does the agent answer to this new message?" | Generate Answer | Stores conversation events and runs normal live behavior |
| "How does this unpublished prompt/KB/rule candidate compare with current behavior?" | `fini-api-draft-validation` | Uses exact request-scoped artifact IDs; creates marked conversations/events; does not publish |
| "Why was this exact answer bad, what should change, and what would the improved answer be?" | `fini-api-refine-with-ai` | Diagnoses the event, creates reviewable draft recommendations, and returns its own replay evidence |
| "Would this known bad response differ with this exact prompt/rule/model candidate?" | Replay | Creates a separate replay conversation; default replay reuses original Rulebook and Attribute execution evidence unless supported artifacts change rule evaluation |
| "How does this Rulebook resolve for these fields?" | Rule test fields plus rule evaluation | Executes rule logic against synthetic or stored context |
| "Did quality or usage change over time?" | Agent analytics | Read-only aggregate observation; not causal proof |
| "Does this published or draft configuration pass saved regression cases?" | Test Suite | Evaluates saved cases with captured Attributes and simulated Action mocks |

Test Suite accepts exact run-level prompt versions, rule drafts and article drafts without publishing. Saved coverage artifacts are not execution overrides. Preserve saved inputs, Attributes, criteria and Action mocks across comparisons; a missing mock fails without calling the live Action.

Use Draft Validation for one new-turn artifact comparison; use Test Suite for saved-case published/draft regression. A go-live gate requires the intended published configuration; simulated results do not prove real integration health.

Use standalone Replay when the caller already selected the candidate or only wants regeneration. Do not create a duplicate Replay when Refine with AI already returned replay interaction/event IDs for the original bad response.

Use conversation ratings, feedback notes, and resolved state as quality labels and evidence. Route changes to those fields through `fini-api-conversations`; they are QA-state mutations, not behavioral tests.

## Workflow

1. Resolve the exact workspace agent, conversation, response event, rule, Test Suite cases/groups, and configuration versions needed for the request.
2. Classify the operation: read-only observation, new-turn draft validation, Replay generation, rule execution, or saved-case Test Suite regression.
3. Capture the baseline before changing or executing anything: inputs, IDs, expected behavior, stored answer or run, time window, filters, and relevant artifact versions.
4. Inspect real external effects for Generate Answer, artifact-backed Replay and rule evaluation. For Test Suite, verify saved Attribute context and recorded/custom Action mocks; missing mocks never fall back to live Actions.
5. Define success before running: expected answer facts, deterministic fields, judge criteria, blocking behavior, and acceptable error count.
6. For a new-turn candidate, use `fini-api-draft-validation`. For persistent cases, read the regression reference and select exact eligible Test Suite run-level prompt/rule/article versions. Preserve baseline setup and change one intended dimension at a time.
7. Save an eligible conversation capture belonging to this bot with source other than `replay`/`testsuite`, or author a scenario directly. Saving a scenario does not require creating a live conversation. The saved answer is source context, not the output a later run merely grades.
8. Re-fetch saved setup and exact run configuration. Preview selected cases and resolve incomplete criteria; preview does not validate draft eligibility or Action mocks. Read group impact and preserve the latest `impactToken` before replacing criteria or unlinking cases.
9. Start one requested run and poll the same ID until all case items are completed/failed, with none queued/running. Resume that ID after interruption; do not create a duplicate run to check progress.
10. Compare evidence, not only verdicts: generated answer, used knowledge, rule/Action evidence, reply type, per-criterion reasoning, external effects, and per-case errors.
11. Re-fetch the exact created or changed resource and report IDs, captured configuration, case-level totals, mock diagnostics and untested paths.
12. Delete only explicitly requested supported cases, folders or membership; preserve reconstruction evidence and verify absence. No public standalone criterion-update/delete, group-delete or run-delete route is exposed. Source-conversation deletion belongs to its owning skill.

## Test Design Defaults

- Reuse the same Test Suite cases, inputs, agent, criteria and mocks for published/draft comparisons.
- Prefer known production failures and representative edge cases over invented happy paths.
- A Test Suite run selects one `botId`; report deliberate cross-agent comparisons separately.
- Use eligible conversation captures or authored scenarios. Fetch capture inputs or Action fixtures before constructing mocks; do not invent node/tool identities.
- Read Test Suite fields context before creating deterministic criteria; do not invent field paths, operators, or quantifiers.
- Use deterministic criteria for directly observable fields, `basic_judge` for one bounded semantic judgment, and `complex_judge` for multi-part or conversation-level quality.
- Make a criterion blocking only when its failure should fail the release gate. Keep exploratory criteria non-blocking.
- Treat generated Rulebook tests as suggestions, not saved cases. Review them, save eligible captures/scenarios, and attach local/shared criteria before running Test Suite.
- Separate failed case execution from a completed case whose `result.verdict` is `fail`.
- Count errored conversations separately from criterion failures.
- Preserve source conversation IDs and criterion-level evidence so every aggregate can be traced back.

## Effects And Intent

- Proceed directly with pure reads such as analytics, list/get replay, list/get Test Suite cases/groups/runs, and field-context inspection.
- Treat saving cases/groups/folders, recovering mocks and starting Test Suite runs, recording feedback, and creating replay records as mutations.
- Treat Generate Answer, artifact-backed Replay creation and rule evaluation as live executions when their reachable configuration can invoke Actions.
- Test Suite creates persisted replay/results and consumes model execution with simulated Actions and captured Attributes. Its REST contract has no `confirmationPhrase`; current MCP gates still apply.
- Before any execution with possible external effects, name the exact agent/cases, run multiplicity, and effect class, then obtain explicit run intent. Once intent is clear, execute without adding artificial approval layers.
- Before deletion, show the exact supported resource type and IDs; never infer legacy cleanup behavior.
- Never expose stored Action credentials or resolved secret headers/bodies. Redact credentials and customer secrets from evidence and reports.
- Do not claim a cleanup route exists when current public docs do not expose one. The replay API currently documents no dedicated replay-deletion endpoint.

## Proof Of Completion

Report:

- test surface chosen and why;
- target IDs, captured published/draft configuration versions, persistent case inputs, and expected behavior;
- baseline/candidate artifact interaction IDs for Draft Validation, plus Test Suite run or Replay IDs where used;
- terminal status and summary totals;
- failed and errored cases with criterion reasoning and evidence;
- Test Suite recorded/custom/missing mock evidence; actual effects from separately requested Draft Validation, Replay, rule evaluation or integration tests;
- cleanup performed, preserved audit snapshot, and exact absence check;
- remaining uncertainty or follow-up owner.

## References

- Read [replays.md](references/replays.md) for single-conversation comparison, modes, event alignment, and replay verification.
- Read [test-suite-regressions.md](references/test-suite-regressions.md) for saved cases, criteria groups, draft selectors, mocks, polling, comparison and cleanup.
- Read [rule-testing.md](references/rule-testing.md) for rule field discovery, generated tests, synthetic evaluation, and conversation-grounded evaluation.

Use `fini-api-draft-validation` for request-scoped exact unpublished prompt, knowledge, snapshot, or eligible intent-rule comparisons before publishing.
