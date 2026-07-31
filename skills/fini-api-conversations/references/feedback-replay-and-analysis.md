# Feedback, Replay, And Analysis Handoffs

Use this reference when conversation evidence leads to event feedback, replay/regression testing, rule evaluation, or agent analytics. Fetch the current docs first because fields and supported test artifacts can change.

## Keep Feedback Writes Separate

All three writes target an exact conversation ID and event ID, but they represent different state:

| Intent | Field | Meaning |
| --- | --- | --- |
| Rate the answer | `approved` | `true` thumbs up, `false` thumbs down, `null` clears |
| Explain the review | `feedback` | Teammate note; an empty value clears that event's note |
| Mark a handled or reopened negative issue | `resolved` | `true` means handled; `false` means unresolved/reopened |

Recommended sequence for a bad answer:

1. Fetch the conversation and identify the exact Fini-authored response event.
2. Set `approved: false`.
3. Add a concise feedback note if qualitative evidence is useful.
4. Diagnose and fix the issue through the owning workflow.
5. Retest.
6. Mark that event `resolved: true` only after the issue has been handled.
7. Re-fetch and report the event rating, note, event resolution, and recalculated conversation resolution separately.

Resolution semantics:

- no negatively rated Fini events -> conversation `resolved: null`
- all negatively rated Fini events handled -> conversation `resolved: true`
- at least one unresolved negative Fini event -> conversation `resolved: false`

Use `resolved: false` when a previously handled negative issue needs to be reopened. Clearing a note does not clear the rating. Clearing the rating can change aggregate resolution independently of the note.

## Replay And Rule-Evaluation Handoff

Use `fini-api-testing` for replay creation and comparison:

1. Preserve the source conversation ID and exact Fini response event ID.
2. Confirm the response has a linked user event.
3. Choose `single` to preserve prior history and regenerate only the target response. Choose `until` to regenerate every user turn through the user event linked to the target response.
4. Supply model or artifact overrides only when they came from validated Fini data. By default replay reuses original Attribute outputs and Rulebook results; an exact rule artifact can trigger Rulebook rerun behavior.
5. Compare the original event with the new replay events and keep both conversation IDs in the result.
6. Use an eligible source conversation as a persistent Test Set case when appropriate. Validate any unpublished exact-ID fix through `fini-api-draft-validation`, publish it separately, then run the Test Set against current live-agent behavior. Replay conversations themselves remain ineligible Test Set inputs.

Also hand conversation-rule evaluation to `fini-api-testing`. Evaluation is a write-scoped test and can execute rule nodes; do not present it as a passive read.

## Analytics Handoff

Use `fini-api-agents` when the question is aggregate:

- performance summaries or comparisons
- resolution/escalation/response-time/CSAT trends
- channel or status breakdowns
- knowledge usage
- rule performance
- escalation reasons

Carry over the agent ID, exact time window, timezone, and filters. Use conversation export afterward only to inspect representative evidence behind the aggregate result.

## Handoff Template

```text
Quality handoff
- Agent:
- Time window / filters:
- Source conversation:
- Target Fini response event:
- Rating / note / resolution:
- Replay mode or Test Set goal:
- Analytics metric or section:
- Owning skill:
- Evidence to verify afterward:
```
