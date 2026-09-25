# OP+ Opportunity Plus: Full Bug & Risk Audit

- **App:** Base44 `696e822e6bb318f6b83d3652` ("OP+ Opportunity Plus")
- **Date:** 25 September 2026
- **Scope:** all ~95 pages, ~90 backend functions, 70 data tables, 40 automations.
- **Method:** six parallel reviews: login & access, bookings, appointments & calendars, staff logs & reports, inquiries/leads/tasks & messaging, build health.
- **Mode:** read-only. No code, data or settings were changed.

Every finding below was checked against the source code, and where possible against live data. Items marked *(confirm)* need a quick check in the Base44 dashboard.

---

## 0. Do these today (no code needed)

| # | Action | Why |
|---|---|---|
| 1 | **Reconnect the Chatbiz WhatsApp sender (94761828838).** | Since **23 Sep** every inquiry alert fails with "No active WhatsApp instance found". Escalations still mark themselves as done, so staff have received **no inquiry alerts for 3 days** and those inquiries will never escalate again. |
| 2 | **Set an access level for `ashani@polaricx.com`.** | Access Management → Users. She is a Base44 admin with no access level, so the app treats her as "pending" and blocks every page. Her pre-assigned `operations_admin` access was never applied. |
| 3 | **Fix 2 users stuck as "approved + pending".** | User IDs `69add55fe87beea300f5dadb` and `69baccc0fbbadb88a6512005`. They're approved but have no access level, so they sit on the waiting screen and never show in the Requests tab. |
| 4 | **Rotate `INTERNAL_FUNCTION_KEY`.** | The current value is written in plain text in 4 workflow files. |
| 5 | **Correct booking 6ab3adad (Jat Holdings).** | The price was typed into the Currency field (`"LKR6,250.00"`), so finance shows LKR 0 revenue and a −670,000 margin. |
| 6 | **Archive junk inquiries.** | Test and spam records such as "John Doe" ×4, "Test" and "testexample" are inflating the unattended counts. |

---

## 1. Login, sign-up & access

| Sev | Problem | Where | Impact |
|---|---|---|---|
| **Critical** | New staff can't submit an access request. The centre list is empty for "pending" users because Branch read permissions exclude them. | `entities/Branch.jsonc` RLS, `Welcome.jsx:29,83` | Every new teacher sees no centres and a disabled Submit button. |
| High | Base44 admins without an `access_level` are treated as pending. | `components/access.js:95` | Admins are locked out (happening now). |
| High | Pre-assigned access only applies on self-signup, not when someone is invited. Emails aren't lower-cased or trimmed. | `workflows/Apply Pre-assigned Access…`, `AccessManagement.jsx:303` | Pre-approved staff still land on "Pending". |
| High | "Remove access" / Decline only sets `approval_status`. `access_level` and `branch_ids` stay, and the data permissions ignore approval status. | `manageUserAccess/entry.ts:39` | Removed staff can still read and write their centre's reports, stock and petty cash. |
| High | A declined user who resubmits stays `rejected` forever and never reappears in Requests. | `Welcome.jsx:65`, `AccessManagement.jsx:64` | Stuck users, and admins are never told. |
| High | The mobile bottom bar is hard-coded to Dashboard / Tasks / Leads / Appointments for everyone. | `MobileNav.jsx:19` | Centre staff on phones hit "No access" on 2 of the 4 tabs. |
| Medium | TeamApprovals, InviteAccept and the Welcome invite check call `asServiceRole` in the browser, which always throws. | `TeamApprovals.jsx`, `InviteAccept.jsx`, `Welcome.jsx:35` | These pages are broken. |
| Medium | Network or unknown auth errors still render the app without a user. | `App.jsx:55`, `RouteGuard.jsx:22` | Blank or crashing pages. |
| Medium | Staff pickers use `User.list()`. Non-admins may see only themselves. *(confirm)* | Leads, Appointments, StaffCalendar, WorkLogAdmin… | Managers can't assign anyone else. |
| Medium | The Team table has no permission rules. | `entities/Team.jsonc` | Any user can create, edit or delete teams. |
| Low | The "Pending" screen never refreshes after approval. Ops managers get silent failures in Access Management. `notifyAccessRequest` can be spammed. | various | Confusion, admin spam. |

## 2. Bookings

