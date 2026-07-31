# Rule Testing

Use Rulebook testing to inspect required inputs, generate candidate cases, or execute one rule in isolation. Read the current [Get rule test fields](https://docs.usefini.com/en/api-reference/get-rule-test-fields), [Preview rule test fields](https://docs.usefini.com/en/api-reference/preview-rule-test-fields), [Evaluate rule](https://docs.usefini.com/en/api-reference/evaluate-rule), [Evaluate conversation rule](https://docs.usefini.com/en/api-reference/evaluate-conversation-rule), and [Generate Rulebook tests](https://docs.usefini.com/en/api-reference/generate-rulebook-tests) pages first.

## Choose The Operation

| Need | Operation |
| --- | --- |
| Discover inputs for a saved rule | Get rule test fields |
| Discover inputs for an unsaved `flowConfig` | Preview rule test fields |
| Draft candidate cases from a flow | Generate Rulebook tests |
| Evaluate a published saved rule with controlled synthetic data | Evaluate rule |
| Evaluate a published saved rule against stored real context | Evaluate conversation rule |

Field discovery and preview do not execute the rule. Generated tests are suggestions and are not proof, persistence, or a Test Set run.

## Evaluate A Rule

1. Fetch the exact published saved rule. Preserve its rule ID, status, `flowConfig`, assignments, and version identifier. These evaluation routes do not execute an unsaved flow or brand-new unassigned draft.
2. Get the saved rule's test fields. Construct only documented fields and values; use Preview separately for structural inspection of an unsaved flow without evaluation.
3. Define the expected matched path, outputs, non-matches, and edge conditions.
4. Inspect the flow for Tool/Action nodes. Rule evaluation does not send a customer reply, but do not assume external effects are suppressed. If an Action is reachable, state the possible call and require explicit evaluation intent.
5. Use synthetic evaluation when controlled input isolation matters. Use conversation evaluation when stored user attributes, messages, or runtime context are essential.
6. Record the returned node/path results and compare them with the expected path. Preserve unexpected branches and errors rather than summarizing only `matched`.
7. Re-fetch the rule if the test was part of a configuration workflow. Evaluation does not publish or update it.
8. Use Generate Answer for final one-case end-to-end conversational proof. When published behavior must hold across persistent cases, use a Test Set live run after preflighting every reachable Action or external effect.

## Generate Candidate Tests

Supply the exact rule description and `flowConfig`, plus the existing suite when deduplication matters. Review the returned suggestions for:

- positive and negative branches;
- boundary and missing-field cases;
- Action-bearing branches;
- duplicate or tautological expectations;
- fields unsupported by the current test-field response.

Do not claim generated suggestions were saved. After review, establish eligible persistent Test Set cases, translate the assertions into criteria, and run them against the current published live agent with controlled inputs and external effects.

## Verification

Report the rule ID/version, input source, exact evaluation context, expected path, actual node/path evidence, outputs, Action effects, and whether end-to-end runtime behavior was tested separately. A matching isolated rule result is not by itself proof that the full agent answer is correct.
