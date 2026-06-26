# Intake And State

Use this reference when running Slack-led onboarding. Keep the conversation easy for a client: offer a single upload/drop mode first, then a step-by-step alternative.

## Intake Prompt

Ask:

```markdown
I can do this two ways:

1. Drop everything at once: a Markdown/PDF/doc bundle, help center URL, SOPs, behavior notes, top questions, examples, and launch constraints.
2. Step by step: I will ask one short section at a time and keep track of what is complete.

Which do you prefer?
```

## All-At-Once Request

Ask the client to include as much as they have:

- what the bot should handle
- what the bot should never handle
- escalation and handoff rules
- tone, persona, and support style
- help center URL
- connected sources already authorized in Fini
- SOPs, PDFs, docs, or Markdown files
- existing prompt or behavior notes
- top customer questions
- bad-answer examples or past ticket examples
- launch date, review owner, and go-live constraints

Do not block if they have only a subset. Start from available inputs and mark missing pieces as open questions.

## Step-By-Step Sections

Ask in this order:

1. Scope: what should the bot handle?
2. Boundaries: what should it never handle?
3. Escalation: when should it hand off?
4. Voice: tone, personality, and formatting preferences.
5. Sources: help center, SOPs, provider docs, PDFs, Markdown, or URLs.
6. Questions: top real customer questions and expected answers.
7. Evidence: bad answers, past tickets, known risk areas.
8. Launch: approval owner, test expectations, go-live constraints.

## Onboarding State

Maintain this lightweight state in the conversation:

```markdown
State:
- workspace/auth: [confirmed / missing / blocked]
- target agent: [unknown / resolved name / botId]
- onboarding type: [fresh / expansion / targeted]
- intake mode: [all-at-once / step-by-step]
- scope collected: [yes / no]
- boundaries collected: [yes / no]
- escalation collected: [yes / no]
- voice collected: [yes / no]
- sources collected: [yes / no]
- questions collected: [yes / no]
- evidence collected: [yes / no]
- launch constraints collected: [yes / no]
- sources processed: [not started / queued / complete / failed]
- knowledge status: [not started / drafts / published / assigned]
- config status: [not started / proposed / changed / verified]
- tests status: [not started / running / pass / fail]
- approvals pending: [list]
```

Update state after each phase. If the conversation is interrupted, resume from the first incomplete state item.

## Approval Language

Use plain approval gates:

- "I can prepare this as a draft now."
- "Creating drafts in Fini is a write. Should I proceed?"
- "Publishing affects live bot answers. Please confirm before I publish."
- "Prompt and rule updates can change production behavior. Please confirm the target agent and change."

Do not bundle draft creation, publishing, prompt updates, and rule publishing into one approval.

## Minimal Test Suite

Create 4-8 tests:

- 2 common support questions
- 1 out-of-scope or escalation case
- 1 tone/personality case
- 1 source-specific factual question
- 1 edge case from past tickets or bad-answer examples when available
- 1 metadata/user-attribute case only if runtime personalization is part of onboarding

For each test, track:

```markdown
Question:
Expected behavior:
Actual answer:
Evidence/events:
Pass/fail:
Failure type:
Next fix:
```