| Sev | Problem | Where | Impact |
|---|---|---|---|
| **Critical** | Every booking except cancelled ones, including **public "requested" and tentative bookings**, is pushed to Google Calendar as **confirmed**. Create-then-update produces **duplicate** Google events. | `syncBookingToGoogle:45`, `publicBookingRequest:108,144` | Staff see spam or unconfirmed visits as confirmed, sometimes twice. |
| **Critical** | The live clash check in the booking form is **5.5 h off** because it sends local time with no timezone. | `BookingFormModal.jsx:76`, `bookingLogic.ts:61` | False "conflict" blocks Save, or real clashes only show up as an error at save time. |
| **Critical** | Old Event records are compared 5.5 h late. | `bookingLogic.ts:89` | A 9–11 event blocks 14:30–16:30 instead. |
| High | Currency is free text next to the price. No price is required for paid modes, and negative values are accepted. | `BookingFormModal.jsx:245` | Wrong finance (live example above). |
| High | The public request form can be abused: the rate limit ignores the normalised phone, there's no IP or email limit, and attacker text is emailed to any address. | `publicBookingRequest:75,150` | Inbox and calendar flooding, spam relay. |
| High | The D-1 reminder has no "already sent" flag. Its auth check fails *open* if the env key is missing. | `bookingReminderDaily:11` | Duplicate reminders; anyone could trigger it. |
| High | All-day and blocked dates land a day early in Google. | `syncBookingToGoogle:47` | "Block 1 Oct" shows on 30 Sep. |
| High | The details drawer shows raw UTC times. | `BookingDetailsDrawer.jsx:102` | A 9:00 booking reads "03:30". |
| High | The audit trail is incomplete: no previous status, no field changes, no who-cancelled; deletes and public creates aren't logged. | `saveEventBooking:124` | You can't tell who changed or cancelled a booking. |
| Medium | Multi-day bookings appear only on day 1 (Month/Week/Day views). Timeline grid classes don't exist, so it's misaligned. Fields can't be cleared on edit. Two people saving at once can double-book. Status buttons overwrite others' edits. Cancel is one click with no confirmation. The ".ics" download gives the whole feed. Summary cards count all history, including requests. Reference numbers can collide. | booking components | Daily friction and wrong info. |

## 3. Appointments, staff calendar & meetings

*Live data: 147 appointments, ~140 with no centre, almost none with an assigned staff member, dates stored as free text ("Feb 12-13", "10am", "flexible").*

| Sev | Problem | Where | Impact |
|---|---|---|---|
| **Critical** | Website visit requests create appointments with **no centre, no staff and unparsed dates**, even though the router has already worked out the centre. | `receiveSubmission:65-78` | Centre staff never see these visits, no reminders go out, and they vanish from Timeline and Calendar. |
| **Critical** | The completed-appointment automations fire on *every* edit. Parents get a follow-up email each time, and a Feedback row is created each time. The function crashes when `lead_id` is empty. The feedback WhatsApp is generated but never sent. | `sendAppointmentFollowUp:8`, `appointmentCompletedTrigger:9` | Parents are spammed and no feedback is actually collected. |
| High | Reminder times are read as UTC, so "in a few hours" reminders arrive **after** the visit starts. The job runs every 5 h against a 3 h window. There are no sent-flags. | `appointmentReminders:20-45` | Missed, late or duplicate reminders. |
| High | WhatsApp reminders are sent twice (the 17:00–19:00 window spans two hourly runs; whole-hour rounding). | `whatsappAppointmentReminder:79` | Parents get duplicates. |
| High | Appointments with "10am" or free-text dates are silently dropped from Timeline, the default view. Visitor name and phone show blank when there's no lead. | `TimelineView.jsx:24`, `StaffCalendar.jsx:305` | Staff miss visits. |
| High | The StaffCalendar **Follow-up button is broken**. It's called with no argument, which closes the dialog and throws an error. | `StaffCalendar.jsx:390` | The feature doesn't work. |
| High | Assignment and status changes aren't recorded. The assignee isn't notified. "No show" is saved as "cancelled". There's no staff double-booking check. | `Appointments.jsx:470`, `SmartActions.jsx:88` | No accountability, wrong reports. |
| High | The appointment email fails for "10am" times. Calendar links are 5.5 h late. It needs a lead email even when a visitor email exists. | `sendAppointmentEmail:35` | Email button fails. |
| Medium | Normal staff can't see meeting minutes (read permissions). "My actions" never match because assignees are names, not emails. Two pages share one cache key. | `Meeting` RLS, `TeamMeetings.jsx`, `PublishedMeetings.jsx` | Staff never see minutes or actions. |
| Medium | The Meetings page is locked to two hard-coded emails, **one missing its "@"**. | `Meetings.jsx:62`, `MeetingView.jsx:97` | An admin is locked out. |
| Medium | My Calendar shows **5:30 AM** for date-only items. Save failures are silent. | `Calendar.jsx:118`, `Appointments.jsx:96` | Misleading times, lost edits. |

