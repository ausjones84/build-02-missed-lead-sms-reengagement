# Build #2: Missed Lead SMS Re-Engagement Bot

> **Stack:** GHL + n8n + OpenAI + LangChain
> **Problem Solved:** Leads that called but hung up are never followed up with
> **Pricing:** $1,500 Setup + $497/month

---

## Overview

This build deploys a fully automated missed-lead recovery system. Within 60 seconds of a missed call or abandoned form in GHL, an AI-powered SMS fires with a personalized message. The conversational AI then handles replies, qualifies the lead, books consultations, and escalates hot responses to the attorney via Slack or SMS.

**Key metric:** Most leads go cold within 5 minutes of a missed call. This bot responds in under 60 seconds — before they call a competitor.

---

## Architecture

```
GHL Missed Call / Abandoned Form
           ↓
    GHL Webhook → n8n (< 60 seconds)
           ↓
  Personalized SMS Sent to Lead
  "Hi [Name], sorry we missed you..."
           ↓
    Lead Replies?
      ↙           ↘
  YES              NO
  ↓                ↓
 OpenAI          End sequence
 Conversation    (tagged: no-reply)
 Engine
  ↓
 Qualify → Book → Escalate
  ↓         ↓       ↓
 GHL       GHL    Slack/SMS
 Update   Calendar  Alert
```

---

## Tech Stack

| Component | Tool | Purpose |
|-----------|------|---------|
| CRM / Trigger | GoHighLevel (GHL) | Detects missed call / abandoned form, fires webhook |
| Workflow | n8n | Orchestrates SMS send, reply handling, routing |
| AI Conversation | OpenAI GPT-4o | Handles reply conversation, qualification, booking |
| Conversation Memory | LangChain | Maintains multi-turn context per contact |
| Notifications | Slack + SMS | Escalates hot replies to attorney |

---

## File Structure

```
build-02-missed-lead-sms-reengagement/
├── README.md
├── n8n/
│   ├── workflow-missed-call-trigger.json   # Main trigger + 60-sec SMS workflow
│   └── workflow-reply-handler.json         # Inbound SMS reply handler + AI logic
├── prompts/
│   ├── system-prompt.md                    # OpenAI system prompt for SMS convo
│   └── qualification-logic.md             # Scoring rules for replies
├── ghl/
│   ├── webhook-setup.md                    # How to configure GHL webhook trigger
│   ├── custom-fields.md                    # Additional custom fields for this build
│   └── workflow-escalation.md             # GHL side of hot-lead escalation
├── docs/
│   ├── setup-guide.md                      # Full setup walkthrough
│   └── pricing.md                          # Pricing and deliverables
└── .env.example
```

---

## How It Works — Step by Step

### Trigger 1: Missed Call
- GHL detects an inbound call that went unanswered
- Fires webhook to n8n within seconds

### Trigger 2: Abandoned Form
- Lead started filling out a contact form but didn't submit
- GHL partial submission webhook fires to n8n

### Step 1: Personalized SMS (< 60 seconds)
n8n pulls contact name from GHL and sends:

> "Hi [FirstName], this is [FIRM NAME]. We just missed your call and don't want to leave you hanging. Can I help you today? Just reply here."

If no name: "Hi there, we just missed your call..."

### Step 2: Reply Detection
- GHL receives inbound SMS reply
- Fires inbound message webhook to n8n reply handler
- n8n identifies the contact and loads conversation history from LangChain memory

### Step 3: OpenAI Conversation Engine
GPT-4o handles the reply with the system prompt context:
- Acknowledges the lead warmly
- Asks 1-2 qualifying questions (what type of case, brief description)
- Offers to book a free consultation
- Handles objections ("I'm just researching", "I'll call back later")

### Step 4: Intent Classification
After each reply, n8n runs intent classification:

| Intent | Action |
|--------|--------|
| **Book Now** | Sends calendar link via SMS, creates GHL task |
| **Hot Lead** | Escalates to attorney via Slack + SMS alert |
| **Needs More Info** | Continues conversation, asks follow-up |
| **Not Interested** | Tags contact, ends gracefully |
| **No Reply (15 min)** | Sends one follow-up, then closes |

### Step 5: Escalation (Hot Replies)
When lead shows strong buying intent:
- Attorney gets Slack message: "@attorney HOT REPLY from [Name] — [excerpt]"
- Attorney gets SMS: "Hot lead responding: [Name] [Phone] — reply received"
- GHL contact tagged: hot-reply, moved to Hot Lead pipeline stage
- Conversation thread logged to GHL contact notes

