# WhatsApp Task Follow-Up Agent

A zero-cost AI agent that turns a WhatsApp message into a tracked follow-up task — built with n8n, Google's Gemini, Google Sheets, and the WhatsApp Business Cloud API.

## What it does

Text yourself (the "owner") an instruction like:

> Ask Rohan when the delivery is arriving, his number is 919876543210

The agent will:
1. Understand the instruction using Gemini (new task vs. update to an existing one)
2. Message the contact on WhatsApp automatically
3. Log the task in a Google Sheet (contact, task, status, notes)
4. Watch for the contact's reply, interpret it with AI, and update the task's status
5. Report back to you with a summary — and chain to a new contact automatically if the reply hands off to someone else (e.g. "ask my engineer, here's his number")

All orchestration runs on n8n's free tier; Gemini's free API tier covers the reasoning; Google Sheets is the database. No paid infrastructure required.

## Architecture

```
WhatsApp message in
      │
      ▼
 Is it from the owner, or a contact replying?
      │                         │
 (owner: new task/update)   (contact: reply to existing task)
      │                         │
 Gemini decides: new task     Gemini interprets the reply,
 or update existing one?      decides next step, drafts replies
      │                         │
 Log/update in Sheets         Update Sheets, notify owner,
      │                       optionally message a new contact
 Message the contact
 Acknowledge the owner
```

## Stack

- **n8n** — workflow orchestration (webhook trigger, branching logic, Google Sheets nodes)
- **Google Gemini** (via n8n's free Gateway credits, or your own API key) — task routing and reply interpretation, with structured JSON output
- **Google Sheets** — task log (ContactName, ContactPhone, TaskDescription, Status, Notes, CreatedAt)
- **WhatsApp Business Cloud API** (Meta) — sending and receiving messages

## Setup

1. Import `whatsapp-followup-agent.json` into your n8n instance.
2. Create a Google Sheet with a `Tasks` tab and columns: `ContactName`, `ContactPhone`, `TaskDescription`, `Status`, `Notes`, `CreatedAt`.
3. Set up a Meta developer app with the WhatsApp product, grab a test number (or your own verified number), and generate a permanent access token.
4. In n8n, fill in:
   - `Set Config` → your own WhatsApp number (digits only, international format) as `ownerPhone`
   - All WhatsApp send nodes → your Phone Number ID from Meta
   - Google Sheets nodes → your Sheet ID
   - Credentials → your WhatsApp OAuth / API credentials, and Google Sheets OAuth
5. **Important:** after configuring the webhook URL in Meta's app dashboard, you also need to explicitly subscribe your app to your WhatsApp Business Account — this isn't handled by the dashboard UI. Do it via [Graph API Explorer](https://developers.facebook.com/tools/explorer/):
   - Method: `POST`
   - Endpoint: `{your-WABA-id}/subscribed_apps`
   - Submit — you should get `{"success": true}`

   Without this step, Meta's dashboard "Test" webhook button will work, but real incoming messages never reach your webhook. This is a genuinely undocumented gotcha.
6. Publish/activate the workflow in n8n.

## Notes

- Credentials, phone numbers, and sheet IDs in this repo are placeholders — fill in your own.
- Uses n8n's free Gateway credits for Gemini by default, so no separate API key is strictly required to try it.
