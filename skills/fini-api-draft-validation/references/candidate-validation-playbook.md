# Candidate Validation Playbook

Use this reference when preparing exact unpublished Fini artifacts for a controlled `fini_generate_answer` comparison.

## Contents

- [Evidence Model](#evidence-model)
- [Acquire Exact Candidates](#acquire-exact-candidates)
- [Run The Isolation Ladder](#run-the-isolation-ladder)
- [Compare Like For Like](#compare-like-for-like)
- [Choose Another Test Surface](#choose-another-test-surface)
- [Result Template](#result-template)

## Evidence Model

Keep four states separate:

1. **Active baseline:** a fresh Generate Answer turn with no artifact overrides.
2. **Isolated candidate:** the same case with exactly one candidate dimension overridden.
3. **Combined candidate:** the same case with the reviewed prompt, knowledge, snapshot, and eligible intent-rule version IDs intentionally combined.
4. **Active proof after publish:** a separate future run without overrides, only when publication was explicitly requested elsewhere.

Candidate runs create stored conversations and events but do not make their artifacts active. Preserve each configuration and interaction separately so one run cannot contaminate another.

## Acquire Exact Candidates

| Dimension | Owning workflow | Required evidence before Generate Answer | Artifact field |
| --- | --- | --- | --- |
| Prompt | `fini-api-agent-configuration` | Persisted draft; exact fetched lifecycle `versionId`; complete, non-truncated arrays; intended workspace and agent | `promptVersionId` |
| Knowledge articles | `fini-api-knowledge` | Draft-intent create/generation; terminal job; concrete persisted draft IDs; exact fetched drafts and provenance | `knowledgeArticleDraftIds` |
| Knowledge snapshot | `fini-api-knowledge` | Exact workspace-owned snapshot ID from a documented response/read | `knowledgeSnapshotId` |
| Intent Rulebook | `fini-api-agent-configuration` | Actual kind is `intent`; exact fetched draft/version ID; owning rule already published and assigned to target bot | `ruleVersionIds` |

Store candidate content through the owning APIs. In `fini_generate_answer`, send IDs only:

```text
artifacts.promptVersionId
artifacts.knowledgeArticleDraftIds
artifacts.knowledgeSnapshotId
artifacts.ruleVersionIds
```

Do not inline prompt text, prompt arrays, KB article bodies, candidate instructions, input schemas, or `flowConfig` in `artifacts`.

Block rather than guess when:

- a knowledge job is still pending, failed, or completed without a persisted ID;
- an exact prompt version cannot be fetched completely;
- a snapshot is described as “current” or “latest” without an exact ID;
- a rule is a Business Rule, a brand-new unassigned intent draft, or has an unverified version ID;
- any artifact belongs to another workspace or agent context.

## Run The Isolation Ladder

Use fresh conversations throughout. Through MCP, set the connector's `markAsTest: true`; direct REST exposes no top-level field by that name:

1. Active baseline: no `artifacts`.
2. Prompt-only candidate, when present.
3. Knowledge-draft-only candidate, when present.
4. Snapshot-only candidate, when present.
5. Eligible intent-rule-only candidate, when present.
6. Combined candidate using only the isolated, reviewed IDs.

Omit dimensions that are not part of the intended change. Verify current docs before combining knowledge snapshots and draft article IDs; do not assume undocumented replacement or merge semantics.

If one dimension fails, stop and classify it before adding more overrides. A combined run is useful only after isolated runs make attribution possible.

## Compare Like For Like

Hold constant:

- workspace and `botId`;
- message text and `role=user`;
- channel, metadata, and caller user attributes;
- controlled user/account identifiers;
- expected behavior and scoring rubric;
- prior-context shape;
- locale, timezone, and other request context;
- known reachable Action and configured Attribute conditions.

Use independent but equivalent threads for multi-turn cases. Do not test a candidate by continuing the active baseline conversation because the earlier answer becomes part of candidate context.

Capture:

- exact artifact set or explicit “active/no artifacts”;
- MCP `markAsTest: true` when the connector is used;
- interaction and created-event IDs;
- final answer and reply type;
- used knowledge/evidence;
- rule path and tags when returned;
- configured Attribute and Action effects;
- latency/errors when relevant;
- material difference from baseline and pass/fail reason.

## Choose Another Test Surface

- **Ordinary Generate Answer:** use `fini-api-generate-answer` when the artifact set is already selected and no baseline/candidate methodology is needed.
- **Replay:** use one when the question is specifically how a known stored response changes under preserved historical context. A replay creates a separate replay conversation and has its own reuse/rerun semantics.
- **Rule evaluation:** use it to inspect a saved Rulebook's node path for supplied synthetic or conversation context. It does not prove the full prompt, retrieval, and answer runtime.
- **Test Suite:** use saved cases/scenarios, criteria groups, and asynchronous run history for multi-case regression. Exact prompt versions, rule drafts, and article drafts are selected on the run; Actions use saved mocks and Attributes use captured context.

The MCP connector's `markAsTest: true` is a traffic marker, not a dry-run switch. Generate Answer still writes conversation/events, configured Attributes can call external systems, and reachable Actions can mutate customer systems. Test Suite uses saved Attribute context and Action mocks; its results do not prove real provider calls.

## Result Template

```text
Draft validation
- Workspace / agent:
- Controlled case and expected behavior:
- Baseline: [interaction ID, event IDs, no artifacts]
- Prompt candidate: [version ID, interaction/event IDs, result or not tested]
- Knowledge candidate: [draft IDs, interaction/event IDs, result or not tested]
- Snapshot candidate: [snapshot ID, interaction/event IDs, result or not tested]
- Rule candidate: [version IDs, eligibility proof, interaction/event IDs, result or not tested]
- Combined candidate: [exact ID set, interaction/event IDs, result or not run]
- Like-for-like controls:
- Attribute/Action effects:
- Verdict and evidence:
- Publication: not performed / separately requested elsewhere
- Remaining uncertainty:
```