## 4. Staff logs, daily reports, safety forms & petty cash

*Live data: 206 work logs, **0 daily reports ever**, **1 safety-form submission ever**, **6 audit-log rows ever**.*

| Sev | Problem | Where | Impact |
|---|---|---|---|
| **Critical** | The admin work-log board loads with no limit, so it shows only ~50 of 206 logs. Stats, filters and "This month" are all wrong. Staff's own history is also capped. | `WorkLogAdmin.jsx:26`, `WorkLog.jsx:28` | Admins can't see or review most logs. |
| **Critical** | **There is no audit trail.** Nothing in the app writes ActivityLog for leads, tasks, approvals, logs, reports, forms or petty cash. The audit table may be editable by users. *(confirm)* | whole app, `entities/ActivityLog.jsonc` | You cannot tell who did what or when. |
| **Critical** | Staff can edit or delete their work logs **after admin review**, including overwriting the admin's remarks, rating and flag. | `entities/WorkLog.jsonc` RLS, `WorkLog.jsx:182` | Logs can't be trusted. |
| **Critical** | The weekly compliance grid is shifted a day (Sun–Sat instead of Mon–Sun) for Sri Lanka users. | `formsLogic.js:186-201`, `WeeklySummaryCard.jsx` | Forms show missing/submitted on the wrong days. |
| High | Safety-form autosave marks the form "saved" *before* it has saved and never retries. It shows "Autosaved" even if nothing saved. Submit errors are silent. There's no offline copy. | `OpsFormFiller.jsx:78-94` | Lost form data, especially outdoors with weak signal. |
| High | Duplicate form submissions are easy to create: autosave can create twice, and weekly/monthly forms start a new record on every visit. | `OpsFormFiller`, `OpsFormsToday.jsx:196` | Duplicate or confusing records. |
| High | Submitted or reviewed forms and daily reports can still be edited through the API by anyone at the centre. | `OpsFormSubmission.jsonc`, `DailyReport.jsonc` | Records altered after review. |
| High | **Daily reports are unusable.** Drafts can't be reopened, errors are silent, only branch managers can create but others see the page, duplicates are allowed, and a photo upload can hang forever. | `DailyReport.jsx`, `DailyReportModal.jsx` | Explains the 0 reports. |
| High | The 5:30 PM missing-report reminder miscounts: no limits, monthly forms re-flagged daily, pending users included. | `checkMissingDailyReports` | False reminders. |
| High | The petty-cash balance is summed from only the last 500 entries (or 2000 across all centres). | `CentrePettyCashView.jsx:21`, `HqPettyCashView.jsx:19` | Balances and day-close differences go wrong as history grows. |
| High | Admin rating and flagged filters don't react. Two admins' remarks overwrite each other. | `WorkLogAdmin.jsx:111`, `AdminRemarkModal.jsx:27` | Broken filters, lost remarks. |
| Medium | **"Today" is taken in UTC in 42+ places**, so between 00:00 and 05:30 records get yesterday's date. | daily report, petty cash, forms, tasks, leads | Wrong dates on early-morning entries. |
| Medium | A form draft's date resets to today on every save. The Closing checklist reuses the Opening submission *(confirm)*. Work logs accept future or back dates with no flag. Closed petty-cash days aren't locked. TimeTracker loses time when the modal closes or the phone locks. | various | Data quality. |

## 5. Inquiries, leads, tasks, messaging & automations

