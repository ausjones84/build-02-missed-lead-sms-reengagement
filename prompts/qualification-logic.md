# Qualification Logic — Missed Lead SMS Re-Engagement Bot

## Overview

After each incoming SMS reply, n8n runs an intent classification step using OpenAI. This document defines the scoring rules, intent categories, and routing logic.

---

## Intent Categories

### 1. book_now
**Trigger signals:**
- Lead explicitly asks to schedule, book, or set up a meeting
- Phrases: "book", "schedule", "appointment", "calendar", "when are you available"

**Action:**
- Send GHL calendar booking link via SMS
- Create task in GHL: "Consultation Booked"
- Tag contact: consultation-scheduled
- Move to pipeline stage: Booked

---

### 2. hot_lead
**Trigger signals:**
- Urgent language: "I need help NOW", "emergency", "ASAP"
- Serious injury or major legal matter implied
- Phrases: "my wife was hurt", "I was in a bad accident", "I need a lawyer today"

**Action:**
- Immediately escalate to attorney via Slack + SMS
- Tag contact: hot-reply
- Move to pipeline stage: Hot Lead
- Log full conversation thread to GHL contact notes

**Escalation Message Format:**
@attorney HOT REPLY from [Name] — [Phone]
Message: "[excerpt]"
Time: [timestamp]

---

### 3. needs_more_info
**Trigger signals:**
- Lead replies but intent is unclear
- Asks about firm, process, or fees
- Phrases: "what do you handle", "how does it work", "is there a fee"

**Action:**
- Continue AI conversation with 1 follow-up question
- Do NOT send calendar link yet
- Tag contact: in-conversation

---

### 4. not_interested
**Trigger signals:**
- Explicit disinterest: "no thanks", "wrong number", "I already hired someone", "STOP"

**Action:**
- Send polite closing message
- Tag contact: not-interested
- Unsubscribe if "STOP" detected (legally required)
- End conversation — do NOT follow up

---

### 5. no_reply
**Trigger signals:**
- No inbound SMS within 15 minutes of initial message

**Action:**
- Send one follow-up, wait 24h
- If still no reply: tag no-reply, close sequence
- Max 2 outbound messages without a reply

---

## Scoring Rubric — JSON Output Schema

{
  "intent": "book_now | hot_lead | needs_more_info | not_interested | no_reply",
  "confidence": 0.0,
  "urgency_score": 0,
  "suggested_response": "",
  "escalate": false,
  "tags": [],
  "pipeline_stage": ""
}

Urgency Score:
- 0: No urgency / not interested
- 1: Low (general inquiry)
- 2: Medium (clear legal matter, not urgent)
- 3: High (urgent injury, strong buying intent)

Escalate = true when urgency_score >= 3 OR intent is hot_lead

---

## Classification Prompt (n8n OpenAI Intent node)

You are a lead qualification assistant for a law firm SMS bot.

Analyze the following SMS reply and return JSON:
- intent: [book_now, hot_lead, needs_more_info, not_interested, no_reply]
- confidence: float 0.0–1.0
- urgency_score: integer 0–3
- suggested_response: warm SMS reply under 160 chars
- escalate: boolean
- tags: array of GHL tags to apply
- pipeline_stage: GHL pipeline stage name

Lead reply: {{reply_text}}
Conversation history: {{conversation_history}}

Return valid JSON only.

---

## Edge Cases

| Scenario | Handling |
|---|---|
| Reply contains "STOP" | Unsubscribe immediately, tag opted-out |
| Non-English reply | Respond in same language; escalate if unsupported |
| Gibberish / accidental | Treat as needs_more_info, ask clarifying question |
| Hostile or profane reply | De-escalate, offer human handoff, tag needs-human |

---

## GHL Tags Reference

| Tag | Meaning |
|---|---|
| consultation-scheduled | Lead booked a consultation |
| hot-reply | Urgent — escalated to attorney |
| in-conversation | Active AI conversation |
| not-interested | Declined — do not re-contact |
| no-reply | No response after follow-up |
| opted-out | STOP received — unsubscribed |
| needs-human | Requires live human takeover |
| qualified | AI confirmed lead is qualified |

---

Built by ausjones84 — Build #2: Missed Lead SMS Re-Engagement Bot
