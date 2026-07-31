---
name: fini-api-testing
description: Use when the user wants to test, evaluate, compare, or monitor Fini agent behavior through public APIs; replay known responses with exact candidate artifacts; inspect or evaluate Rulebooks; run persistent Test Set cases and criteria against current published live-agent behavior; start, poll, compare, or delete Test Set runs/resources; inspect agent analytics; or build evidence-based quality, release-gate, regression, and improvement workflows. Route unpublished active-baseline versus draft-artifact comparisons to `fini-api-draft-validation`.
---

# Fini API Testing

Choose the smallest Fini testing surface that can answer the question, preserve exact baseline and candidate evidence, and distinguish request-scoped unpublished candidates from persistent live-agent regression.

Before endpoint-specific work, fetch `https://docs.usefini.com/llms.txt` and the current docs pages for the selected workflow. Treat the live index and linked pages as the route and schema source of truth.

## Choose The Test Surface

| Question | Use | Important effect |
| --- | --- | --- |
| "What does the agent answer to this new message?" | Generate Answer | Stores conversation events and runs normal live behavior |
| "How does this unpublished prompt/KB/rule candidate compare with current behavior?" | `fini-api-draft-validation` | Uses exact request-scoped artifact IDs; creates marked conversations/events; does not publish |
| "Would this known bad response differ with this exact prompt/rule/model candidate?" | Replay | Creates a separate replay conversation; default replay reuses original Rulebook and Attribute execution evidence unless supported artifacts change rule evaluation |
| "How does this Rulebook resolve for these fields?" | Rule test fields plus rule evaluation | Executes rule logic against synthetic or stored context |
| "Did quality or usage change over time?" | Agent analytics | Read-only aggregate observation; not causal proof |
| "Does the current published agent pass this persistent regression suite?" | Test Set | Asynchronously evaluates the set's cases against live agent behavior and may invoke Actions/external APIs |

Test Sets do not accept the request-scoped unpublished artifact overrides used by Draft Validation. Run the same stable Test Set before and after a separately approved publish when release comparison is needed. Preflight every live run for reachable Actions and customer-API effects.

Use `fini-api-draft-validation` to decide whether unpublished candidates are ready. Use a Test Set after publication to prove that current live-agent behavior passes persistent cases and criteria.

Use conversation ratings, feedback notes, and resolved state as quality labels and evidence. Route changes to those fields through `fini-api-conversations`; they are QA-state mutations, not behavioral tests.

## Workflow

1. Resolve the exact workspace agent, conversation, response event, rule, Test Set, and configuration versions needed for the request.
2. Classify the operation: read-only observation, draft-candidate execution, Replay generation, rule execution, or live Test Set regression.
3. Capture the baseline before changing or executing anything: inputs, IDs, expected behavior, stored answer or run, time window, filters, and relevant artifact versions.
4. Inspect whether Generate Answer, an artifact-backed Replay, rule evaluation, or a Test Set run can reach an Action or external API. State likely external effects and require explicit run intent for controlled targets.
5. Define success before running: expected answer facts, deterministic fields, judge criteria, blocking behavior, and acceptable error count.
6. For an unpublished behavioral candidate, route to `fini-api-draft-validation`: establish an artifact-free baseline, persist/fetch exact prompt, knowledge, snapshot, and eligible intent-rule version IDs, test one dimension at a time, then combine. Do not use a Test Set to apply draft overrides.
7. Before creating a Test Set, verify every persistent case seed is an eligible source conversation: its `source` must be neither `replay` nor `testsuite`. Verify `botId` per case and keep sets single-agent unless a cross-agent comparison is deliberate and separately reported. The saved answer is source context, not the output a later live run merely grades.
8. Before starting a Test Set run, re-fetch the live agent/configuration, persistent cases, criteria, and reachable Action paths. Use controlled accounts/inputs and make repeated effects acceptable.
9. Start one run and poll that run ID until `completed` or `failed`. Do not create a duplicate run merely because polling took longer than expected.
10. Compare evidence, not only verdicts: generated answer, used knowledge, rule/Action evidence, reply type, per-criterion reasoning, external effects, and per-case errors.
11. Re-fetch the exact created or changed resource and report IDs, timestamps, terminal status, live effects, and any untested paths.
12. Delete criteria or Test Sets here only when cleanup is explicitly requested. Before deletion, preserve a reconstruction/audit snapshot containing set metadata, conversation IDs, criterion definitions, run IDs, and retained results. Resolve exact IDs, wait for active runs to finish, execute only the requested deletion, and verify absence by list/get. Route conversation or other resource deletion to its owning skill; never include it implicitly.

