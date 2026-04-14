# Kitchen Booking Workflow (n8n)

An n8n automation that receives kitchen booking requests, applies different scheduling rules for **Regular** visitors vs **Guest Chefs**, books the appropriate Google Calendars, announces the booking on Slack and Discord, and escalates any API failure to the HR Manager by email.

## Deliverables Checklist

- [x] n8n workflow JSON (`workflow/kitchen-booking.json`)
- [x] Sample webhook payloads (`sample-data/`)
- [x] Screenshots of each node executing (`screenshots/`)
- [x] `README.md` — this file
- [x] `WALKTHROUGH.md` — narrated step-by-step process
- [x] `LOGIC_FLOW.md` — plain-English decision logic with diagram

## Folder Structure

```
kitchen-booking-n8n/
├── workflow/
│   └── kitchen-booking.json       # Importable n8n workflow
├── sample-data/
│   ├── guest-chef-request.json    # Example Guest Chef payload
│   └── regular-request.json       # Example Regular visitor payload
├── screenshots/                   # Reference screenshots for docs
└── docs/
    ├── README.md
    ├── WALKTHROUGH.md
    └── LOGIC_FLOW.md
```

## What the Workflow Does

| Visitor Type | Prep Buffer | Cleanup Buffer | Calendars Booked |
|--------------|-------------|----------------|------------------|
| Regular      | None        | None           | Main Kitchen |
| Guest Chef   | -60 min     | +30 min        | Main Kitchen + Prep Station |

On success: posts a confirmation card to the Slack and Discord **Kitchen Updates** channel.
On failure of any calendar, Slack, or Discord call: sends an urgent email to the HR Manager with the error payload.

## Setup Instructions

### 1. Prerequisites

- A running n8n instance (cloud or self-hosted, v1.0+)
- Google Workspace account with access to both calendars
- Slack workspace with an app + bot token
- Discord server with an incoming webhook
- SMTP credentials for outbound email

### 2. Import the Workflow

1. In n8n, open **Workflows -> Import from File**.
2. Select `workflow/kitchen-booking.json`.
3. Save the workflow (do not activate yet).

### 3. Configure Credentials

Create the following credentials in **Settings -> Credentials**:

| Credential Name            | Type                     | Used By |
|----------------------------|--------------------------|---------|
| `GoogleCal_MainKitchen`    | Google Calendar OAuth2   | Main Kitchen booking node |
| `GoogleCal_PrepStation`    | Google Calendar OAuth2   | Prep Station booking node |
| `Slack_KitchenBot`         | Slack API (Bot Token)    | Slack notification node |
| `Discord_Webhook`          | HTTP Header Auth / none  | Discord notification node |
| `SMTP_HROutbound`          | SMTP                     | HR Manager error email |

Then open each node and bind the matching credential.

### 4. Configure Environment Placeholders

Set these as n8n environment variables (or edit the `Set` node named **Config**):

| Variable               | Example                                  | Purpose |
|------------------------|------------------------------------------|---------|
| `MAIN_KITCHEN_CAL_ID`  | `main-kitchen@group.calendar.google.com` | Target calendar ID for all bookings |
| `PREP_STATION_CAL_ID`  | `prep-station@group.calendar.google.com` | Target calendar ID for Guest Chef prep |
| `HR_MANAGER_EMAIL`     | `hr.manager@example.com`                 | Recipient for failure alerts |
| `SLACK_CHANNEL`        | `#kitchen-updates`                       | Slack destination channel |
| `DISCORD_WEBHOOK`      | `https://discord.com/api/webhooks/...`   | Discord Kitchen Updates webhook URL |

> 💡 **Tip**: Keep credentials and env values out of the exported JSON. Use n8n's credential store and `$env.VAR_NAME` expressions.

### 5. Activate the Workflow

Toggle the workflow to **Active**. Copy the **Production Webhook URL** from the Webhook trigger node — this is the endpoint your booking form should POST to.

## How to Test

### Option A: Use Sample Payloads

```bash
# Guest Chef request (triggers buffer logic + dual calendar)
curl -X POST "$WEBHOOK_URL" \
  -H "Content-Type: application/json" \
  -d @sample-data/guest-chef-request.json

# Regular request (single calendar, no buffer)
curl -X POST "$WEBHOOK_URL" \
  -H "Content-Type: application/json" \
  -d @sample-data/regular-request.json
```

### Option B: Manual Execution in n8n

1. Open the workflow.
2. Click **Execute Workflow**.
3. Paste a sample payload into the Webhook node's test input.
4. Watch each node light green; inspect the output panel.

### Expected Results

| Test                       | Expect |
|----------------------------|--------|
| Guest Chef submission      | 2 calendar events created, 1 Slack message, 1 Discord message |
| Regular submission         | 1 calendar event created, 1 Slack message, 1 Discord message |
| Calendar API failure       | HR Manager receives email with subject `URGENT: Kitchen Booking Failure` |

> ⚠️ **Note**: To simulate a failure, temporarily revoke the Google Calendar credential or change the calendar ID to an invalid value.

## Support

For workflow questions, contact the Operations team. For credential or access issues, contact IT.

