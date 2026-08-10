# Test Set Live Runs And Behavioral Regressions

Use Test Sets for persistent cases and criteria evaluated against current published live-agent behavior. A run may invoke Actions or external APIs. Use `fini-api-draft-validation` for unpublished prompt, knowledge, snapshot, or eligible intent-rule candidates because Test Set runs do not accept request-scoped draft artifact overrides. Read the current [Test Sets overview](https://docs.usefini.com/en/api-reference/test-sets), [Fields context](https://docs.usefini.com/en/api-reference/get-test-set-fields-context), [Start a run](https://docs.usefini.com/en/api-reference/start-test-set-run), and [Get a run](https://docs.usefini.com/en/api-reference/get-test-set-run) pages before execution.

## Build The Set

1. Select eligible source conversations as persistent case seeds representing common behavior, known failures, boundary cases, and previously recorded Action-bearing paths. Conversations with `source=replay` or `source=testsuite` are not eligible Test Set inputs. The run executes those cases against current published behavior; it does not merely grade the saved answers.
2. Verify each conversation's `botId`. Keep one agent per Test Set by default; if a cross-agent analysis is deliberate, stratify every result and never present the aggregate as one agent's regression.
3. Create or update a Test Set with the documented conversation limit. Re-fetch it and verify the resolved conversation list before running.
4. Fetch fields context. Use returned defaults, field paths, operators, and quantifiers rather than inventing them.
5. Add only criteria with a clear claim:
   - `deterministic`: directly observable state or execution evidence;
   - `basic_judge`: one bounded semantic judgment;
   - `complex_judge`: multi-part or conversation-level quality.
6. Make only release-critical criteria blocking. Keep diagnostic criteria non-blocking.
7. Re-fetch the Test Set and record criterion IDs, types, definitions, and blocking flags.

Avoid criteria that encode the candidate wording, combine unrelated claims, or cannot be traced to available evidence. Treat generated tests as suggestions, not accepted criteria.

## Run And Poll

Starting a run returns `202`, `status: running`, and `result: null`. The run evaluates the set's cases against the current published live agent. It can execute Rulebook Tool nodes, Actions, Attributes, and customer APIs.

1. Re-fetch the Test Set, source conversations/cases, criteria, target agent, and current published configuration.
2. Inspect every case for reachable Action Tool nodes, configured Attributes, external calls, and repeated-effect risk.
3. Use a controlled workspace/account and idempotent or otherwise acceptable targets. Explain the possible effect count and obtain explicit run intent.
4. Start one run and retain the run ID.
5. Poll Get run with bounded intervals until `completed` or `failed`. If polling is interrupted, return the same run ID as the resume point. Do not start another run.
6. On `completed`, inspect:
   - `result.summary.testSetResult`;
   - passed, failed, and errored conversation counts;
   - total, passed, and failed criteria;
   - every failed/error conversation;
   - criterion verdict, blocking flag, reasoning, and evidence;
   - generated behavior plus Action/Attribute/external-call evidence.
7. On `failed`, report the run error as an execution/run failure. Do not label it a behavioral regression without completed criterion results.

## Prove A Configuration Change

A Test Set run proves only current published live-agent behavior. It cannot apply an unpublished prompt, knowledge, snapshot, or rule artifact.

1. For pre-publish proof, use `fini-api-draft-validation`: run an artifact-free active baseline, isolated exact-ID candidates, and a combined candidate. Through MCP, set the connector's `markAsTest: true`; direct REST exposes no top-level field by that name.
2. Publish only the separately approved exact versions through their owning skills.
3. Re-fetch the live prompt, knowledge visibility, Rulebooks, Actions, and Attributes so the run's configuration is explicit.
4. Run the same stable Test Set after publication, with the same cases and criteria used for the baseline run.
5. Preflight Action and external effects for every run; both baseline and post-publish runs can invoke them.
6. Compare case-aligned criterion and execution evidence, not only aggregate percentages.
7. Record published configuration versions, run timestamps, external state assumptions, and observed effects. If the live configuration or controlled target cannot be established, label attribution inconclusive.

## Compare Runs

Compare run IDs only after verifying the same Test Set version, logical cases, agent ID, inputs, criteria, external-state assumptions, and controlled targets. Record the exact published configuration and run timestamps. A changed case, criterion, agent, or external record can make the comparison inconclusive.

Report:

```text
Baseline run ID / Candidate run ID
Test Set ID and conversation count
Criteria IDs and blocking state
Pass, fail, and error deltas
New failures / fixed failures / unchanged failures
Criterion reasoning and evidence for every delta
Published configuration per run
Action/Attribute/external effects per live run
```

If inputs, criteria, or configuration changed in uncontrolled ways, label the result inconclusive instead of attributing the delta.

## Update And Cleanup

- Before updating conversation membership or criteria, read the exact current Test Set and preserve its IDs for the report.
- Before deleting a criterion, preserve its full definition and linked run evidence, then show the Test Set ID and criterion ID and require explicit deletion intent.
- Before deleting a Test Set, preserve its metadata, conversation IDs, criterion definitions, run IDs, and retained results. Verify that no run is active; the API rejects deletion while a run is active.
- After criterion deletion, re-fetch the Test Set and verify that the criterion ID is absent.
- After Test Set deletion, use list/get to verify absence; distinguish expected not-found evidence from transport failure.
- Never delete source conversations as implicit Test Set cleanup. Conversation deletion is a separate, explicitly requested workflow.
