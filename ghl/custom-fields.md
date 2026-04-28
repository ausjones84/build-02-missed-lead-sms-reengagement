# GHL Custom Fields — Missed Lead SMS Re-Engagement Bot

This document lists all custom contact and opportunity fields to create in GoHighLevel for this build.

---

## How to Add Custom Fields in GHL

1. Go to **Settings** > **Custom Fields**
2. Select **Contact** or **Opportunity** as the field type
3. Click **+ Add Field**
4. Set the field name, type, and key exactly as listed below
5. Save each field before moving to the next

---

## Contact-Level Custom Fields

These fields are stored on the contact record and updated throughout the bot conversation.

| Field Label | Field Key | Type | Description |
|---|---|---|---|
| Bot Conversation Status | bot_conversation_status | Dropdown | Tracks current state of the SMS conversation |
| Bot Last Message Sent | bot_last_message_sent | Text | Text of the last outbound SMS message |
| Bot Last Reply Received | bot_last_reply_received | Text | Text of the last inbound SMS reply |
| Bot Intent | bot_intent | Dropdown | Classified intent from last reply |
| Bot Urgency Score | bot_urgency_score | Number | 0–3 urgency score from AI classification |
| Bot Escalated | bot_escalated | Checkbox | Whether this contact was escalated to attorney |
| Bot Escalation Time | bot_escalation_time | Date/Time | Timestamp of escalation event |
| Bot Conversation ID | bot_conversation_id | Text | Unique ID linking to LangChain memory store |
| Missed Call Time | missed_call_time | Date/Time | Timestamp of the missed call that triggered the bot |
| Trigger Source | trigger_source | Dropdown | What triggered the bot: missed_call or abandoned_form |
| Initial SMS Sent | initial_sms_sent | Checkbox | Whether the first outbound SMS was successfully sent |
| Initial SMS Time | initial_sms_time | Date/Time | Timestamp of first SMS sent |
| Follow-Up SMS Sent | followup_sms_sent | Checkbox | Whether the 15-min follow-up was sent |
| Calendar Link Sent | calendar_link_sent | Checkbox | Whether a booking link was texted to the lead |
| Bot Opted Out | bot_opted_out | Checkbox | Lead sent STOP — opted out of SMS |

### Dropdown Options: bot_conversation_status
- pending_initial_sms
- initial_sms_sent
- awaiting_reply
- in_conversation
- follow_up_sent
- booked
- escalated
- not_interested
- opted_out
- closed_no_reply

### Dropdown Options: bot_intent
- book_now
- hot_lead
- needs_more_info
- not_interested
- no_reply
- unknown

### Dropdown Options: trigger_source
- missed_call
- abandoned_form

---

## Opportunity-Level Custom Fields

These are attached to the deal/opportunity record when a lead is active.

| Field Label | Field Key | Type | Description |
|---|---|---|---|
| Case Type | case_type | Dropdown | Type of legal case identified |
| Lead Source Bot | lead_source_bot | Text | Always set to: missed-lead-sms-bot |
| Consultation Booked | consultation_booked | Checkbox | Whether consultation was booked via bot |
| Consultation Link Sent | consultation_link_sent | Checkbox | Whether calendar link was sent |
| Hot Lead Flag | hot_lead_flag | Checkbox | True if escalated as hot lead |
| Attorney Notified | attorney_notified | Checkbox | Whether attorney alert was triggered |

### Dropdown Options: case_type
- personal_injury
- car_accident
- workers_compensation
- medical_malpractice
- slip_and_fall
- wrongful_death
- criminal_defense
- family_law
- other
- unknown

---

## GHL Pipeline Stages for This Build

Create or verify the following stages exist in your active pipeline:

| Stage Name | Stage Purpose |
|---|---|
| New Lead | Default entry stage for all new contacts |
| SMS Bot Active | Contact is currently in the bot conversation |
| Hot Lead | Urgent lead — escalated to attorney |
| Consultation Booked | Lead booked a free consultation |
| Not Interested | Lead declined — do not re-engage |
| No Reply | Lead did not respond after follow-up |
| Opted Out | Lead sent STOP |

---

## GHL Smart List / Segment Setup

Create the following saved Smart Lists for reporting and monitoring:

**1. Active Bot Conversations**
- Filter: bot_conversation_status = in_conversation OR awaiting_reply

**2. Hot Leads — Needs Follow-Up**
- Filter: bot_escalated = true AND consultation_booked = false

**3. No Reply — Re-Engage Candidates**
- Filter: bot_conversation_status = closed_no_reply AND created within last 30 days

**4. Opted Out**
- Filter: bot_opted_out = true

---

## Notes

- All field keys must match exactly what is referenced in the n8n workflows
- The `bot_conversation_id` field is critical — it links GHL contacts to LangChain memory
- Run the `custom-fields-check` step in the setup guide to verify all fields are correctly created
- Escalation and booking actions in n8n update these fields automatically via GHL API

---

*Part of Build #2 — Missed Lead SMS Re-Engagement Bot*
*Built by ausjones84*
