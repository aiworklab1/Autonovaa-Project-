# PlacidWay Knowledge Chatbot

**Stack:** n8n · Chatwoot · WhatsApp · Supabase Vector Store · PostgreSQL Chat Memory · OpenAI

## Overview

This project connects Chatwoot conversation events to an n8n AI workflow. The workflow maps incoming message and sender details, routes by message type, uses an OpenAI chat model with PostgreSQL-backed conversation memory, retrieves relevant information from the Supabase `documents` vector table, and sends the generated answer back to the relevant Chatwoot conversation.

A separate PostgreSQL query upserts the WhatsApp source ID and last-message timestamp into `whatsapp_users`.

## Workflow

1. **Webhook** — receives POST events on the `chatwoot` path.
2. **Edit Fields** — extracts account ID, conversation ID, message type, sender name, phone number, and message content.
3. **Switch** — routes events by message type; the incoming branch continues to the AI Agent.
4. **AI Agent** — generates a concise answer using the workflow's response instructions.
5. **OpenAI Chat Model** — configured as `gpt-5-mini`.
6. **Postgres Chat Memory** — uses the sender phone number as its custom session key.
7. **Supabase Vector Store** — exposes the `documents` table in retrieve-as-tool mode.
8. **OpenAI Embeddings** — supplies embeddings for vector retrieval.
9. **Send Message** — posts the AI output to the matching Chatwoot conversation as an outgoing, non-private text message.
10. **Execute a SQL query** — upserts contact activity into `whatsapp_users`.

## Knowledge data

The provided Supabase screenshot shows a `public.documents` table with `id`, `content`, `metadata`, and vector embedding columns, and 57 records at the time of capture. The provided workflow export does not include the ingestion workflow or the original knowledge source files.

## Response style configured in the workflow

- Lead with the direct answer.
- Use 1–3 short sentences or up to 3 bullets (about 60 words before the source line).
- Answer only what was asked.
- Add a source line in the format `Source: <title> - <url>` when a source is available.
- Avoid filler, emojis, and closing questions.
- Longer answers are allowed when the user asks for detail or a comparison.

## Import and configuration

1. Import the workflow JSON into the target n8n instance.
2. Reconnect OpenAI, Supabase, and PostgreSQL credentials in the target instance; credential IDs in an export may not be portable.
3. Configure Chatwoot webhook events to call the n8n production webhook URL.
4. Confirm payload fields and account/conversation IDs against actual Chatwoot webhook events.
5. Confirm the Supabase `documents` table, vector dimensions, and embedding model are compatible.
6. Confirm PostgreSQL chat-memory configuration and session-key consistency.
7. Test incoming and outgoing events, duplicate delivery, empty content, and invalid payloads.
8. Confirm the current WhatsApp message-window and template rules for the connected channel.
9. Add monitoring and an error workflow before relying on the chatbot in production.

## Security and production checklist

- [ ] **Rotate the Chatwoot API token in the exported workflow.** The provided JSON contains a literal access token. Revoke/rotate it if the file was shared outside trusted operators; store the replacement securely.
- [ ] Replace SQL string interpolation with parameterized queries and validate webhook inputs.
- [ ] Restrict access to n8n executions, Chatwoot conversations, Supabase, PostgreSQL, and backups.
- [ ] Define retention/deletion rules for personal and health-related conversation data.
- [ ] Verify outgoing Chatwoot events cannot trigger duplicate AI responses.
- [ ] Verify human takeover works and automation does not interfere with manually handled conversations.
- [ ] Implement and test 24-hour WhatsApp-window handling. The workflow has a sticky note about this, but the supplied JSON does not show a connected node that fully enforces it.
- [ ] Add retry/error handling, alerting, and fallback behavior for model, database, and webhook failures.
- [ ] Validate that cited sources actually support each answer and test retrieval misses.
- [ ] Back up workflow, knowledge sources, and database configuration securely.

## Acceptance tests

| Scenario | Expected result |
|---|---|
| Knowledge question | Concise answer based on relevant retrieved content, with source |
| Pricing question | Direct answer with one relevant price and source |
| Medical safety question | No guarantee; recommend qualified clinical review |
| Follow-up message | Conversation memory remains scoped to the correct sender |
| Outgoing Chatwoot event | No unwanted duplicate AI response |
| Human takeover | Human can handle conversation without automation interference |
| Invalid payload | Safe failure or fallback, visible to operators |
| Message-window boundary | Reply method follows current WhatsApp/Meta rules |

## Evidence supplied

Screenshots supplied with this project show:
- Successful n8n execution history.
- Supabase `documents` records containing content, metadata, and embeddings.
- WhatsApp test exchanges about treatment pricing and program inclusions.
- Chatwoot inbox with an AI-handled conversation and a human takeover option.

These screenshots reflect observed test activity only; they do not establish uptime, medical accuracy, security compliance, or production readiness.

## Files

- `PlacidWay_Knowledge_Chatbot_Technical_Documentation.pdf` — detailed project documentation with architecture, screenshots, configuration notes, security checks, and acceptance tests.
- `README.md` — quick project overview and operational checklist.
- `1-PlacidWay Knowledge Chatbot (N8N+SUPABASE+CHATWOOT).json` — original n8n workflow export; treat as sensitive until its embedded API token is rotated.

## Support / ownership notes

Before handing this project to another operator, provide credentials through a secure secret-sharing method rather than embedding them in chat, email, or public documentation. Verify all environment-specific URLs and credentials in the destination environment.
