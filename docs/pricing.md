# Pricing & Deliverables
# Build #2: Missed Lead SMS Re-Engagement Bot

---

## Pricing Summary

| Item | Cost |
|------|------|
| Setup Fee (one-time) | $1,500 |
| Monthly Retainer | $497/month |

---

## Setup Fee - $1,500 (One-Time)

### What Is Delivered

**1. GHL Webhook Configuration**
- Missed call webhook configured + tested
- Inbound SMS webhook configured + tested
- GHL custom fields created (4 fields)
- Pipeline stage added: Missed Call Recovery

**2. n8n Workflow Build & Deployment**
- Workflow 1: Missed call trigger + 60-second SMS (workflow-missed-call-trigger.json)
- Workflow 2: Inbound reply handler + AI conversation engine (workflow-reply-handler.json)
- Both workflows deployed, tested, and activated

**3. OpenAI Conversation Engine**
- GPT-4o configured with custom system prompt
- Firm-specific persona and messaging
- Intent classification logic (book / escalate / continue / stop)
- Escalation triggers tuned for legal urgency signals

**4. Slack Escalation Setup**
- Slack webhook configured
- Hot-lead alert format set up
- Attorney channel configured

**5. Attorney SMS Alert**
- Direct SMS to attorney on hot replies
- Formatted alert with contact name, phone, and message excerpt

**6. Testing**
- 20 test conversations across all intent types
- Escalation flow end-to-end verified
- GHL data population verified
- Slack and SMS alerts verified

**7. Documentation + Handoff**
- Full setup guide (this repo)
- Environment variable reference
- 1-hour handoff call with your team

---

## Monthly Retainer - $497/month

### What Is Included

**Monitoring**
- Weekly workflow health checks
- Alert if webhook failures occur
- Conversation quality review (sampling 10 conversations/week)

**Optimization**
- Monthly prompt refinements based on reply patterns
- Intent classification tuning
- Escalation threshold adjustments
- A/B message variant testing (first message copy)

**Reporting (Monthly)**
- Total missed calls detected
- SMS recovery rate (% that received recovery SMS)
- Reply rate (% that replied)
- Conversion rate (% that booked)
- Hot escalation count
- Attorney response time average
- Estimated cases recovered

**Change Requests**
- Up to 3 hours/month included:
  - New escalation keywords
  - Message copy changes
  - Pipeline stage adjustments
  - New intent types

**Priority Support**
- Response within 4 business hours
- Emergency support for workflow failures

---

## Third-Party Platform Costs (Client Pays Directly)

| Service | Estimated Cost |
|---------|---------------|
| GoHighLevel | $97-$297/mo |
| OpenAI API (GPT-4o) | ~$20-50/mo (varies by volume) |
| n8n Cloud | $20-50/mo |
| Slack | Free (basic) |

**Estimated Total Platform Cost: ~$140-400/month**
**Total All-In Monthly: ~$637-897/month**

---

## ROI Calculator

**Scenario: 20 missed calls/month**
- Industry avg: 35-50% of missed calls never call back without follow-up
- With Build #2: typical reply rate 25-40%, booking rate 10-20%
- 20 missed calls × 15% booking rate = 3 consultations booked
- If 1 of 3 becomes a client at $5,000 avg fee = **$5,000 recovered**
- Against total cost of ~$750/month: **6.7x ROI minimum**

**Scenario: 50 missed calls/month**
- 50 × 15% = 7-8 consultations booked
- 2-3 new clients at $5,000 avg = **$10,000-15,000 recovered/month**
- Against total cost: **13-20x ROI**

---

## Upgrade Options

| Add-On | Cost |
|--------|------|
| Spanish language conversation | +$750 setup, +$150/mo |
| Email follow-up in addition to SMS | +$500 setup, +$100/mo |
| Multi-attorney routing (2+ attorneys) | +$300 setup, +$75/mo |
| Enhanced reporting dashboard | +$150/mo |
| Voicemail drop integration | +$500 setup, +$100/mo |

---

## Contract Terms

- Setup fee due 50% upfront, 50% at launch
- Monthly retainer billed on the 1st of each month
- 2-month minimum commitment on retainer
- 30-day notice to cancel after minimum term
- Work product remains client property upon full payment
