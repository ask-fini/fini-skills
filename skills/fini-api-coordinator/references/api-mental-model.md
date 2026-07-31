# Fini Public API Mental Model

Use this reference for broad API architecture, behavior-change triage, and end-to-end agent setup. Fetch `https://docs.usefini.com/llms.txt` before relying on route names, fields, enums, limits, or response shapes.

## API Layers

| Layer | What it owns | Behavior or state effect |
| --- | --- | --- |
| Agents | Agent identity, bare-agent lifecycle, aggregate analytics | Creation/deletion changes workspace resources; analytics is read-only. |
| Agent configuration | Prompt drafts/publishing, Rulebooks/rules/versions, tags | Published or immediate-live changes can alter answers and routing. |
| Actions and Attributes | Typed schemas, ordered external calls, assignments, Rulebook Action wiring | Attributes run before eligible message generations for assigned agents/channels; active Actions run from published Rulebook Tool nodes, with a narrow exact-draft override for already published and assigned owning rules. |
| Sources | Raw/provider/web content ingestion and refresh | Does not become answerable knowledge by itself. |
| Knowledge | Generated/direct articles, versions, drafts, folders, assignment | Changes answers after live state and agent visibility are correct. |
| Draft validation | Active baseline plus isolated and combined exact-ID artifact candidates | Creates marked test conversations/events without publishing; can still reach configured Actions. |
| Runtime | Ordinary Generate Answer turns and caller metadata | Creates conversation events and can reach configured Actions. |
| Conversations | Stored evidence, QA state, feedback notes, resolution, deletion | QA writes change review state; deletion changes stored records. |
| Testing | Replays, rule evaluation, generated tests, Test Sets, criteria, runs | Replay, rule evaluation, and live Test Set runs can execute Tool nodes or external APIs; Test Sets use current published agent behavior and do not accept draft overrides. |

## What Changes Bot Behavior

| Desired outcome | Workflow | Proof |
| --- | --- | --- |
| Create a new bot identity | Create a bare agent, then configure the required layers | Returned agent ID reappears in agent discovery. |
| Change instructions or tone | Create/review/publish an exact prompt draft | Active prompts match the reviewed version; controlled tests pass. |
| Change routing or workflow behavior | Draft/refine an intent Rulebook, then publish its explicit exact reviewed `draftVersionId`; update a Business Rule as an immediate-live non-versioned change | Published or updated rule targets the intended agent; regression cases pass. |
| Add reporting or routing categories | Create tags/tag groups, then reference them where needed | Tags re-list and Rulebook availability is correct. |
| Fetch customer/account context for eligible messages | Create an Attribute, add external calls, map output, and assign it | Assignment is visible and a controlled eligible runtime case receives expected fields. |
| Perform a customer-system operation | Create an Action, add calls, map output, wire it into an intent Rulebook draft, then publish the explicit reviewed `draftVersionId` | The published Tool node reaches the exact Action and a controlled case produces the expected effect. |
| Add facts from docs | Verify a usable tree -> Sources -> Knowledge -> publish -> assignment | Persisted article IDs exist and agent-scoped visibility is proven. |
| Refresh changed facts | Refresh Sources -> changed subset -> Knowledge drafts -> publish | Changed source IDs and exact live article versions are verified. |
| Prove unpublished behavior before launch | Persist/fetch exact candidates -> active baseline -> isolated artifact runs -> combined candidate | Exact artifact, interaction, and event IDs show request-scoped behavior; active state is unchanged. |
| Prove one scenario | Generate Answer with explicit test metadata or reviewed artifacts | Created events and answer evidence satisfy the case. |
| Prove a release did not regress | Run the same persistent Test Set before and after the published change -> poll -> compare | Published configuration IDs, terminal run IDs, case/criterion results, external effects, failures, and baseline comparison. |
| Diagnose a historical failure | Conversation evidence -> replay and/or rule evaluation | Original and diagnostic outputs remain separately identifiable. |

## Triage A Bad Answer

| Symptom | Likely owner |
| --- | --- |
| Missing or stale fact | `fini-api-sources` then `fini-api-knowledge` |
| Article exists but agent cannot use it | `fini-api-knowledge` |
| Correct facts, wrong tone or general instruction-following | `fini-api-agent-configuration` |
| Wrong escalation, routing, or workflow | `fini-api-agent-configuration` |
| Action is never invoked | `fini-api-agent-configuration` for Tool-node reachability, then `fini-api-actions-attributes` |
| Attribute data is missing or wrong | `fini-api-actions-attributes` |
| Caller already knows the needed context | `fini-api-generate-answer` metadata/user attributes |
| Need active-vs-unpublished candidate proof | `fini-api-draft-validation` |
| Need one historical what-if comparison | `fini-api-testing` replay |
| Need persistent multi-case evidence | `fini-api-testing` Test Set |
| Need dashboard-style aggregate metrics | `fini-api-agents` analytics |
| Need exact transcript or reviewer evidence | `fini-api-conversations` |
| Need one article, saved version, history, or rollback | `fini-api-knowledge` |

## Agent Setup Model

For full onboarding inside an authenticated workspace:

1. Inspect workspace and agent state.
2. Resolve an existing agent or create a bare agent.
3. Design tags and intent Rulebook drafts; treat Business Rules as immediate-live and non-versioned.
4. Create and review prompt drafts.
5. Verify a usable knowledge tree and create or initialize one when absent before source-backed generation.
6. Ingest or refresh Sources.
7. Generate, review, organize, and assign Knowledge drafts.
8. Add Attributes for required context on eligible messages.
9. Add Actions only for required customer-system operations and wire them through reviewed intent Rulebook draft Tool nodes.
10. Validate unpublished prompt, knowledge, snapshot, and eligible intent-rule candidates one dimension at a time through `fini-api-draft-validation`.
11. Publish only separately approved exact versions through their owning skills.
12. Run controlled active Generate Answer cases and retain fresh live-agent conversation IDs.
13. Create a Test Set with persistent cases and criteria, run it against current published live-agent behavior, and inspect terminal results and external effects.
14. Inspect rollout Conversations and Analytics.

Creating the bare agent does not create a workspace, install Slack, authorize providers, or configure any of the subsequent layers. Verify the live docs before claiming those setup steps are publicly automatable.

## Execution Model

- External API calls are ordered steps shared by Actions and Attributes.
- Attribute assignment changes eligible-message latency, data exposure, and potentially cost.
- Action assignment metadata is not an authorization or execution gate; published Rulebook reachability controls invocation.
- "Test Action," "Test Attribute," and "Test external API call" perform real outbound HTTP.
- Rule evaluation, replay, and Generate Answer can be effectful when reachable Tool nodes invoke Actions.
- `markAsTest: true` labels synthetic Generate Answer traffic but does not suppress events, Attributes, Actions, or external effects.
- Replay mode matters: `until` regenerates every user turn through the target linked-user event and can execute effects multiple times; `single` preserves prior event history and regenerates only the target response.
- By default, replay reuses original Attribute outputs and seeded Rulebook results unless exact prompt or rule artifacts cause the Rulebook to rerun.
- Test Set runs evaluate current published live-agent behavior and can invoke Actions or external APIs; preflight their cases and effects before starting.
- Keep stored secrets masked. Do not reconstruct, log, or echo them in reports.
