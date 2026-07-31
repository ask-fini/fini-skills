# Action Workflows

Read the live Actions overview and operation pages discovered through `https://docs.usefini.com/llms.txt` before using a route.

## Create A Working Action

1. Resolve the workspace and inspect Actions with `alwaysGet=false`.
2. Define:
   - the user intent and external effect;
   - required typed inputs;
   - typed outputs needed by downstream Rulebook nodes;
   - the ordered HTTP calls;
   - the Rulebook and target agents that should reach it.
3. Create the resource as an Action with initial kind `alwaysGet=false`. Creation returns an empty schema shell. Choose `source` correctly here; the documented update body does not include it, and `internalHandler` is read-only. A later explicit `fini_update_action(alwaysGet=true)` can convert this same resource into an Attribute.
4. Update the input schema and draft any outputs not dependent on a server-generated step ID.
5. Add external API calls with intentional `stepNumber` values, then re-list the parent chain and verify the ascending order exposed by the public API. Do not infer undocumented internal ordering state.
6. Finalize every API-backed output using:
   - external step `responseMapping`: `outputKey -> response path`;
   - Action output: `source="apiResponse"` and `path="[stepId].[outputKey]"`.
7. Preflight and confirm the real outbound effect, test the unreferenced Action with controlled inputs, then re-fetch and verify.
8. Add a Tool node referencing the exact tested Action ID to the intended Rulebook flow. Put Read/Check nodes upstream when inputs must be extracted or validated.
9. Re-fetch and review the intended intent-rule draft, then publish its explicit exact `draftVersionId` to exact agent IDs through the agent-configuration workflow. Never omit the ID or resolve a current/latest draft after approval. Then run a controlled behavioral proof if requested.

An Action without `outputSchema` returns no usable values to the Rulebook. A bare response-mapping key may resolve at runtime, but the dashboard editor needs the step-qualified path.

When the server assigns step IDs, expect a second parent-schema patch after step creation. Do not leave a bare mapping key merely to avoid that patch.

## Reachability

Action assignments are informational today. The backend stores them but does not enforce them at runtime.

Prove live reachability from:

1. a published Rulebook version;
2. a Tool node containing the exact Action ID;
3. that Rulebook's agent assignments;
4. upstream input availability;
5. downstream handling of success and failure outputs.

Do not report an Action as live merely because it exists, tests successfully, or has an Action-agent junction.

A request-scoped Generate Answer or Replay override can execute an exact draft version only when that draft belongs to a rule already published and assigned to the responding agent. Treat its Tool nodes as effectful, but do not confuse this narrow proof path with live published reachability. A brand-new unassigned rule draft cannot execute the Action through an override.

## Update

1. Read the full current Action, steps, mappings, and Rulebook references.
2. Prepare a minimal diff and omit `alwaysGet` for an ordinary Action update.
3. If changing inputs, update every upstream Rulebook value source.
4. If changing outputs, update response mappings and downstream Check/Reply/Tool nodes.
5. If reordering steps, repair all references to earlier step IDs.
6. Confirm live impact, patch, re-fetch, and run a controlled end-to-end case.

For an explicitly requested Action-to-Attribute conversion, snapshot the Action, Data Steps, masked configuration, assignments, and Rulebook references. Pre-stage the intended Attribute assignments and Rulebook cleanup. Call `fini_update_action` with `alwaysGet=true`, re-fetch the converted resource and its unchanged steps, remove obsolete on-demand Rulebook reachability, establish eligible Attribute agent/channel assignments, and test automatic context behavior. The conversion and rewiring are not atomic and have no documented rollback; use a maintenance window or a deliberately prepared replacement when an intermediate state would be unsafe. Do not recreate or delete the resource merely to change kind when this same-resource cutover is acceptable.

For a material change to an active Action, prefer building and testing an unreferenced replacement, reviewing the Rulebook diff, and cutting over the Tool node deliberately. There is no documented draft/version/rollback surface for the Action or its Data Steps.

## Test

Testing the whole Action runs every configured external API call. `POST`, `PUT`, `PATCH`, and `DELETE` steps can mutate customer systems; a misleadingly named endpoint or `GET` may still have effects.

Before execution, show:

- Action ID and name;
- ordered step IDs;
- sanitized method and host/path per step;
- input values with secrets removed;
- expected external effects;
- the controlled record or environment;
- the user's exact confirmation.

After execution, report per-step success/status and final extracted outputs after redaction. A successful HTTP call is not proof that Rulebook routing works; test the behavioral path separately when requested.

## Delete And Recover

Preflight the full Action, its Rulebook Tool-node references, Action assignments, schemas, and Data Step IDs. Capture a sanitized reconstruction snapshot.

Deleting the Action does not delete its Data Steps. Decide explicitly whether to retain or separately delete them. Remove or replace Rulebook references so published flows do not point at a missing ID.

After deletion, verify the Action is absent, inspect the former Data Steps, and re-read affected Rulebooks. Recovery recreates a new Action with new step IDs, so rebuild output mappings and Tool-node references before testing.