### Step 6: Booking Flow
If lead agrees to book:
1. n8n sends GHL calendar link for free consultation
2. GHL automation sends booking confirmation on appointment
3. 24h + 1h reminder automations fire automatically

---

## Conversation Examples

### Example 1: Quick Convert
> **Bot:** Hi Sarah, this is Johnson Law. We just missed your call and don't want to leave you hanging. Can I help you today?
> **Lead:** Yes I was in a car accident last week
> **Bot:** I'm sorry to hear that, Sarah. I want to make sure you get the right help. Our attorneys handle car accident cases and offer a free consultation with no obligation. Would you like to grab a 15-minute slot today or tomorrow?
> **Lead:** Sure tomorrow works
> **Bot:** Perfect! Here's a link to book: [calendar link]. You'll get a confirmation text right after. Our attorneys look forward to speaking with you.

### Example 2: Escalation
> **Bot:** Hi Mike, this is Johnson Law. We just missed your call...
> **Lead:** My wife was seriously injured by a drunk driver and I need help NOW
> **Bot:** Mike, I'm so sorry about your wife. This is exactly the type of urgent situation our attorneys handle. Let me flag this for our senior attorney right now — you should hear from them within the next few minutes. Is this the best number to reach you?

*[Hot escalation fires to attorney immediately]*

---

## Setup Instructions

See [docs/setup-guide.md](docs/setup-guide.md) for full walkthrough.

### Quick Start

1. **GHL** — Configure missed call webhook and inbound SMS webhook per `ghl/webhook-setup.md`
2. **n8n** — Import both workflow JSONs, set credentials (GHL API, OpenAI API, Slack webhook)
3. **OpenAI** — Configure GPT-4o with system prompt from `prompts/system-prompt.md`
4. **LangChain** — Set up conversation memory store (Redis or in-memory per setup guide)
5. **Slack** — Create webhook URL for escalation channel, add to env vars

---

## Environment Variables

```env
# OpenAI
OPENAI_API_KEY=sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
OPENAI_MODEL=gpt-4o

# GoHighLevel
GHL_API_KEY=your_ghl_api_key
GHL_LOCATION_ID=your_location_id
GHL_PIPELINE_ID=your_pipeline_id
GHL_HOT_REPLY_STAGE_ID=stage_id
GHL_CALENDAR_ID=your_calendar_id

# Slack
SLACK_WEBHOOK_URL=https://hooks.slack.com/services/xxx/xxx/xxx
SLACK_CHANNEL=#hot-leads

# LangChain / Memory
LANGCHAIN_MEMORY_TYPE=redis
REDIS_URL=redis://localhost:6379

# n8n
N8N_MISSED_CALL_WEBHOOK=https://your-n8n.com/webhook/ghl-missed-call
N8N_INBOUND_SMS_WEBHOOK=https://your-n8n.com/webhook/ghl-inbound-sms

# Attorney Notification
ATTORNEY_PHONE=+1XXXXXXXXXX
ATTORNEY_SLACK_ID=@attorney-username
```

---

## Pricing

| Item | Cost |
|------|------|
| **Setup Fee** | $1,500 (one-time) |
| **Monthly Retainer** | $497/month |

### Setup Includes
- GHL webhook configuration (missed call + inbound SMS)
- n8n workflow build (both workflows)
- OpenAI conversation engine + prompt engineering
- LangChain memory integration
- Slack escalation setup
- Attorney SMS alert configuration
- 20 test conversations across scenarios
- Documentation + handoff

### Monthly Retainer Includes
- Conversation monitoring (response quality checks)
- Prompt refinements based on reply data
- Escalation logic tuning
- Monthly report: leads contacted, reply rate, conversions, booked consultations
- Up to 3 hours/month of changes

---

## ROI Expectations

Industry data shows 35-50% of missed calls never call back. For a firm with 20 missed calls/month:
- Even recovering **2 additional cases/month** at $5,000 avg attorney fee = **$10,000/month recovered**
- Against total cost of ~$650/month (retainer + platform costs): **15x ROI**

---

## Repos Referenced

- [n8n](https://github.com/n8n-io/n8n) — Workflow automation
- [LangChain](https://github.com/langchain-ai/langchain) — Conversation memory + chaining

---

## License

MIT — Built by ausjones84
