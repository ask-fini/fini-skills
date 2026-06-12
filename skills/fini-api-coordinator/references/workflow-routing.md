# Workflow Routing

Use this reference when the user describes a Fini API goal and the agent needs to choose the right skill or sequence.

## Routing Rules

| Customer phrase | Likely intent | Skill | First docs lookup |
| --- | --- | --- | --- |
| "Export conversations from last week" | Conversation list/export | `fini-api-conversations` | `llms.txt` -> List conversations |
| "Delete these conversations" | Destructive conversation cleanup | `fini-api-conversations` | `llms.txt` -> Delete/Bulk delete conversations |
| "Send a message to my Fini bot" | Runtime answer generation | `fini-api-generate-answer` | `llms.txt` -> Generate Answer |
| "Test if the bot answers this correctly" | Controlled Generate Answer test | `fini-api-generate-answer` | `llms.txt` -> Generate Answer + List agents |
| "Import our docs website" | Web source ingestion | `fini-api-sources` | `llms.txt` -> Sources, Crawl links, Ingest sources |
| "Sync Zendesk/Notion/Confluence" | Provider discovery, registration, ingestion | `fini-api-sources` | `llms.txt` -> Provider resources + Ingest sources |
| "Refresh our existing docs" | Refresh sources, detect changed sources | `fini-api-sources` then `fini-api-knowledge` | `llms.txt` -> Refresh sources + Bulk generate knowledge |
| "Turn these docs into KB articles" | Source-backed generation | `fini-api-knowledge` | `llms.txt` -> Knowledge + Bulk generate knowledge |
| "Create a KB draft from this feedback" | Single knowledge generation | `fini-api-knowledge` | `llms.txt` -> Generate knowledge |
| "Publish this draft" | Live knowledge change | `fini-api-knowledge` | `llms.txt` -> Publish article draft |
| "Assign this folder to an agent" | Agent visibility / retrieval scope | `fini-api-knowledge` | `llms.txt` -> Assign knowledge to agents |

## Multi-Step Recipes

### First-Time Web Knowledge Import

1. `fini-api-sources`: optionally crawl seed URLs.
2. `fini-api-sources`: ingest selected URLs and poll source completion.
3. `fini-api-knowledge`: bulk-generate draft knowledge from source IDs.
4. `fini-api-knowledge`: poll jobs and review/publish.
5. `fini-api-knowledge`: organize into folders and assign to agents.
6. `fini-api-generate-answer`: test the intended agent if behavioral proof is needed.

### Refresh Existing Knowledge From Changed Sources

1. `fini-api-sources`: refresh source IDs.
2. `fini-api-sources`: poll completion.
3. `fini-api-sources`: filter changed linked sources.
4. `fini-api-knowledge`: bulk-generate updates as drafts by default.
5. `fini-api-knowledge`: poll jobs, review, publish, and verify assignment.

### Conversation Evidence To Knowledge

1. `fini-api-conversations`: fetch the conversation or export a narrow sample.
2. Extract the exact customer feedback/evidence, not the whole transcript by default.
3. `fini-api-knowledge`: generate draft knowledge from the evidence.
4. Publish and assign only after confirmation.

## Clarify Before Acting

Ask one concise question when the route is ambiguous and the wrong choice could mutate live state:

- "Do you want this as draft/review first, or should it go live immediately?"
- "Is this a test agent or a production agent?"
- "Do you want to import sources only, or also generate knowledge from them?"
- "Should deleting sources also delete linked articles?"
- "Which agent should this folder become visible to?"

## Do Not Route To Fini API Skills

- Internal production debugging that needs logs, databases, or Supabase.
- Frontend widget/embed changes.
- General marketing positioning.
- Generic HTTP client setup with no Fini-specific workflow.
- User requests that only need the docs index copied or summarized.
