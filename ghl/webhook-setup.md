# GHL Webhook Setup
# Build #2: Missed Lead SMS Re-Engagement Bot

Two webhooks must be configured in GHL to power this build.

---

## Webhook 1: Missed Call Trigger

### Purpose
Fires when an inbound call goes unanswered.

### Setup Steps
1. Go to **Settings > Integrations > Webhooks** (or **Settings > API > Webhooks**)
2. Click **+ Add Webhook**
3. Configure:
   - **Name:** Build #2 - Missed Call Trigger
   - **URL:** https://your-n8n.com/webhook/ghl-missed-call
   - **Events:** Select **Missed Call** (or **Inbound Call > No Answer**)
4. Click **Save**

### Payload Fields Sent
- contact_id
- contact.firstName
- contact.phone
- locationId
- type: "missed_call"
- dateAdded

---

## Webhook 2: Inbound SMS Reply Handler

### Purpose
Fires whenever a contact sends an inbound SMS — captures their reply to the recovery message.

### Setup Steps
1. Go to **Settings > Integrations > Webhooks**
2. Click **+ Add Webhook**
3. Configure:
   - **Name:** Build #2 - Inbound SMS Reply Handler
   - **URL:** https://your-n8n.com/webhook/ghl-inbound-sms
   - **Events:** Select **Inbound Message** or **SMS Received**
4. Click **Save**

### Payload Fields Sent
- contact_id
- contact.firstName
- contact.phone
- body (the SMS message text)
- conversationId
- locationId
- type: "SMS"

---

## Webhook 3 (Optional): Abandoned Form

### Purpose
Fires when a lead starts filling out a contact form but doesn't submit.

### Setup Step
This requires GHL's native automation, not a direct webhook:
1. Go to **Automation > Workflows**
2. Create trigger: **Form Submitted** with filter: specific form + only partial fills
3. Alternative: Use a time-delay check — if form started but not completed after 10 minutes

OR use a JavaScript event on your website that fires to the n8n webhook when a user begins filling the form and navigates away.

---

## Testing Your Webhooks

### Test Missed Call Webhook
Use curl or Postman:

```bash
curl -X POST https://your-n8n.com/webhook/ghl-missed-call \
  -H "Content-Type: application/json" \
  -d '{
    "contact_id": "test123",
    "contact": { "firstName": "Sarah", "phone": "+15551234567" },
    "locationId": "YOUR_LOCATION_ID",
    "type": "missed_call",
    "dateAdded": "2026-04-27T10:00:00Z"
  }'
```

Expected: SMS sends to +15551234567, GHL contact tagged missed-call.

### Test Inbound SMS Webhook
```bash
curl -X POST https://your-n8n.com/webhook/ghl-inbound-sms \
  -H "Content-Type: application/json" \
  -d '{
    "contact_id": "test123",
    "contact": { "firstName": "Sarah", "phone": "+15551234567" },
    "body": "Yes I was in a car accident last week",
    "conversationId": "conv123",
    "type": "SMS"
  }'
```

Expected: OpenAI generates reply, GHL SMS sent, intent classified.

---

## GHL Custom Fields for Build #2

Create these in Settings > Custom Fields > Contacts:

| Field Name | API Key | Type |
|-----------|---------|------|
| Missed Call Recovery Status | missed_call_recovery_status | Dropdown: sms_sent, replied, booked, hot_escalated, no_reply, opted_out |
| Missed Call At | missed_call_at | Date/Time |
| Recovery Conversation Log | recovery_conversation_log | Text Area |
| Intent Classification | intent_classification | Dropdown: book, escalate, continue, stop |

---

## GHL Pipeline Stage

Add this stage to your intake pipeline:
- **Stage Name:** Missed Call Recovery
- Use this stage ID as GHL_MISSED_CALL_STAGE_ID in your .env
