# External API Call Workflows

The public API and dashboard call these external API calls; the wire object is a Data Step under `/api-function-configs`. Read the current overview and operation pages from `https://docs.usefini.com/llms.txt` before acting.

## Model The Chain

The public contract exposes `stepNumber` as chain order. List steps for the parent `toolId`, sort the returned records by `stepNumber`, and verify that observable ascending order after every topology change. Do not infer, read, or write undocumented internal ordering storage. Each step may consume inputs or extracted values from an earlier step.

Use:

- `${fieldName}` for an Action/Attribute input;
- `${stepId.responseMappingKey}` for an earlier step's extracted value;
- never `{{placeholder}}`, which is unsupported.

Choose unique, intentional step numbers. The documented update body does not include `toolId` or `stepNumber`; reparent or reorder through the documented recreate flow, then re-list the whole chain and verify its returned order. Repair every reference to the old ID. Before inserting, removing, or reordering, inspect every later URL, header, body, response mapping, and output-schema path for dependencies.

## Map Responses

`responseMapping` maps a stable output key to either:

- a response path string; or
- `{ "path": "...", "dataType": "..." }`.

Use `"."` to capture the whole response only when the downstream workflow truly needs it. Prefer narrow fields to limit exposure and payload size.

To return a mapped field from the parent Action or Attribute, set the output schema's `source` to `apiResponse` and `path` to `[stepId].[responseMappingKey]`. Validate both the extracted value and the parent output after testing.

## Create Or Update

1. Inspect the parent resource and existing ordered steps.
2. Define the method, sanitized URL, input tokens, headers/body, response mapping, and expected external effect.
3. Create with the exact parent `toolId` and intended `stepNumber`, or read the existing step and send the complete documented update payload with unchanged fields preserved.
4. Re-fetch the step, list the complete chain, and verify the ascending `stepNumber` order returned through the public API.
5. Repair parent output mappings and later-step interpolation when IDs or mapping keys change.
6. Test only after explicit outbound-effect confirmation.

Do not change a method, host, credentials, or mapped response shape without highlighting that part of the diff.

## Sensitive Values

Sensitive header/body values read as:

```json
{ "hide": true, "value": "********" }
```

Data Step reads retain these backend-masked wrappers so a fetched configuration can be safely round-tripped. The public API does not reveal the stored secret. During update:

- echo that masked wrapper unchanged to preserve the stored value;
- send a user-authorized replacement to rotate it;
- use `{ "hide": true, "value": "<new value>" }` to store a supplied value as sensitive;
- never replace the sentinel with a guessed, empty, or placeholder value;
- never print a supplied or resolved secret in output.

Keep sanitized configuration snapshots for audit and reconstruction planning only. They are not safe write sources. Before any update, fetch a fresh complete Data Step and require its current backend-masked wrappers; block the write if a secret-bearing field is missing, truncated, or no longer represented by the opaque wrapper. A recovery snapshot can record that a secret existed and where a replacement must come from, but not the secret itself.

## Test A Saved Or Inline Call

A single-step test does not persist the inline configuration, but it makes a real outbound HTTP request. Supplying a saved `id` uses its stored URL, method, headers, body, and stored secrets; inline values can override them.

Do not combine a saved step ID with an overridden host while relying on stored headers or body values. That can send resolved stored secrets to a different host.

Before testing, show:

- saved step ID or that the call is inline;
- HTTP method and sanitized host/path;
- whether stored credentials will be resolved;
- variables and payload shape with values redacted as needed;
- expected external effect and controlled target.

Obtain confirmation for the exact call. Then report `success`, `statusCode`, a redacted response summary, and `error`. Do not treat `2xx` as proof that response mappings or the complete parent chain work.

## Delete And Recover

Before deleting a step:

1. snapshot its sanitized configuration;
2. identify its parent and `stepNumber`;
3. find later `${stepId.responseMappingKey}` references;
4. find parent output-schema paths using that step ID;
5. explain the resulting chain change;
6. obtain exact-ID confirmation.

After deletion, verify the step is absent, list the remaining ordered chain, and repair broken references. Recreating the step produces a new ID, so update all step-qualified interpolation and output mappings before testing.
