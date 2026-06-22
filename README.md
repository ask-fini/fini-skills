# Fini API Skills

Workflow skills for AI coding agents using the Fini public API for Fini client bot/runtime and knowledge-base management.

These skills do not replace the Fini API docs. They teach an agent how to choose the right workflow, sequence API calls, handle async polling, avoid draft/live and source/knowledge mistakes, configure existing agents through prompts/rules/tags, translate customer phrases like "train the bot" or "run a golden set" into API workflows, and verify that an API workflow actually worked.

## Install

```bash
npx skills add ask-fini/fini-skills
```

List available skills without installing:

```bash
npx skills add ask-fini/fini-skills --list
```

Install all skills non-interactively:

```bash
npx skills add ask-fini/fini-skills --all
```

## Skills

| Skill | Use when |
| --- | --- |
| `fini-api-coordinator` | Choose the right Fini public API workflow, update/check skill freshness, and route customer-language jobs to focused skills. |
| `fini-api-agent-configuration` | Configure existing Fini agents with prompts, prompt history, rulebooks/rules, rule drafts, tags, and tag groups. |
| `fini-api-conversations` | Export, inspect, analyze, paginate, retrieve, delete, bulk-delete, and build QA/golden-set evidence from Fini conversations. |
| `fini-api-generate-answer` | Send a message turn into a Fini agent, start/continue conversations, pass user attributes, test bot behavior, and interpret returned events. |
| `fini-api-sources` | Import, crawl, register, ingest, refresh, poll, detect changed content for, or delete Fini sources before knowledge generation. |
| `fini-api-knowledge` | Generate, manage, publish, improve, organize, move, and assign Fini knowledge and folders. |

## Freshness

The coordinator skill tells agents to update only the Fini skills before endpoint-specific work. For project-scoped installs, run from the project root:

```bash
npx skills update --project fini-api-coordinator fini-api-agent-configuration fini-api-conversations fini-api-generate-answer fini-api-sources fini-api-knowledge -y
```

For global installs, use `--global` instead of `--project`. If the install is not tracked by the CLI, re-run:

```bash
npx skills add ask-fini/fini-skills --all --copy -y
```

If a newly released Fini skill is missing after `skills update`, run the same `skills add --all --copy -y` command once so the local lock file includes the new skill.

Do not use a broad `npx skills update` as a Fini workflow step; that can update unrelated installed skills.

Endpoint details should still be checked against current docs:

```text
https://docs.usefini.com/llms.txt
```

## Design Boundary

These skills intentionally avoid copying endpoint tables, request schemas, and long examples from the docs. The docs are the source of truth for endpoint facts. The skills add:

- workflow choice
- endpoint sequencing
- API mental models and lifecycle expectations
- agent prompt/rule/tag configuration sequencing
- async polling expectations
- source ingestion vs live knowledge distinction
- draft vs publish behavior
- folder/agent assignment checks
- golden-set and QA improvement loops
- user-attribute and runtime-test guidance
- read/write scope awareness
- destructive-action confirmation rules
- proof-of-completion templates

## Current Shape

The current package uses six skills. Knowledge generation, article/draft management, and tree/assignment workflows are grouped under `fini-api-knowledge`. Prompts, rulebooks/rules, and tags/tag groups are grouped under `fini-api-agent-configuration` because they compose into one existing-agent configuration workflow. These can be split later if customer usage shows either skill triggers too broadly.
