The **client config table** is the centerpiece of your “one set of n8n workflows, many clients” architecture.

Instead of duplicating workflows per client (nightmare), you keep **one shared workflow** and make it behave differently based on **configuration pulled at runtime**. That configuration lives in a `clients` (client_config) table in Supabase.

---

## What the client config table is
A database table (Supabase/Postgres) where **each row = one client** you manage.

It stores:
- identifiers used to recognize the client when a webhook hits you
- credentials / API info needed to act on their behalf (or references to those secrets)
- business rules (quiet hours, follow-up cadence, staff routing)
- content/brand voice settings (templates, tone, booking CTA)

This becomes the “source of truth” for your automations.

---

## Why you need it (in plain terms)
If you don’t have this table, you end up:
- hardcoding values inside n8n nodes
- duplicating the workflow for each client
- manually changing 10 places when a client changes their booking link or phone number

With a config table:
- adding a new client is mostly “insert row + add mappings”
- updates are data changes, not workflow edits

---

## What it’s used for (concrete examples)

### 1) Multi-client webhook routing (“router” pattern)
You use one endpoint:
- `POST /webhook/leads`

Meta/TikTok/Forms send payloads that include identifiers such as:
- `form_id`
- `page_id`
- `ad_account_id`
- or sometimes you add your own hidden field

Your workflow does:
1) read `form_id`
2) query Supabase:
   - “Which client owns this form_id?”
3) load that client’s config and proceed

This is how one webhook serves all clients.

---

### 2) Provider selection and credentials
Different clients may use:
- Telnyx vs Twilio
- Zoho vs HubSpot
- different booking links

Your config determines:
- which API key to use
- which “from number” to send from
- which CRM endpoint to push to
- which message templates apply

---

### 3) Business rules (compliance + quality)
Per client you may need:
- quiet hours (don’t text after 8pm)
- max messages/day
- lead qualification questions
- handoff rules (“if lead says pricing, escalate to human”)

Those rules are stored per client so the same workflow enforces them.

---

### 4) Content tone and templates
You’ll want different tones:
- gym: motivating, concise
- trucking: direct, operational
- landscaping: friendly and local

Store:
- `brand_voice`
- `default_cta_text`
- message templates keys

Your LLM prompt builder uses these fields so messages don’t sound generic.

---

## What it typically looks like (recommended structure)

### Minimal columns (good v1)
- `id` (uuid)
- `business_name`
- `status` (active/inactive)
- `timezone`
- `website_url`
- `booking_url`
- `gbp_link`
- `config` (jsonb) ← flexible per-client settings

### Example `config` JSON (realistic)
```json
{
  "lead_sources": {
    "meta_form_id": "1234567890",
    "tiktok_form_id": "abc123"
  },
  "sms": {
    "provider": "telnyx",
    "from_number": "+13125550100",
    "quiet_hours_start": "20:00",
    "quiet_hours_end": "08:00",
    "max_msgs_per_day": 4
  },
  "crm": {
    "provider": "zoho",
    "region": "us",
    "owner_email": "owner@client.com"
  },
  "copy": {
    "brand_voice": "direct, friendly, blue-collar",
    "booking_cta": "Book your free estimate here:"
  }
}
```

This lets you evolve without schema migrations.

---

## How n8n uses it in practice
A typical lead flow starts like this:

1) Webhook receives lead payload  
2) Extract identifier (e.g., `form_id`)  
3) Query `clients` by that identifier  
4) Set variables:
- `client_id`
- `booking_url`
- `sms.from_number`
- `brand_voice`
5) Continue the workflow (send SMS, create CRM lead, schedule follow-ups)

If you later change the booking URL, you update one DB field—no workflow edits.

---

## Security note (important)
You *can* store secrets in the config JSON, but best practice is:
- store only what you need
- restrict access (service role only)
- consider encrypting sensitive fields (Telnyx/Twilio keys)

At minimum, keep Supabase internal-only and do not expose service_role keys to a frontend.

---

## Summary
**Client config table = the switchboard.**

It enables:
- one shared automation codebase
- per-client behavior
- fast onboarding
- low maintenance
- scalable operations

If you want, I can propose a finalized `clients` table schema (including recommended JSON structure) and the exact SQL query patterns you’ll use in n8n to route by Meta/TikTok/Telnyx inbound webhooks.