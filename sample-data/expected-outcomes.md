# Expected Outcomes — Kitchen Booking Test Matrix

Business hours assumption: **08:00–18:00 local (Europe/Berlin, +02:00)**, Mon–Fri.
Guest Chef rule: effective start = requestedStart − 60min (prep), effective end = requestedEnd + 30min (cleanup). Books **Main Kitchen** + **Prep Station**.
Regular rule: effective times = requested times. Books **Main Kitchen** only.

| # | Visitor | Type | Requested Window | Effective Window (after buffer) | Calendars Booked | Slack/Discord Summary | Error-Email (HR) |
|---|---------|------|------------------|---------------------------------|------------------|-----------------------|------------------|
| 1 | Sarah Kowalski | Regular | 2026-04-15 10:00–11:30 | 10:00–11:30 (unchanged) | Main Kitchen | "Booking confirmed: Sarah Kowalski (Regular) — Main Kitchen, Wed 15 Apr 10:00–11:30." | None |
| 2 | Marco Benedetti | Guest Chef | 2026-04-16 14:00–16:00 | **13:00–16:30** (−60 prep / +30 cleanup) | Main Kitchen + Prep Station | "Guest Chef booking: Marco Benedetti — Main Kitchen & Prep Station, Thu 16 Apr 13:00–16:30 (incl. prep + cleanup)." | None |
| 3 | Yuki Tanaka | Guest Chef | 2026-04-17 09:00–11:00 | **08:00–11:30** (prep clamped to 08:00 open) | Main Kitchen + Prep Station | "Guest Chef booking: Yuki Tanaka — 08:00–11:30. WARNING: prep buffer truncated to business hours open." | None (warning only; soft alert to HR channel optional) |
| 4 | Priya Ramanathan | Guest Chef | 2026-04-20 12:00–13:30 | **11:00–14:00** | Main Kitchen + Prep Station | "Guest Chef booking: Priya Ramanathan — 11:00–14:00." | None |
| 5 | David Okafor | Regular | 2026-04-20 13:30–14:30 | 13:30–14:30 | — **REJECTED** | "Booking conflict: David Okafor (Regular) 13:30–14:30 overlaps Priya Ramanathan cleanup buffer (ends 14:00). Please reschedule to ≥14:00." | HR alert: conflict detected, no calendar write |
| 6 | Elena Vasquez | Guest Chef | 2026-04-21 10:00–16:00 | **09:00–16:30** | Main Kitchen + Prep Station | "Guest Chef booking: Elena Vasquez — long session 09:00–16:30 (6h + buffers)." | None |
| 7 | Tom Fletcher | Regular | 2026-04-22 12:15–12:45 | 12:15–12:45 | Main Kitchen | "Booking confirmed: Tom Fletcher (Regular) — 30-min slot 12:15–12:45." | None |
| 8 | Hans Mueller | Guest Chef | 2026-04-23 16:30–18:00 | **15:30–18:30** — cleanup exceeds 18:00 close | Main Kitchen + Prep Station (with warning) OR rejected per policy | "Guest Chef booking: Hans Mueller — 15:30–18:30. WARNING: cleanup extends past business hours close (18:00)." | Soft HR alert: out-of-hours cleanup flagged for facilities |
| 9 | (missing) | Guest Chef | invalid / broken-email | N/A — validation failure | None | "Booking rejected: payload validation failed (missing visitorName, invalid requestedStart, invalid email)." | **HR error email sent** — subject: "Kitchen Booking Validation Error", body includes raw payload + field-level errors |
| 10 | Aisha Bello | Regular | 2026-04-25 11:00–12:30 (Saturday) | N/A — weekend | None | "Booking rejected: weekend booking not permitted (Sat 25 Apr). Kitchen available Mon–Fri only." | HR alert (informational): weekend request received |
| 11 | Lucas Chen | Guest Chef | 2026-04-27 13:00–15:00 | **12:00–15:30** (would be valid) | None — **calendar API failure simulated** | "Booking FAILED: Lucas Chen — calendar API unreachable after retries. Manual intervention required." | **HR error email sent** — subject: "Kitchen Booking API Failure", body includes booking payload, retry count, upstream error, and instructions to manually add to calendars |

## Prep-Buffer Logic Validation Checklist

- [x] Row 1: Regular → no buffer applied (baseline)
- [x] Row 2: Guest Chef → −60 / +30 applied on both calendars
- [x] Row 3: Prep buffer clamped at business-hours open
- [x] Rows 4+5: Buffer overlap correctly blocks subsequent Regular booking
- [x] Row 6: Buffers applied to long session without distortion
- [x] Row 7: Short Regular slot passes through untouched
- [x] Row 8: Cleanup buffer exceeds close → warning path
- [x] Row 9: Malformed payload → validation error path → HR email
- [x] Row 10: Weekend → policy rejection path
- [x] Row 11: Upstream API failure → error path → HR email with retry metadata

## Notes for Nova (Backend) & Archi (Architect)

- Row 5 assumes the conflict check runs against **effective** (buffered) times in Main Kitchen, not raw requested times. Confirm with Archi before implementation.
- Row 3 / Row 8 behavior (clamp vs. reject) is a policy decision; current expectation is "clamp + warn." Flag to Archi if strict rejection is preferred.
- HR alert email address should be a single config value (e.g. `HR_ALERT_EMAIL`) — do not hard-code in workflow nodes.
