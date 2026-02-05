The **chat_history table** is the “memory + audit log” for your SMS AI nurture system. It lets you handle two-way texting reliably even though webhooks (and n8n executions) are stateless.

Without it, every inbound SMS looks like a brand-new conversation and your AI will respond inconsistently or repeat itself.

---

## What the chat_history table is
A table where each row is **one message** in a conversation:

- which client this belongs to (multi-tenant)
- which lead/customer phone number (conversation key)
- message direction / role (`user` vs `assistant`)
- message text
- timestamps
- optional metadata (provider message id, costs, handoff flags)

It is used both for:
1) **context to generate the next AI reply**
2) **compliance/audit** (what was said, when, and by whom)

---

## How it’s used in your SMS AI flow (step-by-step)

### Trigger: inbound SMS arrives (Telnyx webhook → n8n)
1) **Identify client**
   - Use “to number” (the client’s Telnyx number) or subaccount mapping
   - Query `clients` table to get `client_id` and rules

2) **Write inbound message to chat_history**
   - Insert a row:
     - `client_id`
     - `phone_number` = sender
     - `role` = `user`
     - `content` = message body

3) **Fetch recent context**
   - Query last N messages for that `client_id + phone_number`
   - Typically last 10–30 messages or last 24 hours

4) **Build the LLM prompt**
   - System instructions (business rules, tone, forbidden claims)
   - Context window (chat_history messages)
   - Current message (already included)

5) **Generate AI reply**
   - Call LLM (Groq/OpenAI)
   - Apply safety rules (length cap, disallowed content)

6) **Send reply via SMS provider**
   - Telnyx send message

7) **Write AI reply to chat_history**
   - Insert row:
     - `role` = `assistant`
     - `content` = response
     - optional: `token_count`, `provider_message_id`

This loop repeats for every incoming text. The conversation stays coherent because the table holds the history.

---

## Why it’s essential (what it enables)

### 1) “Memory” for the AI
- Avoid repeating the same question
- Remember what service they asked about
- Remember that you already gave the booking link
- Keep a consistent tone

### 2) Handoff rules
You can store flags like:
- `handoff_required = true`
- `handoff_reason = 'pricing request'`
- `opted_out = true`

Then future workflows know to stop AI responses or route to a human.

### 3) Compliance and safety
If a client ever asks:
- “What did your system text my customer?”
You can show the exact log.

Also useful for:
- STOP compliance
- complaint investigation
- debugging hallucinations

### 4) Rate limiting / loop prevention
You can query recent history to prevent “infinite loop of death”:
- if you sent 3 messages in 5 minutes and user hasn’t replied → stop and alert you

---

## Recommended schema (practical v2)
If you want to improve beyond the minimal table, add fields that make operations easier:

- `provider` (telnyx/twilio)
- `provider_message_id`
- `direction` (inbound/outbound) or keep `role`
- `conversation_id` (optional; usually phone+client is enough)
- `intent` (optional, from LLM classification)
- `handoff` boolean
- `cost_usd` numeric (optional)

Example:

```sql
create table chat_history (
  id uuid primary key default gen_random_uuid(),
  created_at timestamptz not null default now(),

  client_id uuid not null references clients(id) on delete cascade,

  phone_number text not null,
  role text not null check (role in ('user', 'assistant', 'system')),

  content text not null,

  provider text,
  provider_message_id text,

  token_count int,
  cost_usd numeric(10,4),

  meta jsonb not null default '{}'::jsonb
);

create index idx_chat_lookup
  on chat_history (client_id, phone_number, created_at desc);
```

That index is important because you’ll query “last messages” on every inbound SMS.

---

## How it connects to your other tables
- `clients` table tells you:
  - which business this conversation belongs to
  - what rules apply (quiet hours, max messages)
  - templates/booking link/brand voice
- `job_queue` (if you build it) can schedule follow-ups based on chat state
- `tasks` can be created if:
  - the AI flags handoff needed
  - lead is hot and should be called now

---

## Summary
**chat_history = your AI’s memory + your compliance log.**

It is how you:
- keep SMS conversations coherent
- prevent hallucination damage with audits and guardrails
- support human takeover
- safely scale to many clients without losing context

If you want, I can sketch the exact n8n flow for inbound SMS using this table (queries + inserts + safety checks) and a companion “follow-up scheduler” flow that uses the queue table.