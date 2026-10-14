# Intake And State

Use this reference while running Slack-led onboarding. Keep the client interaction simple while preserving enough state to resume safely.

## Contents

- [Intake Mode](#intake-mode)
- [All-At-Once Inputs](#all-at-once-inputs)
- [Step-By-Step Order](#step-by-step-order)
- [Onboarding State](#onboarding-state)
- [Mutation Language](#mutation-language)
- [Representative Cases](#representative-cases)

## Intake Mode

Ask:

```markdown
I can do this two ways:

1. Drop everything at once: docs, help center, SOPs, behavior notes, automation and customer-context needs, example tickets, tests, and launch constraints.
2. Step by step: I will ask one short section at a time and track what is complete.

Which do you prefer?
```

## All-At-Once Inputs

Accept whatever the client has:

- what the bot should and should not handle
- escalation and handoff rules
- tone, persona, and response format
- existing or desired agent name
- Slack/channel installation status
- help center, SOPs, PDFs, Markdown, provider content, and URLs
- existing prompt or behavior notes
- customer context needed before replies
- customer-system operations the bot should perform
- customer API ownership, auth readiness, idempotency, and safe test targets
- top questions and expected answers
- bad-answer examples and historical conversations
- launch date, approval owner, regression requirements, and go-live constraints

Do not block because inputs are incomplete. Start from what exists and record missing prerequisites.

## Step-By-Step Order

1. Scope: what should the bot handle?
2. Boundaries: what should it never handle?
3. Agent and channel: existing agent or desired name; Slack installation status.
4. Escalation: when and how should it hand off?
5. Voice: tone, personality, and formatting.
6. Sources: help center, providers, SOPs, PDFs, Markdown, and URLs.
7. Context: what customer/account fields should be fetched automatically for eligible messages?
8. Operations: what customer-system Actions should the bot perform?
9. Questions: top cases and expected outcomes.
10. Evidence: bad answers, conversations, known risks, and regressions.
11. Launch: owner, test criteria, safe integration-test targets, and go-live constraints.

## Onboarding State

Maintain:

```markdown
State:
- workspace/auth: [confirmed / missing / blocked]
- agent: [unknown / existing name+botId / create requested / created name+botId]
- Slack/channel installation: [ready / user action / blocked]
- onboarding type: [fresh / expansion / targeted]
- intake mode: [all-at-once / step-by-step]
- scope/boundaries/escalation/voice: [complete / missing items]
- sources: [collected / processing / complete / failed]
- knowledge: [not started / drafts / published / assigned / blocked]
- configuration: [not started / draft / published / verified]
- Attributes: [not needed / requirements / configured / assigned / verified]
- Actions: [not needed / requirements / configured / Rulebook-wired / verified]
- customer API prerequisites: [not needed / ready / user action / blocked]
- draft validation: [not needed / baseline / isolated candidates / combined candidate / blocked]
- active smoke: [not started / marked test interaction+event IDs / passed / failed]
- Test Suite: [not started / cases ready / criteria ready / run ID+case statuses]
- blocking failures: [list / none]
- approvals or user actions pending: [list / none]
```

Resume from the first incomplete dependency, not merely the next numbered phase.

## Mutation Language

Use plain, target-specific language when intent is not yet clear:

- "I can create a bare Fini agent named X; Slack installation remains separate."
- "I can save this as a reviewable draft or publish it immediately. Which do you want?"
- "Assigning this Attribute makes it run for eligible message generations for agent/channel X."
- "This Action test sends a real request to Y with operation Z."
- "This Generate Answer check can invoke Action Y. The Test Suite regression uses saved mocks and does not prove the real provider operation."
- "Deleting or rolling back this exact resource changes live workspace state."

Do not repeat an approval question when the client already gave explicit intent and an exact target.

## Representative Cases

Define 4-8 cases covering the relevant subset:

- two common support questions
- one source-specific factual question
- one out-of-scope or escalation case
- one tone/personality case
- one caller-metadata or eligible-message Attribute case
- Action success and denial/failure cases
- one historical edge case

Use exact draft validation before publication when needed. After publication, preserve representative cases for a persistent Test Suite. Test Suite runs simulate Actions with saved mocks and reuse captured Attributes; verify those fixtures and inputs. For each case, track:

```markdown
Case:
Expected behavior:
Source conversation/case ID:
Generated result:
Criteria and execution evidence:
Recorded/custom Action mocks:
Pass/fail/error:
Failure owner:
```

Then create/update the Test Suite, attach launch criteria/groups, preview readiness, start the requested simulated run, and record:

```markdown
Case/group IDs:
Criterion IDs:
Run ID:
Terminal status:
Blocking failures:
Baseline or prior run:
Go-live disposition:
```

Test Suite supports captured published or selected draft configuration and simulated Actions. A go-live gate must use the intended published configuration; a draft pass is pre-publish evidence. Real Generate Answer, standalone Replay, rule evaluation, and integration tests retain their own external-effect preflight. Replay until/single mode remains relevant to historical diagnosis.
