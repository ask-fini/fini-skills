---
name: fini-api-actions-attributes
description: Use when the user wants to list, inspect, create, update, test, assign, unassign, or delete Fini Actions or Attributes through public APIs; configure their typed input/output schemas; create, order, inspect, update, test, or delete external API calls/Data Steps; preserve or replace masked secrets; expose customer context to an agent; wire an Action into a Rulebook; or diagnose why an Action, Attribute, mapping, or API chain is not running or returning data.
---

# Fini API Actions And Attributes

Use typed Fini tools for Actions, Attributes, and external API calls. Treat the live API index as the endpoint/schema source of truth; keep this skill focused on sequencing, runtime meaning, safety, and proof.

Before endpoint-specific work:

1. Fetch `https://docs.usefini.com/llms.txt`.
2. Read the current Actions and Attributes overview plus the pages for the intended operations.
3. Resolve names to exact workspace IDs and inspect current state before changing it.

## Choose The Resource

| Intent | Resource | Runtime |
| --- | --- | --- |
| Fetch context needed on every reply | Attribute | Runs before every chat message/LLM call for assigned, eligible agent/channel combinations |
| Perform work only when a rule decides | Action | Active behavior comes from a published Rulebook Tool node; an exact draft override can execute only when its owning rule is already published and assigned |
| Configure one HTTP call in either chain | External API call / Data Step | Public reads expose ascending `stepNumber` order; verify it by re-listing the chain |

Actions and Attributes share the `/hc-tools` wire resource. Preserve their semantic distinction:

- Fix `alwaysGet=false` when creating an Action and `alwaysGet=true` when creating an Attribute.
- The shared list returns both kinds. Filter and verify list/get results by `alwaysGet`; never trust a name alone.
- Omit `alwaysGet` on an ordinary same-kind update. Use the documented optional update field only for an explicit conversion: `fini_update_action` with `alwaysGet=true` converts Action -> Attribute; `fini_update_attribute` with `alwaysGet=false` converts Attribute -> Action.
- Treat conversion as an explicit live topology change: snapshot the resource and dependencies, call the typed conversion update, re-fetch the same resource, repair Rulebook/assignment wiring, and retest its new runtime model.

When live endpoint examples conflict with formal `ParamField` definitions or the shared object schema, follow the formal definitions and verify the returned object. Do not copy inconsistent generated examples.

## Build Or Change A Chain

1. Read the target resource, its assignments, and its ordered external API calls.
2. Define the typed input and output contract before editing HTTP steps.
3. Create or update the Action/Attribute record.
4. Create or update external API calls with unique `stepNumber` values and explicit response mappings. Re-list the parent chain and verify the observable ascending order; do not infer or manipulate undocumented internal ordering state.
5. Map final output fields to `[stepId].[responseMappingKey]`.
6. Test the unassigned/unreferenced chain only after the user sees and confirms the sanitized outbound effect.
7. For an Attribute, set output-field `sendToLlm` deliberately, then assign it only to verified eligible agents and supported channels. This flag does not control the agent's complete context.
8. For an Action, wire the exact tested Action ID into an intent Rulebook draft, re-fetch and review its exact `draftVersionId`, then publish only that explicit version. Never resolve a current/latest draft implicitly. Assignment alone is not reachability.
9. Re-fetch every changed resource and verify the chain, schema, mappings, assignments or Rulebook reference, and controlled runtime result.

Read the relevant workflow reference before acting:

- [action-workflows.md](references/action-workflows.md) for Action construction, Rulebook reachability, testing, and removal.
- [attribute-workflows.md](references/attribute-workflows.md) for automatic context, agent assignment, AI visibility, and removal.
- [external-api-calls.md](references/external-api-calls.md) for ordered steps, interpolation, response mappings, secrets, tests, updates, and deletion.

## Mutation Gate

The user may authorize live and destructive work. Before each mutation:

1. Show the exact workspace, resource kind, resource ID, operation, and intended diff.
2. For outbound execution, also show the HTTP method, sanitized host/path, variables, payload shape, and expected external effect. A test is a real request, not a dry run.
3. For assignment, Rulebook wiring, kind conversion, secret replacement, or deletion, explain the runtime consequence.
4. Obtain explicit confirmation when it has not already been given for that exact operation and target.
5. Execute the smallest mutation, then verify with reads.

Do not turn broad permission into permission for an unspecified target. Batch only independently reviewable changes and report partial failures per item.

Actions, Attributes, and Data Steps have no documented draft, version, transaction, or rollback surface. The typed `alwaysGet` conversion is the supported same-resource kind change. For a different material schema or chain change that has no safe intermediate state, prefer building and testing an unassigned/unreferenced replacement, then perform a controlled cutover.

## Secret And Output Rules

- Treat `{ "hide": true, "value": "********" }` as an opaque stored secret, never as the literal secret.
- Data Step reads preserve backend-masked configuration for safe round trips. Echo each returned masked wrapper unchanged to preserve a stored secret during update.
- Replace a secret only when the user explicitly supplies or identifies the replacement; never claim the old value can be recovered.
- Never reproduce secret values in plans, summaries, logs, eval evidence, or test reports.
- Sanitize resolved headers, bodies, and raw response data before reporting.
- Preserve rich API outputs after redaction; do not silently truncate documented fields.

`sendToLlm` applies to fields in the Attribute output schema. It does not prove that no other source, channel metadata, UI metadata, widget JWT fields, prompts, or runtime context are visible to the agent. Before claiming that the AI can see only named fields, inspect the full source/channel context and run an end-to-end proof. If the public API cannot control another context source, state that limit instead of promising exclusivity.

## Delete And Recovery

Before deletion, snapshot the resource, assignments, ordered steps, schemas, mappings, and Rulebook references. Name every dependent ID and obtain exact-target confirmation.

Deleting an Action or Attribute removes that record but does not delete its external API call records. Delete those separately only when intended. Deleting a step can break later `${stepId.responseMappingKey}` references and output-schema paths. Verify the target is absent and check for orphaned or broken dependencies.

There is no public undelete workflow. Recovery means recreating from the sanitized snapshot, receiving new IDs, repairing mappings and wiring, and retesting.

## Proof Of Completion

- Action: exact ID, `alwaysGet=false`, schemas, ordered steps, output mappings, explicit published Rulebook `draftVersionId`, and Tool-node reachability are verified.
- Attribute: exact ID, `alwaysGet=true`, schemas, ordered steps, output-field `sendToLlm` policy, and eligible agent/channel assignments are verified; any broader AI-context visibility remains explicitly scoped.
- External API call: method, sanitized URL, step order, interpolation, response mapping, and masked-secret preservation are verified.
- Conversion: exact tool ID remains traceable; returned `alwaysGet` proves the new kind; existing Data Step IDs, mappings, and masked wrappers were re-read and verified unchanged; obsolete assignments or Rulebook references were reconciled; new runtime behavior was tested. Round-trip a Data Step's complete backend-masked payload only when that step is also intentionally edited.
- Test: confirmation preceded execution; returned status and extracted outputs are reported with secrets and sensitive payloads redacted.
- Delete: exact target is absent; remaining steps, assignments, and Rulebook or mapping references are reconciled; recovery snapshot location or summary is reported.
