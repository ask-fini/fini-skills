---
name: fini-api-coordinator
description: Use when the user asks which Fini public API workflow or skill to use; wants to create, inspect, delete, configure, or analyze an agent; validate unpublished prompt, knowledge, snapshot, or eligible intent-rule candidates; onboard a Fini support bot from Slack; configure Actions, Attributes, or external API calls; manage prompts, Rulebooks, tags, sources, or knowledge; send or inspect conversations; run analytics, replays, rule evaluations, Test Sets, or regression testing; or sequence several Fini API domains end to end. Route to the narrowest Fini skill and verify current endpoint contracts through docs.usefini.com/llms.txt.
---

# Fini API Coordinator

Use this skill as the front door for cross-domain Fini public API work. Route the request to the smallest focused skill, then let that skill own execution and proof.

Before relying on a route, field, enum, scope, response shape, or limit, fetch `https://docs.usefini.com/llms.txt` and the relevant linked Markdown page. Treat those public docs as endpoint truth; keep endpoint tables out of this skill pack.

## Package Freshness

For read-only questions, inspect and report the installed Fini skill-pack version without silently changing it. If the user asks to install or refresh the project-scoped pack, run from the project root:

```bash
npx skills add ask-fini/fini-skills --all --copy -y
```

Use the scoped `add --all --copy` flow so newly released Fini skills are included. A refreshed skill pack still does not replace the live docs check.

## Route By Intent

| User intent | Owning skill |
| --- | --- |
| List, resolve, create, or delete agents; inspect full or sectioned agent analytics | `fini-api-agents` |
| Configure Actions, Attributes for eligible messages, their agent assignments, or ordered external API calls | `fini-api-actions-attributes` |
| Configure prompt drafts/history/publishing, Rulebooks/rules/versions, tags, or tag groups | `fini-api-agent-configuration` |
| Compare active behavior with exact unpublished prompt, knowledge, snapshot, or eligible intent-rule version IDs before publishing | `fini-api-draft-validation` |
| Run Test Sets, criteria, asynchronous regression runs, replays, or rule diagnostics/evaluations | `fini-api-testing` |
| Run a full Slack-led onboarding inside an authenticated Fini workspace | `fini-api-slack-onboarding` |
| List, inspect, export, annotate, resolve, analyze, or delete conversations | `fini-api-conversations` |
| Send or continue one runtime turn when the configuration choice is already known | `fini-api-generate-answer` |
| Crawl, register, ingest, refresh, inspect, or delete raw sources | `fini-api-sources` |
| Generate or manage knowledge, article drafts/history/rollback, folders, and agent visibility | `fini-api-knowledge` |
| Inspect one exact article or saved article version | `fini-api-knowledge`, not Conversations |

Use these distinctions when requests sound similar:

- Use `fini-api-draft-validation` for an active baseline, isolated exact-ID candidates, and a combined unpublished candidate.
- Use `fini-api-generate-answer` for one controlled turn when no candidate-comparison methodology is needed.
- Use `fini-api-conversations` for stored conversation evidence and QA state.
- Use `fini-api-testing` for Replay, rule diagnosis, or persistent live-agent regression. A Test Set does not accept unpublished draft artifact overrides and may invoke Actions or external APIs.
- Use `fini-api-agents` for aggregate analytics rather than rebuilding dashboard metrics from raw exports.
- Use `fini-api-actions-attributes` for a supported business-action integration. The customer still owns the target API, credentials, permissions, idempotency, and business rules.

## Sequence Cross-Domain Work

For a full bot workflow, use only the stages the request needs:

1. Use `fini-api-agents` to resolve an existing agent or create a bare agent when none exists.
2. Use `fini-api-agent-configuration` for prompt drafts, tags, and intent Rulebook drafts; keep Business Rules on their immediate-live, non-versioned lifecycle.
3. Use `fini-api-knowledge` to verify a usable folder tree and create or initialize one when absent before source-backed knowledge generation. This is an operational empty-workspace safeguard, not a prerequisite for importing or polling raw sources.
4. Use `fini-api-sources` to ingest or refresh raw content.
5. Use `fini-api-knowledge` to generate, review, organize, and assign knowledge drafts.
6. Use `fini-api-actions-attributes` when the bot needs live customer context or business operations.
7. Use `fini-api-agent-configuration` to wire Actions into intent Rulebook draft Tool nodes.
8. When unpublished behavior needs proof, use `fini-api-draft-validation` to run an artifact-free baseline, isolated exact-ID candidates, and then an intentional combined candidate.
9. Publish only the separately approved exact prompt, knowledge, and intent-rule versions through their owning skills, then use `fini-api-generate-answer` for active-behavior proof.
10. Use `fini-api-testing` for Replay diagnostics and persistent Test Set regression against current published live-agent behavior.
11. Use `fini-api-conversations` for rollout evidence and improvement inputs.

