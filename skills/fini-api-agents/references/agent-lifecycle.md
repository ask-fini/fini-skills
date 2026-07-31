# Agent Lifecycle

Use this reference for exact agent resolution, bare-agent creation, dependency-aware deletion, and post-create handoff.

## Public Endpoints

| Operation | Method and path | Scope | Important behavior |
| --- | --- | --- | --- |
| List agents | `GET /v2/bots/public` | `read` | Returns every non-deleted agent in one unpaginated array |
| Create agent | `POST /v2/bots/public` | `write` | Creates one agent from a unique `name` |
| Delete agent | `DELETE /v2/bots/{id}/public` | `write` | Removes one exact agent from the workspace |

Fetch the current pages from `https://docs.usefini.com/llms.txt` before using these contracts.

## Resolve An Agent

1. List agents.
2. Match the user's supplied ID exactly when present.
3. For a supplied name, show any exact and close matches before choosing. Do not silently choose among ambiguous names.
4. Retain the returned `id` as `botId` for downstream calls.

List agents includes the name, creation timestamp, and rendered planning, guideline, email, and chat prompt strings. Those prompt strings are useful for inspection but are not an editable prompt model. Use Get prompts from `fini-api-agent-configuration` before modifying prompt sections.

There is no pagination, filtering, or limit on List agents. An empty array means no non-deleted agents are visible to the current workspace credential.

## Create A Bare Agent

1. List agents and check whether the intended name already exists.
2. If it exists, reuse it only when the user intends that exact agent; otherwise choose a genuinely distinct name.
3. Create the agent with the intended name.
4. Capture `id`, `name`, `companyId`, timestamps, and `hcFlowMode` when returned.
5. Re-list agents and verify the new ID and name.
6. Inspect and configure the required downstream layers rather than assuming defaults.

The endpoint guarantees creation of an agent record. It does not guarantee:

- a new workspace;
- Slack or another channel installation;
- provider OAuth or customer integration authorization;
- reviewed prompts, rules, tags, knowledge, Actions, or Attributes;
- production readiness.

Route follow-up work:

1. `fini-api-agent-configuration` for prompt drafts, rules, and tags.
2. `fini-api-sources` and `fini-api-knowledge` for content, folders, and visibility.
3. `fini-api-actions-attributes` for external capabilities and agent assignments.
4. `fini-api-draft-validation` for unpublished candidates, marked `fini-api-generate-answer` for targeted active smoke, then `fini-api-testing` for focused diagnostics or persistent regression against current published live-agent behavior.

## Dependency Preflight Before Delete

Resolve the exact ID, then create a concise dependency record:

- agent name, ID, creation time, and rendered prompt snapshot;
- structured prompts and version IDs needed for audit or reconstruction;
- knowledge folders assigned to the agent;
- Action and Attribute assignment junctions containing the agent ID;
- published and draft rules whose `botIds` contain the agent ID;
- recent conversation volume and any Test Sets or operational reports that depend on its conversations;
- channel or provider integrations known outside the public agent endpoint.

This inventory identifies impact; it is not an instruction to delete or unassign every dependency. Change related resources only when the user requests that additional scope.

The public deletion documentation does not define downstream cascade, restoration, retention, or recovery guarantees. Do not promise that dependent records are deleted, retained, or recoverable. Preserve IDs and relevant configuration before deletion when later audit or reconstruction matters.

## Delete And Verify

1. Re-list immediately before deletion and bind the request to one exact ID and current name.
2. Summarize the dependency findings and the expected loss of agent availability.
3. Proceed when the current request clearly authorizes deletion of that exact resource. Do not require the user to repeat authorization already given.
4. Delete `/v2/bots/{id}/public`.
5. Record the returned message.
6. Re-list agents and verify the deleted ID is absent.
7. Report any dependent resources whose post-delete status remains unknown; do not infer cascade from the agent's absence.

A `404` before deletion means the ID is not a visible agent in this workspace, not proof that a similarly named agent should be substituted. A successful response without subsequent list verification is incomplete proof.
