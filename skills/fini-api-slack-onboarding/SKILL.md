---
name: fini-api-slack-onboarding
description: "Use only when the user or client explicitly wants a full Slack-led Fini onboarding for an existing Fini workspace or agent: setting up a new Fini support bot and knowledge base, onboarding to Fini from Slack, configuring bot behavior plus sources/knowledge/prompts/rulebooks/tags, running tests, and preparing go-live readiness. First verify whether this is a fresh onboarding by inspecting workspace state; if it is only a targeted KB import, prompt edit, rule change, source refresh, or answer test, route to the narrower Fini API skill instead."
---

# Fini API Slack Onboarding

Use this skill as the conductor for Slack-guided onboarding of a new Fini client or a newly adopted Fini support agent. It coordinates the narrower Fini API skills; it does not replace them.

Before endpoint-specific work, fetch `https://docs.usefini.com/llms.txt` and the relevant docs pages. Do not copy endpoint schemas into the answer.

## Boundary

- This flow assumes the client already has, or can authenticate into, an existing Fini workspace.
- Dashboard/UI is only for provider OAuth or connector setup when the public APIs do not expose that step.
- Do not promise workspace creation, Slack installation, provider OAuth, or brand-new agent provisioning unless current docs expose those routes.
- Do not use this skill for one-off tasks like "import this URL", "update this article", "test this prompt", or "export conversations". Route those to the focused skills.

## Fresh Onboarding Check

For ambiguous requests such as "set up my Fini bot and KB", inspect current workspace state before starting the full onboarding interview:

1. List agents to see how many bots exist and identify the likely target.
2. List articles and drafts to estimate existing KB volume.
3. Get knowledge folders, optionally scoped to the target bot, to see whether a tree and assignment already exist.
4. List sources to see whether web/provider content has already been ingested.
5. If prompt/rule/tag APIs are relevant, inspect current prompts, rules, tag groups, and tags through current docs.

Classify the request:

| Classification | Meaning | Next action |
| --- | --- | --- |
| Fresh onboarding | Minimal sources/KB/config or a newly adopted agent | Continue this skill |
| Existing client expansion | Meaningful KB/config already exists | Ask whether they want full onboarding or a targeted update |
| Targeted update | User wants one specific operation | Route to `fini-api-sources`, `fini-api-knowledge`, `fini-api-agent-configuration`, or `fini-api-generate-answer` |

## Intake Modes

Offer two modes, then follow the client preference:

- All at once: ask the client to drop a Markdown/PDF/doc bundle, help center URL, SOPs, behavior notes, test questions, and examples in one message.
- Step by step: ask one section at a time and keep an explicit onboarding state.

Read [intake-and-state.md](references/intake-and-state.md) when collecting inputs, tracking progress, or producing the final readiness summary.

## Core Workflow

1. Resolve the target agent with `fini-api-coordinator` or the List agents docs.
2. Confirm provider/source access:
   - if provider OAuth is not done, tell the client to complete it in dashboard/UI;
   - if source URLs or files are provided, continue through API workflows.
3. Use `fini-api-sources` to crawl, register, ingest, refresh, and poll sources.
4. Use `fini-api-knowledge` to generate draft knowledge, propose or inspect folders, publish after approval, and assign folders to the target agent.
5. Use `fini-api-agent-configuration` to design tags, draft/refine/publish rules, and inspect/update prompts.
6. Use `fini-api-generate-answer` to run a mini onboarding test suite.
7. Use `fini-api-conversations` only when real conversation evidence, QA exports, or golden-set candidates are needed.

## Defaults

- Prefer draft/review paths before live changes.
- Prefer source-backed knowledge generation for imported docs and SOPs.
- Prefer rule drafts for natural-language behavior setup.
- Read current state before writes.
- Ask for explicit approval before publish, prompt update, rule publish, delete, or folder assignment that changes production retrieval.
- Never ask clients to paste raw API keys into Slack or chat.
- Keep behavior failures separate from knowledge failures.

## Test Loop

Build a mini test suite from the intake:

- top customer questions
- expected answers or success criteria
- out-of-scope questions
- escalation cases
- tone/personality checks
- one or two edge cases from past tickets or bad-answer examples

For each failure:

| Failure type | Fix path |
| --- | --- |
| Missing or stale facts | Sources -> Knowledge draft -> review/publish/assign |
| Correct facts, wrong style | Prompt update plan |
| Wrong escalation/routing/workflow | Rule draft/refine/publish |
| Wrong category/reporting output | Tag group/tag design, then rules if needed |
| Needs caller/account context | Generate Answer metadata/user attributes |

Retest changed cases before marking onboarding ready.

## Completion Proof

End with a concise readiness report:

```markdown
Onboarding classification: [fresh / expansion / targeted update]
Target agent: [name / botId]
Sources: [not started / ingested / refreshed / blocked]
Knowledge: [drafts generated / published / assigned / blocked]
Folders: [tree proposed / persisted / assigned / blocked]
Prompts: [not changed / proposed / updated / verified]
Rules/tags: [not changed / drafted / published / verified]
Tests: [not run / pass / fail / needs review]
Approvals pending: [publish / prompt / rule / assignment / none]
Go-live blockers: [list]
Next action: [one concrete step]
```
