# Kitchen Booking Workflow — Node Map

Reference table for the `kitchen-booking.workflow.json` automation. Every node is listed in execution order with its type and purpose.

| # | Node Name | n8n Node Type | Purpose |
|---|-----------|---------------|---------|
| 1 | Intake Form Webhook | `n8n-nodes-base.webhook` | Entry point. Receives POST with `visitorName`, `visitorType`, `requestedStart`, `requestedEnd`, `email`, `notes`. Swap for a Form Trigger if collecting via n8n-hosted form. |
| 2 | Normalize & Calculate Buffers | `n8n-nodes-base.set` | Single source of truth for timing math. Flattens webhook body and computes `effectiveStart`, `effectiveEnd`, `isGuestChef`, and buffer sizes. Guest Chef gets -60 min prep / +30 min cleanup; Regular keeps requested times. |
| 3 | Check Visitor Type | `n8n-nodes-base.if` | Conditional router. TRUE output = Guest Chef path. FALSE output = Regular path. |
| 4 | Book Main Kitchen (Guest Chef) | `n8n-nodes-base.googleCalendar` | Creates Main Kitchen event using effective (buffered) window. Calendar ID placeholder: `MAIN_KITCHEN_CAL_ID`. Errors route to HR fallback. |
| 5 | Book Prep Station (Guest Chef) | `n8n-nodes-base.googleCalendar` | Creates Prep Station reservation covering only the 60-min prep window before requested start. Placeholder: `PREP_STATION_CAL_ID`. Errors route to HR fallback. |
| 6 | Book Main Kitchen (Regular) | `n8n-nodes-base.googleCalendar` | Creates Main Kitchen event at exact requested times (no buffer). Placeholder: `MAIN_KITCHEN_CAL_ID`. Errors route to HR fallback. |
| 7 | Format Confirmation Message | `n8n-nodes-base.set` | Builds a single confirmation string (visitor, type, requested vs effective times, buffer Y/N, calendars booked) reused by both chat channels. |
| 8 | Send Slack Confirmation | `n8n-nodes-base.slack` | Posts confirmation to the `Kitchen Updates` Slack channel. Errors route to HR fallback. |
| 9 | Send Discord Confirmation | `n8n-nodes-base.discord` | Posts same confirmation to the `Kitchen Updates` Discord channel. Parallel sibling of Slack — one failing does not block the other. Errors route to HR fallback. |
| 10 | Fallback: Alert HR Manager | `n8n-nodes-base.emailSend` | Central error sink. Every external call routes its error output here. Sends URGENT email to `HR_MANAGER_EMAIL` placeholder with workflow name, node name, timestamp, error message, and original payload for manual recovery. |

## Placeholders to replace before going live

| Placeholder | Where | Replace with |
|-------------|-------|--------------|
| `MAIN_KITCHEN_CAL_ID` | Nodes 4 & 6 | Google Calendar ID for Main Kitchen |
| `PREP_STATION_CAL_ID` | Node 5 | Google Calendar ID for Prep Station |
| `HR_MANAGER_EMAIL` | Node 10 | HR manager's email address |
| `n8n-alerts@example.com` | Node 10 (from) | Verified sender address on your SMTP credential |

## Flow summary

```
Webhook → Normalize+Buffer → IF (Guest Chef?)
                              ├─ TRUE  → Book Main Kitchen → Book Prep Station ─┐
                              └─ FALSE → Book Main Kitchen ───────────────────┤
                                                                                ▼
                                                              Format Confirmation
                                                                 ├─ Slack
                                                                 └─ Discord

All 5 external calls (3 Calendar, Slack, Discord) have onError:continueErrorOutput
routing to → Fallback: Alert HR Manager (email)
```