## Test Design Defaults

- Reuse the same Test Set cases, inputs, agent, and criteria before and after a published change.
- Prefer known production failures and representative edge cases over invented happy paths.
- Keep each Test Set single-agent by default. If a deliberate cross-agent set is necessary, stratify and report every result by `botId`.
- Do not add conversations with `source=replay` or `source=testsuite` to a Test Set. When an eligible persistent case must be seeded, create a controlled `markAsTest: true` conversation through Generate Answer and treat its input as the reusable case; the later Test Set run generates current live-agent behavior rather than grading that stored answer.
- Read Test Set fields context before creating deterministic criteria; do not invent field paths, operators, or quantifiers.
- Use deterministic criteria for directly observable fields, `basic_judge` for one bounded semantic judgment, and `complex_judge` for multi-part or conversation-level quality.
- Make a criterion blocking only when its failure should fail the release gate. Keep exploratory criteria non-blocking.
- Treat generated Rulebook tests as suggestions, not Test Set rows. Review and deduplicate them, establish eligible persistent cases, and translate their assertions into criteria before running the live suite.
- Separate `status: failed` (the run failed) from a completed run whose `testSetResult` is `fail`.
- Count errored conversations separately from criterion failures.
- Preserve source conversation IDs and criterion-level evidence so every aggregate can be traced back.

## Effects And Intent

- Proceed directly with pure reads such as analytics, list/get replay, list/get Test Set/run, and field-context inspection.
- Treat creating or updating Test Sets and criteria, recording feedback, and creating replay records as mutations.
- Treat Generate Answer, artifact-backed Replay creation, rule evaluation, and Test Set runs as live executions when their reachable configuration can invoke Actions.
- Treat a Test Set run as asynchronous live-agent evaluation that creates a run record, evaluates its persistent cases against current published configuration, and may call customer Actions or external APIs.
- Before any execution with possible external effects, name the exact agent/cases, run multiplicity, and effect class, then obtain explicit run intent. Once intent is clear, execute without adding artificial approval layers.
- Before deletion, show the exact resource type and IDs. Test Set deletion is blocked while a run is active.
- Never expose stored Action credentials or resolved secret headers/bodies. Redact credentials and customer secrets from evidence and reports.
- Do not claim a cleanup route exists when current public docs do not expose one. The replay API currently documents no dedicated replay-deletion endpoint.

## Proof Of Completion

Report:

- test surface chosen and why;
- target IDs, published configuration versions, persistent case inputs, and expected behavior;
- baseline/candidate artifact interaction IDs for Draft Validation, plus live Test Set run or Replay IDs where used;
- terminal status and summary totals;
- failed and errored cases with criterion reasoning and evidence;
- any Actions or external effects invoked during Draft Validation, Replay, rule evaluation, or live Test Set runs;
- cleanup performed, preserved audit snapshot, and exact absence check;
- remaining uncertainty or follow-up owner.

## References

- Read [replays.md](references/replays.md) for single-conversation comparison, modes, event alignment, and replay verification.
- Read [test-suite-regressions.md](references/test-suite-regressions.md) for Test Set design, criteria, asynchronous polling, comparison, and cleanup.
- Read [rule-testing.md](references/rule-testing.md) for rule field discovery, generated tests, synthetic evaluation, and conversation-grounded evaluation.

Use `fini-api-draft-validation` for request-scoped exact unpublished prompt, knowledge, snapshot, or eligible intent-rule comparisons before publishing.
