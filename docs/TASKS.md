# Task Descriptions — Kitchen Booking n8n Workflow

Ticket-style breakdown derived from Milestone 1 evidence and known issues.

---

## TASK-001 — Replace Google Calendar ID placeholders with real calendar IDs
**Type:** Configuration · **Priority:** P0 · **Owner:** Backend / Ops · **Est:** 15 min

**Description**
The three Google Calendar nodes (`Book Main Kitchen (Guest Chef)`, `Book Prep Station (Guest Chef)`, `Book Main Kitchen (Regular)`) currently reference placeholders `MAIN_KITCHEN_CAL_ID` and `PREP_STATION_CAL_ID`. Bookings cannot succeed until real IDs are set.

**Acceptance Criteria**
- Real calendar IDs pasted into all three Calendar nodes.
- Google Calendar OAuth credential attached and authorized.
- A test Guest Chef payload creates two events (Main Kitchen + Prep Station) with the correct buffered times.
- A test Regular payload creates one event (Main Kitchen only).

---

## TASK-002 — Fix Discord notification node (currently erroring)
**Type:** Bug · **Priority:** P0 · **Owner:** Backend · **Est:** 20 min

**Description**
Canvas screenshot shows `Send Discord Confirmation` with a red error badge. Likely cause: missing Discord webhook URL or malformed payload. Slack path works; Discord does not.

**Acceptance Criteria**
- Discord webhook URL configured in node credential.
- Test run posts a formatted confirmation to the "Kitchen Updates" Discord channel.
- Node shows green success badge after a full happy-path run.

---

## TASK-003 — Set Webhook response mode to "When last node finishes"
**Type:** Configuration · **Priority:** P1 · **Owner:** Backend · **Est:** 5 min

**Description**
Earlier test returned an immediate echo of the request body, indicating the Webhook node is responding before downstream nodes complete. Caller should receive the final workflow status, not the raw intake.

**Acceptance Criteria**
- Webhook node `Respond` set to "When last node finishes".
- Final response body contains a confirmation object (visitor, type, effective times, calendars booked).
- HTTP status reflects real outcome: 200 on success, 500 only if fallback itself fails.

---

## TASK-004 — Define and enforce business-hours policy
**Type:** Policy · **Priority:** P1 · **Owner:** HR / Product + Backend · **Est:** 1 hr

**Description**
Data engineer flagged two open policy questions: (a) business-hours window (assumed 08:00–18:00 Mon–Fri) and (b) whether conflict detection should run against buffered (effective) times or raw requested times. Current workflow does not reject out-of-hours bookings.

**Acceptance Criteria**
- Policy decision documented in `docs/LOGIC_FLOW.md`.
- New validation node added before calendar booking: rejects bookings where effective start/end falls outside business hours.
- Rejection path posts a polite message back to the webhook caller and notifies Slack.
- Test rows 3, 7, 9 in `sample-data/test-bookings.csv` produce the expected policy outcomes.

---

## TASK-005 — Capture high-resolution screenshots of conditional logic nodes
**Type:** Documentation · **Priority:** P1 · **Owner:** Technical Writer · **Est:** 30 min

**Description**
Deliverables spec requires high-resolution screenshots of the entire workflow AND of the specific conditional logic nodes (IF, Set buffer calculation). Milestone 1 has only two full-canvas shots.

**Acceptance Criteria**
- Screenshot of `Check Visitor Type` IF node showing the condition expression.
- Screenshot of `Normalize & Calculate Buffers` Set node showing all computed fields and expressions.
- Screenshot of one Calendar node showing buffered start/end expression bindings.
- Screenshot of `Fallback: Alert HR Manager` node showing recipient, subject template, and body template.
- All saved to `screenshots/milestone-2/` at ≥1920px width.

---

## TASK-006 — Produce process walkthrough PDF or short video
**Type:** Documentation · **Priority:** P1 · **Owner:** Technical Writer · **Est:** 2 hr

**Description**
Deliverable spec requires a PDF or short video demo showing a successful Guest Chef entry producing the dual-calendar booking and the notification trigger. `docs/WALKTHROUGH.md` has the script and screenshot placeholders — needs to be rendered.

**Acceptance Criteria**
- PDF or ≤3-minute video saved to `docs/walkthrough.pdf` or `docs/walkthrough.mp4`.
- Covers: Guest Chef submission → buffer calculation → dual booking → Slack + Discord confirmation → error-path fallback to HR email.
- Suitable for non-technical staff review.

---

## TASK-007 — Run full test suite from `test-bookings.csv`
**Type:** QA · **Priority:** P1 · **Owner:** QA · **Est:** 1 hr

**Description**
Eleven sample rows exist in `sample-data/test-bookings.csv` with expected outcomes documented in `sample-data/expected-outcomes.md`. Each must be executed against the live workflow and validated.

**Acceptance Criteria**
- All 11 rows POSTed to the webhook.
- Each actual outcome matches `expected-outcomes.md` (calendar events created, notification sent, or HR alert fired).
- Results logged in a new `sample-data/test-results.md` with pass/fail per row.
- Any deviations filed as new tasks.

---

## TASK-008 — Export and publish final workflow JSON
**Type:** Release · **Priority:** P2 · **Owner:** Backend · **Est:** 15 min

**Description**
Deliverable spec requires either a JSON export or a public URL to the workflow template. Current JSON is the pre-deployment scaffold.

**Acceptance Criteria**
- Final workflow (post TASK-001 through TASK-004) exported fresh.
- Saved to `workflow/kitchen-booking.workflow.json` overwriting the scaffold.
- Credentials stripped from export.
- Optional: public n8n template URL added to `README.md`.

---

## Dependency Graph
```
TASK-001 ──┐
TASK-002 ──┼──► TASK-007 ──► TASK-008
TASK-003 ──┤
TASK-004 ──┘
TASK-005 ──► TASK-006
```
