# Customer Recipes

Use this reference when the user describes a customer job rather than an endpoint name. Keep endpoint details in the live docs; use this file to choose the workflow and proof.

## Configure Or Onboard An Existing Agent

Customer phrases:

- "Set up this Slack agent."
- "Configure the bot behavior."
- "Add rulebooks and tags."
- "Update the prompt for this agent."

Workflow:

1. `fini-api-agent-configuration`: resolve the target agent.
2. Design routing or output tag groups before rules that depend on them.
3. Create/refine rule drafts from behavior instructions; publish only after review and explicit target agents.
4. Read prompt history/current prompts; update a new prompt version only after approval.
5. Use `fini-api-sources` and `fini-api-knowledge` if the behavior gap is missing content rather than configuration.
6. Use `fini-api-generate-answer` for targeted runtime tests.
7. Use `fini-api-conversations` for rollout QA or golden-set evidence.

Proof:

- Agent ID resolved.
- Tags/tag groups listed or created and Rulebook availability is clear.
- Rules are draft or published with target `botIds` explicit.
- Prompt version status is explicit.
- Knowledge changes are separated from configuration changes.
- Runtime tests or conversation evidence show whether behavior changed.

Boundary:

- This configures an existing Fini agent. Do not promise full workspace, Slack install, or new-agent provisioning unless the current public docs expose those routes.

## Full Slack-Led Client Onboarding

Customer phrases:

- "Onboard us to Fini."
- "Set up my Fini bot and KB."
- "We connected Zendesk; help us get the bot ready from Slack."
- "Build our support bot behavior, KB, and tests."

Workflow:

1. `fini-api-slack-onboarding`: verify workspace state first: agents, articles/drafts, folders, and sources.
2. If fresh onboarding, offer all-at-once intake or step-by-step intake.
3. Gather scope, boundaries, escalation, tone, sources, top questions, examples, and launch constraints.
4. Use `fini-api-sources` for source discovery, crawl, register, ingest, refresh, and polling.
5. Use `fini-api-knowledge` for draft knowledge, folder/tree setup, publish-after-review, and assignment.
6. Use `fini-api-agent-configuration` for prompts, rule drafts, rule publish after approval, tags, and tag groups.
7. Use `fini-api-generate-answer` for a 4-8 question onboarding test suite.
8. Produce a readiness summary with blockers and approvals.

Proof:

- Fresh/expansion/targeted classification is explicit.
- Target agent is resolved.
- Client inputs are collected or marked missing.
- Sources, KB, folders, config, and tests each have a status.
- Live-impacting operations have explicit approvals.

## Train The Bot From Customer Docs

Customer phrases:

- "Train the bot on our help center."
- "Import these internal docs."
- "Use our website pages as knowledge."
- "Sync Notion, Zendesk, or Confluence."

Workflow:

1. `fini-api-sources`: discover/register/ingest or crawl/ingest the raw sources.
2. Poll source status; do not treat queued ingestion as completion.
3. `fini-api-knowledge`: bulk-generate draft knowledge from processed sources.
4. Review/publish after approval.
5. Assign the containing folder to the intended agent.
6. `fini-api-generate-answer`: run a targeted behavior test if proof is needed.

Proof:

- Source records processed.
- Drafts or articles created.
- Published state is explicit.
- Folder assignment is visible in the scoped agent tree.
- Test answer uses the intended knowledge when runtime proof is requested.

## Refresh Changed Docs Safely

Customer phrases:

- "Our docs changed."
- "Sync only changed articles."
- "Update the KB without breaking live answers."

Workflow:

1. Refresh existing source IDs.
2. Poll source completion.
3. List changed sources and keep the ones linked to knowledge.
4. Bulk-generate drafts by default.
5. Review, publish, and verify assignment.
6. Test the intended agent if the customer expects answer behavior to change.

Proof:

- Changed source IDs.
- Draft/live result.
- Published article IDs.
- Assignment verification.
- Remaining unchanged or failed sources.

## Prove Bot Quality With A Golden Set

Customer phrases:

- "Run our golden set."
- "Check expected answers."
- "Compare against expected knowledge references."
- "Make sure the bot does not regress."

Workflow:

1. Treat each row as `question`, `expected answer`, and optional `expected knowledge reference`.
2. Use Generate Answer against the intended agent with test metadata.
3. Capture actual answer, used articles, event IDs, pass/fail, and reviewer notes.
4. For KB-driven failures, generate draft knowledge updates or identify stale sources.
5. Retest failures before publishing.

Proof:

- Total tests, passed, failed, skipped.
- Failed row reasons.
- Used article mismatch when relevant.
- Draft/publish next action.

## Use User Attributes Or Metadata

Customer phrases:

- "Use account status in answers."
- "Pass plan, locale, user ID, tags, or segment."
- "Personalize the bot response."

Workflow:

1. Confirm the exact attributes the caller already knows.
2. Send them through `metadata.user_attributes` on Generate Answer.
3. Keep keys stable and values minimal.
4. Mark synthetic tests clearly.
5. Inspect returned events and used articles.

Rules:

- Do not invent CRM, billing, or helpdesk fields.
- Do not ask users to paste secrets into chat.
- Do not include sensitive fields unless the customer confirms they are safe for the Fini context.

## Make Knowledge Visible To One Agent

Customer phrases:

- "Assign this KB to one bot."
- "Make these articles visible to the billing agent."
- "Why can the agent not use this article?"

Workflow:

1. List agents and resolve `botId`.
2. Fetch the knowledge folders/tree.
3. Confirm the target article lives in the intended folder.
4. Add or remove the folder-agent assignment.
5. Re-fetch the scoped tree for that `botId`.
6. Test behavior if needed.

## Recognize Out-Of-Scope Customer Actions

Customer phrases:

- "Refund this user."
- "Cancel or change a subscription."
- "Retry payment."
- "Update the customer's account."

These are customer-owned business actions. The current bot/KB skills should not promise that Fini public APIs can execute them. First check `https://docs.usefini.com/llms.txt`. If no supported public route exists, gather requirements and state that this needs a separate customer integration or future public action API.

Collect:

- Business action and risk level.
- Read-only vs write.
- Required user/account identifiers.
- Auth method and token lifetime.
- Confirmation and idempotency requirements.
- Success/error response shape.
- Audit trail expected by the customer.
