# GHL Workflow — Hot Lead Escalation

This document covers the GoHighLevel automation workflow for the GHL side of hot lead escalation, working alongside the n8n logic.

---

## Overview

When n8n classifies a lead as hot/urgent, it fires a webhook back to GHL to:
1. Move the contact to the Hot Lead pipeline stage
2. Apply the hot-reply tag
3. Create an internal task for the attorney
4. Send a real-time SMS notification to the attorney
5. Log the conversation excerpt to contact notes

---

## GHL Automation 1: Hot Lead Escalation Receiver

### Trigger
- Type: Webhook (Inbound)
- Copy the generated webhook URL into n8n escalation node

### Action 1: Update Contact Tags
- Add Tag: hot-reply
- Add Tag: in-conversation (if not already present)

### Action 2: Move Pipeline Stage
- Pipeline: [Your Active Pipeline]
- Stage: Hot Lead

### Action 3: Create Internal Task
- Task Name: HOT LEAD — Follow up immediately: {{contact.first_name}} {{contact.last_name}}
- Assigned To: Lead Attorney
- Due Date: Now + 15 minutes
- Priority: High
- Description: Lead responded urgently via SMS bot. Message excerpt logged in contact notes.

### Action 4: Send Attorney SMS Alert
- To: Attorney phone number
- Message:
  HOT LEAD: {{contact.first_name}} {{contact.last_name}}
  Phone: {{contact.phone}}
  Reply: "{{custom_field.bot_last_reply_received}}"
  Time: {{now}}

### Action 5: Add Contact Note
- Note:
  [HOT LEAD ESCALATION — {{now}}]
  AI Bot detected urgent reply.
  Lead message: {{custom_field.bot_last_reply_received}}
  Urgency Score: {{custom_field.bot_urgency_score}}
  Escalated via: n8n SMS Bot workflow
  Attorney notified via SMS + task created.

### Action 6: Update Custom Fields
- bot_escalated = true
- bot_escalation_time = {{now}}
- bot_conversation_status = escalated
- attorney_notified = true

---

## GHL Automation 2: Consultation Booked Confirmation

Fires when a lead books via the calendar link sent by the bot.

### Trigger
- Type: Appointment Status Changed to Confirmed

### Actions
1. Send SMS confirmation to lead
2. Add tag: consultation-scheduled
3. Remove tag: in-conversation
4. Move pipeline stage: Consultation Booked
5. Update custom fields: consultation_booked = true
6. Wait 24h before appointment — send reminder SMS
7. Wait until 1h before appointment — send final reminder SMS

---

## GHL Automation 3: No Reply Close Sequence

Triggered by n8n webhook after the follow-up receives no response.

### Trigger
- Type: Webhook (Inbound) — fired by n8n after no-reply timeout

### Actions
1. Add tag: no-reply
2. Remove tag: in-conversation
3. Update field: bot_conversation_status = closed_no_reply
4. Move pipeline stage: No Reply

---

## GHL Automation 4: Opt-Out Handler

Fires when a lead sends STOP.

### Trigger
- Type: Inbound SMS contains keyword STOP (GHL handles compliance automatically)

### Actions
1. Add tag: opted-out
2. Update field: bot_opted_out = true
3. Update field: bot_conversation_status = opted_out
4. Remove from active sequences
5. Move pipeline stage: Opted Out

---

## Webhook URLs to Copy into n8n

| n8n Node | GHL Automation | Field in n8n |
|---|---|---|
| Hot Lead Escalation | Hot Lead Escalation Receiver | GHL Webhook URL |
| No Reply Close | No Reply Close Sequence | GHL Webhook URL |

---

## Testing the Escalation Flow

1. Create a test contact in GHL
2. Manually trigger the missed call webhook from n8n
3. Reply to the test SMS with: "I need help urgently"
4. Verify in GHL:
   - Contact tagged: hot-reply
   - Pipeline stage: Hot Lead
   - Task created and assigned to attorney
   - Attorney SMS received
   - Contact note added with message excerpt

---

*Part of Build #2 — Missed Lead SMS Re-Engagement Bot*
*Built by ausjones84*
