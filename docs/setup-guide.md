# Setup Guide — Missed Lead SMS Re-Engagement Bot

Complete step-by-step setup walkthrough for Build #2.

**Time to complete:** ~2–3 hours
**Prerequisites:** Active GHL account, n8n instance, OpenAI API key, Slack workspace

---

## Step 1: GoHighLevel Setup

### 1a. Add Custom Fields
Follow ghl/custom-fields.md to create all required contact and opportunity fields.

Verify these fields exist before continuing:
- bot_conversation_status
- bot_last_reply_received
- bot_urgency_score
- bot_escalated
- bot_conversation_id
- missed_call_time
- trigger_source
- initial_sms_sent

### 1b. Configure Phone Number
1. Go to Settings > Phone Numbers
2. Purchase or assign a GHL phone number for the bot
3. Enable SMS sending on the number
4. Note the phone number — add it to your .env file as GHL_PHONE_NUMBER

### 1c. Set Up Webhooks for Missed Call Trigger
Follow ghl/webhook-setup.md for full instructions.

Quick summary:
1. Go to Settings > Integrations > Webhooks
2. Create a new webhook: Missed Call
3. Set the URL to your n8n webhook: {{N8N_MISSED_CALL_WEBHOOK}}
4. Enable events: Missed Call, Voicemail
5. Save and copy the webhook URL into your n8n workflow

### 1d. Set Up Inbound SMS Webhook
1. In GHL, go to Settings > Integrations > Webhooks
2. Create a new webhook: Inbound SMS
3. Set the URL to: {{N8N_INBOUND_SMS_WEBHOOK}}
4. Enable events: Inbound Message (SMS)
5. Save

### 1e. Create the Pipeline Stages
In GHL, go to Pipelines and verify or create these stages:
- New Lead
- SMS Bot Active
- Hot Lead
- Consultation Booked
- Not Interested
- No Reply
- Opted Out

Note the Pipeline ID and each Stage ID — add them to your .env file.

### 1f. Build GHL Automations
Follow ghl/workflow-escalation.md to create:
- Hot Lead Escalation Receiver automation
- Consultation Booked Confirmation automation
- No Reply Close Sequence automation
- Opt-Out Handler automation

---

## Step 2: n8n Setup

### 2a. Import Workflows
1. Open your n8n instance
2. Go to Workflows > Import from File
3. Import n8n/workflow-missed-call-trigger.json
4. Import n8n/workflow-reply-handler.json
5. Both workflows should appear in your workflow list

### 2b. Configure Credentials
In n8n, go to Settings > Credentials and create:

**GHL API Credential**
- Type: HTTP Header Auth
- Header Name: Authorization
- Header Value: Bearer {{GHL_API_KEY}}

**OpenAI Credential**
- Type: OpenAI API
- API Key: {{OPENAI_API_KEY}}

**Slack Credential**
- Type: Slack OAuth2 or Webhook URL
- Webhook URL: {{SLACK_WEBHOOK_URL}}

**Redis Credential** (for LangChain memory)
- Type: Redis
- Host: {{REDIS_HOST}}
- Port: 6379

### 2c. Update Workflow Variables
In each workflow, update the following nodes with your actual values:

workflow-missed-call-trigger.json:
- GHL API node: set credential to your GHL API credential
- SMS Send node: set From number to your GHL phone number
- Set firm name in SMS template message

workflow-reply-handler.json:
- GHL API node: set credential
- OpenAI node: set credential and model (gpt-4o)
- LangChain memory node: set Redis credential
- Slack node: set webhook URL and channel
- GHL calendar link: update with your actual booking URL

### 2d. Activate Both Workflows
1. Open workflow-missed-call-trigger
2. Toggle Active to ON
3. Copy the webhook URL from the Webhook trigger node
4. Paste this URL into GHL missed call webhook (Step 1c)
5. Open workflow-reply-handler
6. Toggle Active to ON
7. Copy the webhook URL
8. Paste into GHL inbound SMS webhook (Step 1d)

---

## Step 3: OpenAI Configuration

### 3a. System Prompt
Copy the contents of prompts/system-prompt.md and paste it into the OpenAI node inside workflow-reply-handler.json.

Customize:
- Replace [FIRM NAME] with your actual firm name
- Replace [ATTORNEY NAME] with the lead attorney's name
- Update the practice areas to match your firm's focus
- Adjust the tone if needed

### 3b. Intent Classification
The qualification logic in prompts/qualification-logic.md is already embedded in workflow-reply-handler.json. Review and adjust the urgency thresholds if needed.

---

## Step 4: LangChain Memory Setup

### Option A: Redis (Recommended for Production)
1. Set up a Redis instance (Railway, Render, Upstash, or self-hosted)
2. Get the Redis connection URL
3. Add REDIS_URL to your .env file
4. In n8n, configure the Redis credential with your URL
5. The LangChain memory node will automatically store conversation history keyed by contact ID

