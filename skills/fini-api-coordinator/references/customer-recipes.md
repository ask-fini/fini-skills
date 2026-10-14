# Customer Recipes

Use this reference when the user describes a customer job rather than an endpoint. Keep route schemas in the live docs; use these recipes for selection, sequencing, and proof.

## Contents

- [Create And Configure A Support Agent](#create-and-configure-a-support-agent)
- [Full Slack-Led Client Onboarding](#full-slack-led-client-onboarding)
- [Add An Action Backed By A Customer API](#add-an-action-backed-by-a-customer-api)
- [Add An Attribute](#add-an-attribute)
- [Train The Bot From Customer Docs](#train-the-bot-from-customer-docs)
- [Refresh Changed Docs](#refresh-changed-docs)
- [Diagnose And Fix A Bad Answer](#diagnose-and-fix-a-bad-answer)
- [Prove Quality With A Test Suite](#prove-quality-with-a-test-suite)
- [Use Caller Metadata Or Configured Attributes](#use-caller-metadata-or-configured-attributes)
- [Make Knowledge Visible To One Agent](#make-knowledge-visible-to-one-agent)

## Create And Configure A Support Agent

Customer phrases:

- "Create a support bot."
- "Set up a new billing agent."
- "Configure this agent's prompts and Rulebooks."

Workflow:

1. Inspect the existing agent list and resolve duplicate names.
2. Use `fini-api-agents` to create a bare agent only when the requested agent does not already exist.
3. Make the boundary explicit: creation does not install Slack, authorize providers, create knowledge, configure prompts, or wire integrations.
4. Use `fini-api-agent-configuration` for prompt drafts, tags, and intent Rulebook drafts; keep Business Rules on their immediate-live, non-versioned lifecycle.
5. Use Sources/Knowledge when the agent needs customer content.
6. Use Actions/Attributes only when the use case needs live context or customer-system operations.
7. Validate unpublished candidates through `fini-api-draft-validation` when pre-publish behavioral proof is required.
8. After approved versions are published, use marked Generate Answer for focused active smoke.
9. Use `fini-api-testing` for a persistent Test Suite go-live gate and verify saved inputs, effective criteria, and Action mocks.

Proof:

- Exact agent name and ID.
- Existing versus newly created status.
- Configuration, knowledge, integration, and channel-installation status kept separate.
- Terminal Test Suite run and unresolved blockers.

## Full Slack-Led Client Onboarding

Customer phrases:

- "Onboard us to Fini from Slack."
- "Set up our Fini bot and KB."
- "Build our support bot behavior, integrations, knowledge, and tests."

Workflow:

1. `fini-api-slack-onboarding`: inspect workspace state and classify fresh onboarding, expansion, or targeted update.
2. Gather scope, boundaries, escalation, voice, content sources, automation needs, required customer context, examples, and launch constraints.
3. Resolve the target agent or create a bare one.
4. Confirm user-owned Slack installation, provider OAuth, and other prerequisites not covered by public routes.
5. Verify a usable folder tree and create or initialize one when absent before source-backed knowledge generation.
6. Use Sources and Knowledge for ingestion, draft generation, review, publish, organization, and assignment.
7. Use Agent Configuration for prompt drafts, tags, and Rulebooks.
8. Add optional Attributes and Actions when requirements call for them.
9. Validate review drafts through `fini-api-draft-validation` before publishing when pre-launch candidate proof is required.
10. After publication, use marked Generate Answer for focused active smoke.
11. Run the persistent Test Suite under the intended published configuration, inspect the same run until its case items are terminal, and inspect blocking failures and mock diagnostics before declaring go-live readiness.

Proof:

- Onboarding classification and target agent.
- Each user-owned prerequisite marked ready or blocked.
- Persisted source, knowledge, configuration, and integration IDs.
- Test Suite/run IDs, pass/fail summary, and go-live blockers.

## Add An Action Backed By A Customer API

Customer phrases:

- "Let the bot refund an order."
- "Allow customers to cancel subscriptions."
- "Update an account through our API."

Workflow:

1. Confirm the customer-owned endpoint, operation, auth source, required identifiers, business permissions, idempotency behavior, and audit expectations.
2. Use `fini-api-actions-attributes` to create a typed Action and its ordered external API steps.
3. Map only the fields required by later steps and the final Action output.
4. Keep stored secret values masked and out of reports.
5. Test only when the supplied inputs and target make the real external effect acceptable.
6. Use `fini-api-agent-configuration` to add a Tool node to the intended intent Rulebook draft and record its exact reviewed `draftVersionId`.
7. Publish the explicitly reviewed exact Rulebook `draftVersionId`; never omit it or resolve a current/latest draft after approval.
8. Use marked Generate Answer with acceptable targets for a focused active smoke. Use Replay separately for historical diagnosis.
9. Run persistent success, denial, failure, and escalation Test Suite cases with saved Action success/failure mocks. Verify real HTTP behavior separately through an explicitly requested integration test.

Proof:

- Action and external-call IDs.
- Input/output contracts and step order.
- Exact Rulebook/Tool-node reachability.
- Test outcome without raw headers, secrets, or sensitive external bodies.

Remember: Action assignment metadata alone does not make the Action runnable. Published Rulebook reachability does.

## Add An Attribute

Customer phrases:

- "Fetch the user's plan before every answer."
- "Give the bot order or account context automatically."
- "Expose this API field to our rules."

Workflow:

1. Identify the minimum required fields for eligible messages and where each should be visible: model, Rulebook, or downstream call.
2. Create the Attribute and ordered external calls.
3. Verify response mappings and masking.
4. Run one controlled lookup.
5. Assign the Attribute to the exact agent/channel.
6. Use marked Generate Answer to verify one focused eligible-message behavior, latency, and missing-data case.
7. Run persistent Attribute cases through Test Suite using captured Attribute outputs or explicit overrides. Test real provider lookups separately.

Proof:

- Attribute and step IDs.
- Exact assignment.
- Expected fields present without unnecessary customer data.
- Runtime and regression results.

Attribute assignment changes eligible-message behavior. Treat it differently from Action assignment.

## Train The Bot From Customer Docs

Customer phrases:

- "Train the bot on our help center."
- "Import our internal docs."
- "Sync Notion, Zendesk, or Confluence."

Workflow:

1. Verify a usable knowledge tree and create or initialize one when absent before source-backed generation.
2. `fini-api-sources`: discover/register/ingest or crawl/ingest the raw sources.
3. Poll source status; queued ingestion is not completion.
4. `fini-api-knowledge`: bulk-generate draft knowledge from processed source IDs.
5. Poll jobs and require persisted article/draft IDs.
6. Review, publish, organize, and assign to the intended agent.
7. Use marked Generate Answer for a focused active retrieval smoke, then run the go-live Test Suite against current published behavior.

Proof:

- Processed source IDs.
- Persisted article or draft IDs.
- Published state and agent-scoped visibility.
- Test Suite results for source-specific questions.

## Refresh Changed Docs

Customer phrases:

- "Our docs changed."
- "Sync only changed sources."
- "Update the KB without losing the live version."

Workflow:

1. Refresh existing source IDs and poll completion.
2. Select changed sources linked to knowledge.
3. Generate updates as drafts.
4. Review against the current article and its history.
5. Publish and verify assignment.
6. Rerun the same persistent Test Suite against current published behavior and compare with its prior run.

Proof:

- Changed source IDs.
- Previous and new article versions.
- Published article IDs.
- Regression comparison.

## Diagnose And Fix A Bad Answer

Customer phrases:

- "Why was this answer wrong? Propose the fix and replay it."
- "Use Refine with AI and show the improved replay."
- "Fix this and prove nothing else regressed."

Workflow:

1. Use `fini-api-conversations` to preserve the exact conversation and event evidence.
2. If the user asked only for a read-only explanation, stop with Conversations evidence. When they want AI diagnosis plus a proposed correction, use `fini-api-refine-with-ai`; read an active session first and do not duplicate its drafts or replay.
3. Review the returned root cause, old/replayed answers, and every exact article/prompt/rule draft ID. Verify factual claims against trusted evidence.
4. Route each exact draft to Knowledge or Agent Configuration for its normal review and separate publication approval.
5. Use `fini-api-draft-validation` only for additional clean cases or combined-candidate proof.
6. Publish only the exact approved artifact, then rerun the persistent Test Suite under the intended published configuration and compare with baseline.
7. Update feedback/resolution state only after the live evidence supports it.

Proof:

- Original event and root-cause evidence.
- Refine session/iteration, exact generated artifact IDs, and old/replayed answers.
- Factual review plus exact separately published artifact/version, if approved.
- Test Suite regression result.

## Prove Quality With A Test Suite

Customer phrases:

- "Run our golden set."
- "Set up a regression gate."
- "Check the release against past conversations."

Workflow:

1. Save eligible conversation captures or authored scenarios with stable inputs and expected outcomes.
2. Attach case criteria or reusable criteria groups; organize cases/groups in folders when useful.
3. Resolve published baseline or exact draft configuration and preserve saved Attribute context and Action mocks.
4. Preview the exact selection and address cases without effective criteria.
5. Start one requested run and inspect the same ID until all case items are terminal.
6. Compare criterion evidence, replies, errors, mock diagnostics, and sameSetup across runs.
7. Test candidate versions without publishing. Publish only when separately requested; a go-live gate must evaluate the intended published configuration.

Proof:

- Case, criteria-group, criterion, and run IDs.
- Terminal status, totals, and blocking failures.
- Baseline-versus-candidate comparison.
- Clear go-live decision.

Test Suite runs capture published or selected draft configuration and simulate Actions. Use conversation captures or authored scenarios as cases; standalone Replay output is not an eligible captured-conversation source.

## Use Caller Metadata Or Configured Attributes

- Use Generate Answer `metadata.user_attributes` when the caller already knows and supplies the context for one turn.
- Use a configured Attribute when Fini should fetch context automatically for eligible messages.
- Do not invent CRM, billing, or helpdesk fields.
- Send or expose only the minimum customer data required for the workflow.
- Keep keys stable across tests so regression evidence is comparable.

## Make Knowledge Visible To One Agent

1. Resolve the exact agent.
2. Fetch the knowledge tree.
3. Confirm the article lives in the intended folder.
4. Add or remove the folder-agent assignment.
5. Re-fetch the agent-scoped tree.
6. Use marked Generate Answer for one focused active retrieval smoke, then run the relevant persistent Test Suite criterion against captured published or selected draft configuration when broader proof is needed.
