# Attribute Workflows

Read the live Attributes overview and operation pages discovered through `https://docs.usefini.com/llms.txt` before using a route.

## Create A Working Attribute

1. Resolve the workspace, exact agents, source, supported channels, and context required on their replies. Verify that each intended agent/channel combination is eligible in the current workspace.
2. Inspect Attributes with `alwaysGet=true` to avoid duplicate lookups.
3. Define typed inputs, typed outputs, ordered external API calls, and per-output AI visibility.
4. Create the resource as an Attribute with initial kind `alwaysGet=true`. Creation returns an empty schema shell. Choose `source` correctly here; the documented update body does not include it, and `internalHandler` is read-only. A later explicit `fini_update_attribute(alwaysGet=false)` can convert this same resource into an Action.
5. Update the input schema and draft any outputs not dependent on a server-generated step ID.
6. For each Attribute output, set `sendToLlm=true` only when the agent needs that value. Keep operational or sensitive output fields hidden without treating this as a global agent-context control.
7. Add and map the external API calls.
8. Test the real chain with controlled inputs after an outbound-effect confirmation.
9. Assign the exact Attribute ID to verified eligible agent IDs and supported channels with `ADD`.
10. Re-list assignments and verify the Attribute in a controlled conversation.

An Attribute runs only for an assigned, eligible responding agent/channel combination. Unlike Actions, the assignment junction is a runtime gate, but it does not make an unsupported source or channel eligible.

When the server assigns step IDs, finalize API-backed output-schema paths after creating the steps. Expect a second parent-schema patch; do not leave bare mapping keys.

## Context And Visibility

Use an Attribute for information needed automatically in conversations, such as account state, subscription plan, or recent-order context. Attached Attributes are fetched before every chat message/LLM call and refresh on the next message. Use an Action for work that should run only after a Rulebook decision.

Attribute fields may bind from metadata, JWT context, literals/defaults, or earlier API responses according to the current schema. Do not invent runtime paths; inspect the channel's actual metadata contract first.

For API-backed output fields:

- map the response in the owning external API call;
- set `source="apiResponse"`;
- use `path="[stepId].[responseMappingKey]"`;
- set `sendToLlm` deliberately.

Visibility to AI is a data-minimization decision, not proof that the data is secret-safe. Avoid exposing tokens, raw headers, full customer records, or fields irrelevant to the response. The public schema documents `sendToLlm`; do not invent fields for other dashboard switches.

`sendToLlm` controls Attribute output-schema fields only. Other context can already be visible through the source, channel metadata, UI metadata, widget JWT fields, prompts, or runtime state. To satisfy a request such as "only expose these two fields," inspect those other context surfaces and validate a controlled conversation. If the public API cannot disable another visible source, report that limitation rather than claiming exclusivity.

## Update

1. Read the current Attribute, assignments, schema, steps, and AI visibility policy.
2. Prepare the smallest patch and omit `alwaysGet` for an ordinary Attribute update.
3. Preserve masked secrets in Data Steps.
4. Reconcile renamed or removed fields with response mappings and downstream prompt/rule references.
5. Confirm changes that broaden AI visibility, agent reach, channels, or outbound effects.
6. Patch, re-fetch, test the chain, and validate a controlled conversation.

For an explicitly requested Attribute-to-Action conversion, snapshot the Attribute, Data Steps, backend-masked configuration, assignments, and downstream consumers. Pre-stage the intended Rulebook wiring and automatic-assignment cleanup. Call `fini_update_attribute` with `alwaysGet=false`, re-fetch the converted resource and its unchanged steps, remove obsolete automatic agent/channel assignments, establish Rulebook Tool-node reachability, and verify on-demand behavior. The conversion and rewiring are not atomic and have no documented rollback; use a maintenance window or a deliberately prepared replacement when an intermediate state would be unsafe. Do not recreate or delete the resource merely to change kind when this same-resource cutover is acceptable.

For a material change to an active Attribute, prefer building and testing an unassigned replacement, then cut over agent/channel assignments deliberately. There is no documented draft/version/rollback surface for the Attribute or its Data Steps.

## Assign Or Unassign

Use junction actions `ADD` and `DELETE` with exact `botId`, `toolId`, and a channel list supported by the current workspace/docs. If channels are omitted, do not infer all-channel behavior; verify the resulting junction and runtime behavior.

Before adding, explain that the Attribute will run automatically for that agent's conversations and may add latency or external API traffic. Before removing, explain which context the agent will lose.

Re-list assignments after the mutation. Do not infer success only from `{ "success": true }`.

## Test

An Attribute test runs the full external API chain. It is side-effecting whenever any step or upstream endpoint has effects, even though the product calls it a test.

Preflight the Attribute ID, ordered sanitized requests, controlled inputs, expected effects, and visibility policy. Confirm before execution. Afterward, inspect:

- per-step status;
- final `extractedData`;
- `attributeLlmPolicy`;
- redaction of response bodies and resolved request details.

The test proves the chain, not assignment behavior. Use a controlled Generate Answer or equivalent runtime proof for the exact assigned agent when the user requests end-to-end validation.

## Delete And Recover

Snapshot the Attribute, assignments, schemas, mappings, and Data Steps. Explain that deleting it stops its automatic context for all assigned agents.

Deleting the Attribute does not delete its Data Steps. Decide separately whether each step should be retained or deleted. Verify the Attribute is absent, inspect remaining steps, and validate affected agents no longer depend on its outputs.

Recovery recreates a new Attribute and steps with new IDs. Repair output paths, restore assignments, test the chain, and validate one controlled conversation.