### Option B: In-Memory (Development/Testing Only)
1. In the LangChain memory node, switch from Redis to In-Memory buffer
2. Note: conversation history will be lost if n8n restarts — not suitable for production

---

## Step 5: Slack Configuration

### 5a. Create Incoming Webhook
1. Go to api.slack.com/apps
2. Create a new app (or use existing)
3. Enable Incoming Webhooks
4. Add a webhook to your #hot-leads channel (or create this channel first)
5. Copy the webhook URL to SLACK_WEBHOOK_URL in .env

### 5b. Identify Attorney Slack User
1. In Slack, find the attorney's Slack user ID (right-click > View profile > More > Copy member ID)
2. Add to ATTORNEY_SLACK_ID in .env (format: @username or U0XXXXXXX)

---

## Step 6: Environment Variables

Create a .env file based on .env.example and fill in all values:

GHL_API_KEY — from GHL Settings > Integrations > API
GHL_LOCATION_ID — from GHL Settings > Business Info (Location ID)
GHL_PIPELINE_ID — from GHL Pipelines (in URL when viewing pipeline)
GHL_CALENDAR_ID — from GHL Calendars > your calendar > Settings
OPENAI_API_KEY — from platform.openai.com/api-keys
SLACK_WEBHOOK_URL — from api.slack.com (Step 5a)
ATTORNEY_PHONE — attorney's direct mobile number (+1XXXXXXXXXX)
REDIS_URL — your Redis connection string
N8N_MISSED_CALL_WEBHOOK — from n8n workflow trigger node
N8N_INBOUND_SMS_WEBHOOK — from n8n reply handler trigger node

---

## Step 7: Testing

### Test 1: Missed Call Trigger
1. In GHL, open a test contact with a phone number
2. Manually trigger the webhook using a tool like Postman or curl:
   POST {{N8N_MISSED_CALL_WEBHOOK}}
   Body: { "contact_id": "TEST_ID", "phone": "+1XXXXXXXXXX", "first_name": "Test" }
3. Verify: SMS is sent within 60 seconds
4. Verify: GHL contact updated with bot_conversation_status = initial_sms_sent

### Test 2: Reply Handler — Book Now Intent
1. Reply to the test SMS with: "Yes I want to schedule a consultation"
2. Verify: Bot responds with calendar link
3. Verify: GHL contact tagged consultation-scheduled
4. Verify: Pipeline stage moved to Consultation Booked

### Test 3: Reply Handler — Hot Lead Escalation
1. Reply to test SMS with: "My wife was seriously injured, I need help NOW"
2. Verify: Bot sends holding message
3. Verify: Attorney receives Slack notification within 30 seconds
4. Verify: Attorney receives SMS alert
5. Verify: GHL contact tagged hot-reply and moved to Hot Lead stage
6. Verify: Contact note added with message excerpt

### Test 4: No Reply Follow-Up
1. Send the initial SMS to a test number
2. Do not reply
3. After 15 minutes, verify: follow-up SMS is sent
4. Wait 24 hours (or manually trigger the no-reply webhook)
5. Verify: Contact tagged no-reply, sequence closed

### Test 5: Opt-Out
1. Reply to the test SMS with: STOP
2. Verify: No further messages are sent
3. Verify: GHL contact tagged opted-out
4. Verify: bot_opted_out field set to true

---

## Step 8: Go Live Checklist

Before going live with real leads, confirm:

- [ ] All GHL custom fields created and verified
- [ ] Both n8n workflows active
- [ ] SMS sends within 60 seconds on test trigger
- [ ] AI replies are warm and accurate for each intent type
- [ ] Hot lead escalation reaches attorney via Slack and SMS
- [ ] STOP/opt-out compliance working correctly
- [ ] Calendar link in SMS is correct and bookings are tracking in GHL
- [ ] Redis memory is persisting conversations correctly
- [ ] Attorney has been briefed on the escalation flow
- [ ] Test conversations completed across all 5 intent types

---

## Troubleshooting

**SMS not sending:**
- Check GHL phone number is SMS-enabled
- Verify GHL API key has send message permission
- Check n8n workflow is active and webhook URL matches GHL

**AI not responding:**
- Verify OpenAI API key is valid and has credits
- Check the inbound SMS webhook URL in GHL matches n8n
- Check n8n reply handler workflow is active

**Escalation not firing:**
- Check urgency_score threshold in intent classification node (should be >= 3)
- Verify Slack webhook URL is valid
- Check attorney phone number format (+1XXXXXXXXXX)

**Memory not persisting:**
- Verify Redis connection string is correct
- Check bot_conversation_id is being written to GHL contact
- Confirm LangChain memory node uses contact ID as session key

---

*Build #2 — Missed Lead SMS Re-Engagement Bot*
*Stack: GHL + n8n + OpenAI + LangChain*
*Pricing: $1,500 Setup + $497/month*
*Built by ausjones84*
