# Fini API Skills

Workflow skills for AI coding agents using the Fini public API for agent lifecycle, configuration, customer API integrations, knowledge, runtime, analytics, conversations, Refine with AI, and quality.

These skills do not replace the Fini API docs. They teach an agent how to choose and sequence workflows, handle async and versioned lifecycles, distinguish draft from live state, preserve source/knowledge boundaries, use Refine with AI for conversation-grounded fixes, validate exact unpublished candidate artifacts, run persistent cases against current published behavior, and verify that an API workflow actually worked.

## Install In Claude

The Fini Claude plugin installs all twelve workflow skills and connects the
authenticated Fini MCP toolset in one package.

In Claude:

1. Open **Settings → Plugins**.
2. Select **Add → Add marketplace**.
3. Enter `ask-fini/fini-skills` and select **Sync**.
4. Install the **Fini** plugin.
5. Connect Fini, sign in, and approve access to the intended workspace.

Claude registers the public OAuth client automatically when you click
**Connect**. No OAuth client ID, Fini API key, OAuth secret, or workspace
credential needs to be entered or stored in this repository.

If standalone Fini skills or the standalone Fini connector are already
installed, remove them before installing the plugin to avoid duplicate skills
or connector entries.

Marketplace updates come from this GitHub repository. Refresh the marketplace
to pick up new skill releases. MCP tool changes are delivered by the hosted
service and appear after Claude reconnects to it.

For maintainers: bump the version in `.claude-plugin/plugin.json` and
`package.json` whenever releasing plugin or skill changes so synced Claude
marketplaces recognize the new release.

## Install In Coding Agents

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
| `fini-api-coordinator` | Route cross-domain Fini work across agent lifecycle, configuration, Actions/Attributes, knowledge, runtime, analytics, conversations, and quality. |
| `fini-api-slack-onboarding` | Run full Slack-led onboarding, validate unpublished candidates when needed, and use a controlled live Test Set run as the go-live gate. |
| `fini-api-agent-configuration` | Draft, review, and publish versioned prompts and intent Rulebooks; safely manage immediate-live Business Rules, tags, and tag groups; audit workspace Reply Rules. |
| `fini-api-agents` | Create, inspect, delete, and analyze Fini agents, then route them into configuration, knowledge, Actions/Attributes, and testing workflows. |
| `fini-api-actions-attributes` | Configure Actions, eligible-message Attributes, external API steps, assignments, Rulebook wiring, testing, and cleanup. |
| `fini-api-conversations` | Export, inspect, annotate, rate, resolve, replay-route, analyze, and delete Fini conversation evidence, including exact-event execution traces. |
| `fini-api-refine-with-ai` | Plan or execute a bad-response improvement workflow, generate reviewable prompt/article/rule drafts, inspect the improved replay, and route exact artifacts toward separately approved publication. Execution requires one exact eligible response. |
| `fini-api-generate-answer` | Start or continue ordinary active-runtime turns, pass caller context, and capture conversation and Action evidence. |
| `fini-api-draft-validation` | Compare an active baseline with exact unpublished prompt, knowledge, snapshot, or eligible intent-rule candidates through request-scoped artifact IDs without publishing them. |
| `fini-api-sources` | Import, crawl, register, ingest, refresh, poll, detect changed content for, or delete Fini sources before knowledge generation. |
| `fini-api-knowledge` | Generate and manage knowledge, article drafts/history/rollback, folders, publishing, organization, and agent visibility. |
| `fini-api-testing` | Choose among Generate Answer, replays, rule evaluation, analytics, and persistent Test Sets that run against current published live-agent behavior; interpret regression evidence and external effects. |

## Freshness

For read-only work, the coordinator inspects and reports the installed Fini skill-pack version without silently changing it. When the user asks to install or refresh the project-scoped pack, run from the project root:

```bash
npx skills add ask-fini/fini-skills --all --copy -y
```

This intentionally uses `add --all --copy` rather than a hardcoded `skills update --project <skill names>` command so newly released Fini skills are installed too.

List current package skills with:

```bash
npx skills add ask-fini/fini-skills --list
```

Do not use a broad `npx skills update` as a Fini workflow step; that can update unrelated installed skills.

Endpoint details should still be checked against current docs:

```text
https://docs.usefini.com/llms.txt
```

## Design Boundary

These skills intentionally avoid copying endpoint tables, request schemas, and long examples from the docs. Use the current contract layer that matches the call:

- REST docs are authoritative for direct HTTP routes, fields, enums, limits, and response shapes.
- The current MCP tool schema is authoritative for connector-only inputs, translations, defaults, and confirmation gates.
- Skills provide workflow judgment, sequencing, and safety; verified runtime state is observed evidence and should be labeled as such.

The skills add:

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

## Hard Workflow Guardrails

- As an operational safeguard for empty or stale workspaces, verify a usable folder tree and create or initialize one when absent before source-backed knowledge generation. Source ingestion itself can happen before this check.
- Source ingestion is not KB, and source processing does not mean answers changed.
- A completed knowledge job is only successful when it produces an article/draft ID, or the no-op/duplicate result is explicitly accepted.
- Folder reads are snapshot-backed; if snapshots disagree with current agents/articles/sources, stop and flag stale workspace state.
- Knowledge is ready only after article/draft verification, publish/review decision, folder assignment, and runtime proof when needed.
- Draft Validation sends exact persisted artifact IDs only, marks synthetic traffic, isolates one candidate dimension at a time, and never implies publication.
- The MCP connector's `markAsTest: true` input is not a dry run. Generate Answer and live Test Set runs can invoke Attributes, Actions, Data Steps, and customer APIs; use controlled identities and acceptable effects. Direct REST docs expose no top-level field by that name.
- Test Sets run persistent cases against current published live-agent behavior. They do not accept request-scoped unpublished draft overrides.
- Keep ordinary Action/Attribute updates same-kind by omitting `alwaysGet`. Use the typed update's optional `alwaysGet` only for an explicit in-place Action-to-Attribute or Attribute-to-Action conversion, then reconcile the non-atomic runtime topology cutover.
- Build every Data Step update from a fresh complete backend read, preserve `{ "hide": true, "value": "********" }` wrappers unchanged, never use a sanitized summary as the write source, and block when a secret-bearing wrapper is missing or truncated.

## Current Shape

The current package uses twelve skills. `fini-api-refine-with-ai` owns planning and execution of AI-assisted bad-response diagnosis, generated draft lineage, and the embedded replay; execution begins only after resolving one exact eligible response. Conversations retains evidence and QA state, while Knowledge and Agent Configuration retain exact review and publication. `fini-api-slack-onboarding` remains intentionally narrow: it should trigger only for full new-client-style onboarding, then route into focused skills. Agent lifecycle and analytics live in `fini-api-agents`; customer API integrations live in `fini-api-actions-attributes`; exact pre-publish artifact proof lives in `fini-api-draft-validation`; ordinary active-runtime turns live in `fini-api-generate-answer`; and standalone Replay, rule diagnostics, and persistent Test Sets against current published behavior live in `fini-api-testing`.