| Sev | Problem | Where | Impact |
|---|---|---|---|
| **Critical** | WhatsApp is down (see §0). Escalation ignores send failures and marks records escalated anyway. | `inquiryEscalation:62` | Silent loss of alerts. |
| **Critical** | The public inquiry webhooks have no secret or validation. | `receiveInquiryBridge`, `receiveSubmission` | Anyone can create inquiries and trigger 4 automations each. |
| **Critical** | ~14 alert and briefing functions have **no caller check**. For example, anyone can make the company number send text of their choice to the 10 alert staff. | `whatsappInquiryAlert`, `polaMorningBriefing`, `generateDailyFollowUps`… | Phishing risk, wasted credits. |
| High | Several automations fire on the same new record. A Submission runs Auto-Assign + Router + POLA + WhatsApp, and the two assigners pick different people. | `workflows/*` | Duplicate messages; the wrong person is told "assigned to you". |
| High | Duplicate leads: 20 phone groups, 51 records. Dedupe is an exact string match (`077…` vs `+9477…`). | `receiveInquiryBridge:103` | Same parent contacted repeatedly. |
| High | 190 of 234 leads have no owner. SenSoMo centres have no assignment rules. One submission with status "New" (capital N) will never escalate. | data + `inquiryEscalation:41` | Inquiries fall through the cracks. |
| High | The daily follow-up generator creates duplicate, unassigned tasks and misses the oldest overdue leads. | `generateDailyFollowUps` | Useless task noise. |
| Medium | Monthly and mid-month reports just re-send the daily report. Almost no MessageLog is written. Phone numbers are normalised inconsistently. Lead and submission lists stop at 200, and the Dashboard at 100. | various | Missing history, hidden records. |
| Low | Task recurrence is saved but never acted on. Director contacts are hard-coded. `getSetting()` can create duplicate rows. Nobody uses "Mark attended". | various | Misleading features. |

## 6. Build & overall health

- **Build:** 0 errors. **Lint:** 357 unused-import errors, all auto-fixable. The lint config accidentally turns off core checks, but re-running with them on found 0 real undefined-variable errors.
- **Cache-key collisions (High):** `['tasks']`, `['leads']`, `['events']` and others are shared by pages with different limits and sorting, so lists randomly shrink or reorder when you move between pages.
- **Dashboard (High):** it loads the 100 *latest* due tasks, so old overdue tasks are missing. Lead stats come from 100 leads only.
- **Unbounded backend lists:** 141 calls with no limit. Briefings and reminders may skip staff as the team grows.
- **Silent errors:** 44 catch blocks only log to the console. Many save actions have no error message.
- **Performance:** one 3.6 MB bundle, because all pages load up front. Presence is written every 30 s and cleaned up every 5 min. Inquiries loads 6000 records.
- **Dead code:** 11 unused components, an AI-assistant toggle that does nothing, and two sources of routes.
- **Schedules:** all named times are correct in Colombo time, but the mix of UTC and Asia/Colombo crons is fragile.

---

## Recommended fix plan

**Phase 1: Stop the bleeding (security + lost alerts), ~1 day**
1. Add a secret check to public webhooks and a key/admin check to all internal functions. Fail closed when the key is missing.
2. Escalation and alerts should only mark "sent" on success, and log every send (success or failure) to MessageLog.
3. Disable the duplicate automations so the Inquiry Router is the single source.
4. Fix the completed-appointment triggers so they fire once only.

**Phase 2: Logging you can trust, ~2 days**
1. Record an audit trail server-side (entity-triggered) for bookings, leads, tasks, appointments, work logs, reports, forms and petty cash, including who/when/what changed. Lock the audit table.
2. Lock work logs, forms and reports after submission or review. Move admin remarks out of staff-editable fields.
3. Add paging and limits to WorkLogAdmin, staff history, Leads, Submissions, Dashboard and backend checks.
4. Make daily reports usable: reopen drafts, one report per centre per day, error messages, correct permissions.
5. Make safety-form autosave failure-safe, with an offline draft and no duplicates.

**Phase 3: Timezone & bookings correctness, ~1–2 days**
1. Use one shared Colombo date/time helper and replace all 42+ UTC "today" usages. Fix the weekly grid.
2. Bookings: fix conflict checks, Google sync statuses, idempotency, all-day dates, drawer times, multi-day display, finance validation.
3. Appointments: set centre and staff at creation, parse dates and times, fix reminders and add sent-flags, fix the Follow-up button, add a "No show" status.

**Phase 4: Everyday usability, ~1–2 days**
1. Role-aware mobile bottom bar, access-request flow fixes, auto-refresh on the Pending screen.
2. Error messages on every save, confirmation before cancel or delete.
3. Fix cache keys, lazy-load pages, remove dead code.
4. Merge duplicate leads, add assignment rules for SenSoMo centres, fix the follow-up generator.
