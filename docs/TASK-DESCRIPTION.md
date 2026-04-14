# Task Description — Kitchen Booking Automation

## Overview
Build an automated n8n workflow that handles kitchen bookings for two distinct visitor types — **Regular** users and **Guest Chefs** — with different calendar rules, automatic prep-time buffering, dual-channel team notifications, and robust error escalation to HR.

## Objective
Eliminate manual coordination of kitchen bookings. The workflow must automatically recognise the visitor type, apply the correct time adjustments, book the appropriate calendars, notify the team, and escalate any integration failure to HR — with zero human intervention.

## Scope of Work

### 1. Intake
- Accept booking requests via a webhook endpoint.
- Capture: visitor name, visitor type (`Regular` | `Guest Chef`), requested start time (ISO-8601), requested end time (ISO-8601), contact email, and free-text notes.

### 2. Conditional Logic
- Route requests based on `visitorType`:
  - **Guest Chef path:** apply a 60-minute prep buffer (subtract from start) and a 30-minute cleanup buffer (add to end), then book TWO calendars — Main Kitchen and Prep Station.
  - **Regular path:** no buffer; book only the Main Kitchen calendar with submitted times.

### 3. Notifications
- After successful booking, post a formatted confirmation to the **"Kitchen Updates"** channel on both **Slack** and **Discord**.
- Message must include visitor name, visitor type, requested times, effective times (after buffer), buffer-applied flag, and which calendars were booked.

### 4. Error Handling
- Any failure on an external API call (Google Calendar, Slack, Discord) must route to a fallback node that emails the **HR Manager** with: workflow name, timestamp, failing node name, original payload, and error detail.
- No booking failure may be silent.

### 5. Documentation
- Every node must have a clear, human-readable name so non-technical operations staff can understand the flow.
- Include setup instructions, process walkthrough, logic rationale, and sample test data.

## Deliverables
1. **n8n workflow export** — JSON file, importable into n8n 1.x, credentials stripped.
2. **Setup screenshots** — full workflow canvas plus close-ups of conditional-logic nodes.
3. **Process walkthrough** — PDF or short video explaining a successful Guest Chef entry producing the dual-calendar booking and notifications, aimed at non-technical staff.
4. **Sample data** — CSV or text file containing at least 10 test rows covering happy paths, edge cases (buffer at business-hour boundary, back-to-back bookings, weekend, malformed payload), and a row that simulates API failure to exercise the HR alert path.

## Success Criteria
The workflow is considered successful when:

- ✅ It correctly distinguishes between Regular and Guest Chef requests and produces different outcomes for each.
- ✅ Guest Chef bookings automatically occupy both Main Kitchen and Prep Station calendars with the -60 min / +30 min buffer applied.
- ✅ Regular bookings occupy only the Main Kitchen with unmodified times.
- ✅ Team notifications appear on both Slack and Discord "Kitchen Updates" channels on success.
- ✅ HR receives an urgent email whenever an external call fails.
- ✅ The workflow is self-sufficient — no manual step between submission and completion.

## Judgement Criteria
Work is evaluated on three dimensions:

1. **Elegance of conditional logic** — Is timing math centralised? Are branches symmetric? Does the IF node only route, or does it carry business logic?
2. **Robustness of error handling** — Does every external call have a wired error path? Is the HR fallback tested? Are failures impossible to swallow?
3. **Clarity of workflow documentation** — Can a non-technical operations staff member read the node names and understand what the workflow does? Can a reviewer understand the design decisions without opening the JSON?

## Assumptions
- Business hours: 08:00–18:00, Monday–Friday (documented; enforcement deferred).
- Conflict detection uses buffered (effective) times, not raw requested times.
- SMTP credential available for HR fallback email.
- Google Calendar OAuth, Slack webhook or OAuth, and Discord webhook are provided.

## Out of Scope
- Pre-booking conflict detection (recommended as follow-up).
- Webhook authentication (recommended as follow-up).
- Localisation / multi-language confirmation messages.
- Calendar cancellation or modification workflow.