## Operating Rules

- Read current state before changing it.
- Prefer a reviewable draft when a draft lifecycle exists. Use an immediate-live route when the user explicitly requests it.
- Do not ask for duplicate confirmation when the user has already given clear execution intent and exact targets. If intent or target is ambiguous, explain the live effect and ask one concise question.
- Preserve documented backend limits and return rich structured results; do not invent smaller wrapper limits.
- Treat deletions, article rollback, prompt/rule publish, agent assignment, and external execution as real mutations, not as read-only checks.
- Treat Action, Attribute, external-call, rule-evaluation, replay, Generate Answer, and Test Set execution as potentially side-effecting when Tool nodes or customer APIs are involved. A live Test Set run can invoke Actions.
- For synthetic Generate Answer candidate traffic, use `markAsTest: true`; this marks traffic but does not suppress configured Attributes, Actions, external effects, or stored events.
- Persist candidate content through its owning API and pass only exact stored IDs in Generate Answer `artifacts`. Never inline prompt, knowledge, or rule content there.
- Never ask for or repeat raw secrets in chat. Preserve a documented masked value only when the API defines that as the way to keep an existing stored secret.
- Translate customer language into Fini concepts:
  - "train/sync docs" means Sources -> Knowledge -> visibility -> proof;
  - "use account context" may mean configured Attributes for eligible messages, not only caller-supplied runtime metadata;
  - "refund/cancel/update an account" can mean a Fini Action backed by a customer-owned API;
  - "prove we did not regress" means a persistent Test Set run against current published live-agent behavior, with external effects preflighted.

## Boundaries

- Agent creation creates a bare agent inside an existing authenticated workspace. It does not create a Fini workspace, install Slack, authorize a provider, configure prompts, add knowledge, or wire Actions.
- Provider OAuth and Slack/channel installation may still require dashboard or user-owned setup when the live public docs expose no route.
- An Attribute assigned to an agent runs before eligible message generations for its assigned agent/channel. Active Action behavior requires a published Rulebook Tool node; a request-scoped exact draft override can execute only when its owning rule is already published and assigned. An Action assignment record alone is not runtime reachability.
- A completed asynchronous job or run is not sufficient proof. Require the expected persisted IDs or terminal result and re-read the resulting state.

## High-Value Gotchas

- Prompt changes should normally follow create draft -> fetch exact version -> review -> publish exact version. Handle stale-draft conflicts by re-reading current prompts.
- Natural-language intent Rulebook work should normally create or refine a draft before publishing its explicit exact reviewed `draftVersionId`; never resolve a current/latest draft implicitly. Business Rules are immediate-live and non-versioned.
- Rule duplication is live-impacting for every kind. Route it through exact source-kind inspection, exact confirmation, and re-fetch of the independent copy.
- Tag groups marked output-only are unavailable to Rulebooks.
- Source ingestion is not live knowledge, and published knowledge can still be invisible when folder assignment is wrong.
- Public source APIs use `document*` wire names even though product language says "sources."
- Conversation/source pagination is newest-first; follow the cursor contract from the current docs.
- Generate Answer creates events; it is not a conversation-list response.
- Draft validation does not publish. Test one artifact dimension at a time before a combined candidate, and keep every comparison like-for-like.
- Test Set runs asynchronously evaluate persistent cases against current published live-agent behavior and may invoke configured Actions or external APIs.
- Action/Attribute tests and individual external-call tests execute outbound HTTP; the word "test" does not imply a dry run.
- Analytics requires an explicit agent and time window; use sectioned analytics when the user only needs one view.

## References

Read [api-mental-model.md](references/api-mental-model.md) for architecture, behavior-change layers, triage, or end-to-end agent setup.

Read [workflow-routing.md](references/workflow-routing.md) for a multi-step plan or detailed skill selection.

Read [customer-recipes.md](references/customer-recipes.md) when the request is phrased as a customer job rather than an API domain.
