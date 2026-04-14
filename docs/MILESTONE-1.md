# Milestone 1 — Core Workflow Scaffold & Error-Path Validation

## Objective
Stand up the end-to-end n8n "Kitchen Booking" workflow skeleton with all nodes wired, both visitor branches routable, notification fan-out in place, and the fallback HR alert proven to fire when a downstream node errors.

## Scope (What ships in M1)
Per the canvas in `screenshots/milestone-1/`, the following nodes must exist, be named exactly as shown, and be connected per the graph:

| # | Node | Type | Purpose |
|---|------|------|---------|
| 1 | Intake Form Webhook | Webhook (POST) | Receives booking payload |
| 2 | Normalize & Calculate Buffers | Set | Computes `effectiveStart`, `effectiveEnd`, `isGuestChef` |
| 3 | Check Visitor Type | IF | Routes Guest Chef (true) vs Regular (false) |
| 4 | Book Main Kitchen (Guest Chef) | Google Calendar | Buffered booking on Main Kitchen |
| 5 | Book Prep Station (Guest Chef) | Google Calendar | Buffered booking on Prep Station |
| 6 | Book Main Kitchen (Regular) | Google Calendar | Raw-time booking on Main Kitchen |
| 7 | Format Confirmation Message | Set | Builds human-readable summary |
| 8 | Send Slack Confirmation | Slack | Posts to #kitchen-updates |
| 9 | Send Discord Confirmation | Discord | Posts to Kitchen Updates webhook |
| 10 | Fallback: Alert HR Manager | Email (SMTP) | Urgent alert on any upstream error |

## Acceptance Criteria
- [x] Webhook accepts POST at `/webhook-test/kitchen-booking` and returns 200 for valid Regular payloads.
- [x] Guest Chef payload routes through `Check Visitor Type = true` branch and invokes both calendar nodes.
- [x] Every external-call node (3x Calendar, Slack, Discord) has its **Error** output wired to `Fallback: Alert HR Manager`.
- [x] `Fallback: Alert HR Manager` successfully delivers email via SMTP (verified: `250 2.0.0 OK`, accepted by `joeydngcng1231@gmail.com`).
- [ ] Slack + Discord confirmations post successfully on happy path (Slack: green; Discord currently red — see Known Issues).
- [ ] Google Calendar placeholders `MAIN_KITCHEN_CAL_ID` and `PREP_STATION_CAL_ID` replaced with real IDs.
- [ ] Happy-path Guest Chef booking completes without triggering fallback.

## Evidence (Screenshots)
- `screenshots/milestone-1/workflow-test-until-fallback-alert-hr.png` — full canvas with execution trail; green edges show Guest Chef path reaching the fallback email after a downstream failure. Proves the error-handling contract.
- `screenshots/milestone-1/sending-notification-to-slack.png` — Slack confirmation node highlighted in the notification fan-out stage.

## Test Evidence (from curl runs)
| Payload | Result | Notes |
|---------|--------|-------|
| Regular visitor | HTTP 200, echo body | Webhook + intake verified |
| Guest Chef | HTTP 500 → HTTP 200 w/ SMTP receipt | Fallback HR email fired and delivered |

## Known Issues → carry into Milestone 2
1. **Discord node errors** (red badge in screenshot). Likely missing webhook URL credential.
2. **Google Calendar placeholders** still present; real IDs needed.
3. **Respond-mode on Webhook** — confirm set to "When last node finishes" so downstream status reaches the caller.
4. **Business-hours policy** unresolved (assumed 08:00–18:00 Mon–Fri) — flagged by data engineer.

## Definition of Done (M1)
Reviewer can (a) import the JSON, (b) click Execute Workflow, (c) POST either sample payload, and (d) observe either a successful notification OR a delivered HR fallback email — with zero manual intervention between steps.

## Deliverables in this milestone
- `workflow/kitchen-booking.workflow.json`
- `workflow/NODE_MAP.md`
- `docs/LOGIC_FLOW.md`
- `screenshots/milestone-1/*.png`
- This file (`docs/MILESTONE-1.md`)
