# Workflow Routing

Use this reference to choose a skill or sequence for a Fini API goal. Fetch `https://docs.usefini.com/llms.txt`, then the relevant linked pages, before using endpoint details.

If the request mixes agent lifecycle, configuration, content, integration, runtime, and quality concerns, read [api-mental-model.md](api-mental-model.md) first.

## Contents

- [Routing Rules](#routing-rules)
- [Choose The Right Proof Surface](#choose-the-right-proof-surface)
- [Cross-Domain Recipes](#cross-domain-recipes)
- [Clarify Only Material Ambiguity](#clarify-only-material-ambiguity)
- [Do Not Route To Fini API Skills](#do-not-route-to-fini-api-skills)

## Routing Rules

| Customer phrase | Route |
| --- | --- |
| "Create a support agent" / "delete this agent" | `fini-api-agents` |
| "Show this agent's analytics, trends, knowledge usage, rules, or escalations" | `fini-api-agents` |
| "Onboard us to Fini" / "set up our support bot and KB from Slack" | `fini-api-slack-onboarding` after workspace-state classification |
| "Update the bot prompt" / "show prompt history" / "publish this reviewed prompt draft" | `fini-api-agent-configuration` |
| "Show when we use no reply, an internal comment, or a direct reply" | Audit Reply Rules and fields context in `fini-api-agent-configuration` |
| "Compare these unpublished prompt, KB, snapshot, or Rulebook candidates with the current bot" | `fini-api-draft-validation` |
| "Create or test a Rulebook" / "show rule versions" | Author in `fini-api-agent-configuration`; diagnose or regress in `fini-api-testing` |
| "Create routing or outcome tags" | `fini-api-agent-configuration` |
| "Fetch account context for eligible messages" | `fini-api-actions-attributes` Attribute workflow |
| "Refund, cancel, look up, or update through our API" | `fini-api-actions-attributes` Action workflow, then Rulebook wiring in `fini-api-agent-configuration` |
| "Configure or test an external API step" | `fini-api-actions-attributes` |
| "Export, inspect, annotate, resolve, or delete conversations" | `fini-api-conversations` |
| "Explain why this exact Fini response used this knowledge, tag, Attribute, or Rule" | Exact event metadata in `fini-api-conversations` |
| "Refine/fix this bad Fini answer, propose the right draft changes, and show the improved replay" | `fini-api-refine-with-ai` |
| "Send one message to the bot" | `fini-api-generate-answer` |
| "Replay this historical answer with this exact candidate" | `fini-api-testing` |
| "Run our regression suite" / "compare pre- and post-change quality" | `fini-api-testing` Test Suite workflow |
| "Evaluate this rule against context or a conversation" | `fini-api-testing` |
| "Import or refresh web, Zendesk, Notion, or Confluence content" | `fini-api-sources`, then `fini-api-knowledge` if it should affect answers |
| "Generate, inspect, version, draft, publish, roll back, or assign an article/KB content" | `fini-api-knowledge` |

## Choose The Right Proof Surface

| Need | Use |
| --- | --- |
| One new turn | Generate Answer |
| One active-vs-unpublished exact-ID candidate comparison | Draft Validation |
| One exact bad response plus AI diagnosis, generated draft fixes, and embedded replay | Refine with AI |
| One historical answer under another configuration | Replay |
| One Rulebook's node-level behavior | Rule evaluation |
| Persistent regression against captured published or selected draft configuration | Test Suite |
| Raw stored conversation evidence | Conversations |
| One exact Fini-authored event's stored execution trace | Event metadata through Conversations |
| Current no-reply, internal-comment, and direct-reply conditions | Reply Rules plus fields context through Agent Configuration |
| Aggregate agent performance | Agent analytics |

These surfaces complement each other. Do not substitute raw conversation aggregation for analytics, or a handful of Generate Answer calls for a requested persistent Test Suite. Draft Validation applies unpublished artifacts to new Generate Answer requests. Test Suite runs saved cases with captured published/draft configuration and simulated Actions. Event metadata is stored evidence for one exact Fini-authored event, while Reply Rules describe current workspace response-mode configuration. Neither should be presented as the other, and Reply Rules are not Rulebooks.

## Cross-Domain Recipes

### Full Slack-Led Onboarding

Use only for full onboarding inside an authenticated workspace:

1. `fini-api-slack-onboarding`: inspect state and classify fresh, expansion, or targeted work.
2. `fini-api-agents`: resolve the target or create a bare agent when none exists.
3. Confirm any user-owned Slack installation and provider OAuth prerequisites.
4. `fini-api-knowledge`: verify a usable tree and create or initialize one when absent before source-backed knowledge generation.
5. `fini-api-sources`: ingest and poll source content.
6. `fini-api-knowledge`: generate, review, organize, and assign drafts.
7. `fini-api-agent-configuration`: create prompt drafts, tags, and intent Rulebook drafts; keep Business Rules immediate-live and non-versioned.
8. `fini-api-actions-attributes`: add only the Attributes and Actions required by the use case.
9. `fini-api-agent-configuration`: wire Actions into reviewed intent Rulebook draft Tool nodes.
10. `fini-api-draft-validation`: compare active behavior with isolated exact-ID candidates, then the reviewed combined candidate when needed.
11. Publish separately approved exact versions through their owning skills.
12. `fini-api-generate-answer`: create fresh active-behavior conversations and retain their IDs.
13. `fini-api-testing`: run the persistent Test Suite against captured published or selected draft configuration, poll it, inspect case-level verdicts and mock diagnostics, and use the terminal result as the go-live gate.

### Customer API Action

1. Confirm the customer-owned operation, identifiers, permissions, auth approach, idempotency behavior, and expected response.
2. `fini-api-actions-attributes`: define the typed Action and output contract.
3. Add ordered external API steps and response mappings without exposing stored secrets.
4. Test only against a target where the requested real effect is acceptable.
5. `fini-api-agent-configuration`: wire the Action into an intent Rulebook draft Tool node and record its exact reviewed `draftVersionId`.
6. Review and publish the explicit exact Rulebook `draftVersionId`; never omit it or infer a current/latest draft.
7. Use marked Generate Answer with an acceptable target for a focused active smoke. Use Replay only for separate historical diagnosis.
8. `fini-api-testing`: run persistent success, denial, failure, and escalation cases under captured configuration; verify recorded/custom Action mocks and saved context; a real HTTP integration smoke requires separate execution intent.

### Eligible-Message Customer Context

1. Identify the minimum customer fields the agent needs on eligible messages.
2. `fini-api-actions-attributes`: create an Attribute and its external call steps.
3. Map only the required output fields and choose which fields reach the model or Rulebooks.
4. Test a controlled lookup, then assign the Attribute to the exact agent/channel.
5. Verify eligible-message runtime behavior and latency with marked Generate Answer.
6. Run persistent Attribute cases through Test Suite using captured Attribute outputs or explicit overrides. Test real provider lookups separately.

### Prompt Or Rule Change

1. Read current configuration and supporting evidence.
2. Create or refine a reviewable draft.
3. Use `fini-api-draft-validation` to establish an artifact-free baseline, persist/fetch exact IDs, test one dimension at a time, then run a combined candidate when justified.
4. Review and publish the exact version.
5. Run the same persistent Test Suite against current published behavior and compare with the prior run.
6. Use Conversations and Analytics for rollout follow-up.

### First-Time Knowledge Import

1. `fini-api-knowledge`: verify a usable tree and create or initialize one when absent before source-backed generation.
2. `fini-api-sources`: crawl/register/ingest and poll.
3. Generate drafts and require persisted article/draft IDs.
4. Review and validate exact drafts when needed, then publish, organize, and assign.
5. Use marked Generate Answer for a focused active retrieval smoke.
6. Gate go-live with a persistent Test Suite against current published behavior; verify saved Attribute context, Action mocks, and effective criteria.

### Conversation Evidence To Improvement

1. `fini-api-conversations`: preserve exact conversation/event evidence and QA state.
2. When the user wants Fini to diagnose and propose the correction, use `fini-api-refine-with-ai`; read the active session first, create one confirmed iteration, and poll it to a terminal state.
3. Review the old/replayed answers, root cause, and exact returned article/prompt/rule draft IDs. Do not recreate those drafts manually.
4. Route each artifact to Knowledge or Agent Configuration for exact review and separate publication approval.
5. Use Draft Validation only for additional clean cases or combined-candidate proof; after publish, rerun the stable Test Suite against captured published or selected draft configuration.
6. Mark feedback resolved only after the live evidence supports it.

## Clarify Only Material Ambiguity

Ask one concise question when the wrong route or target could produce an unintended live effect:

- "Which existing agent should I use, or should I create a bare agent named X?"
- "Should this be an Attribute for eligible messages or a Rulebook-invoked Action?"
- "Is this external API target safe for the requested real test operation?"
- "Do you want a reviewable draft or an immediate-live update?"
- "Which exact agent, version, article, or resource should be deleted or rolled back?"

Do not ask again when the user already supplied clear intent and an exact target.

## Do Not Route To Fini API Skills

- Internal production debugging that requires non-public systems.
- Frontend widget or embed code changes.
- Creating a brand-new Fini workspace when no current public route exists.
- Installing Slack or completing provider OAuth when no current public route exists.
- General HTTP-client work with no Fini workflow.
