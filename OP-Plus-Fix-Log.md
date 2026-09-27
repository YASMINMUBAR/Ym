# OP+ Opportunity Plus: Fix Log

- **App:** Base44 `696e822e6bb318f6b83d3652`
- **Dates:** 25–27 September 2026
- **Findings fixed:** see `OP-Plus-Audit-Report.md`
- **Restore points in Base44 version history:**
  - "Before audit fixes (baseline 25 Sep 2026)" is the original app.
  - "Audit fixes + final review fixes …" is the current version.

**Status:**

- **Verified before saving:** the app builds with 0 errors, every backend function and shared file compiles, and every data-table and automation file is valid.
- **Reviewed:** three independent reviews covered access and permission rules, backend functions and automations, and the most-used pages. No critical issues remained, and every medium finding was fixed.
- **Not live yet:** nothing has been published. The changes go live when **Publish** is pressed in the Base44 editor.

## What changed

### Foundations (new)

- **Sri Lanka time helpers** (`src/components/utils/timezone.jsx`, `base44/shared/dates.ts`). Every "today" and every date/time now uses Asia/Colombo. About 45 places used UTC, which stamped early-morning entries with yesterday's date. Platform timestamps are read correctly as UTC.
- **Backend security check** (`base44/shared/guard.ts`). Scheduled and internal functions accept only the internal key or an admin, and refuse the call if the key is missing. Functions started by a record change re-read that record from the database instead of trusting the incoming data.
- **Audit trail:**
  - The `auditTrail` function plus 13 "Audit Trail — …" automations record every create, edit and delete of bookings, appointments, leads, submissions, tasks, work logs, daily reports, ops forms, ops issues, petty cash entries, day-closes, purchase requests and stock transfers.
  - Each entry records who, what changed (old → new) and when.
  - The app stamps `last_modified_by` / `last_modified_at` on every write, so changes are attributed to the right person. Anything else is labelled "Automation".
  - Only the automations can write entries, and admins cannot edit them.
  - New **Activity Log** page for admins and operations managers.
- **Pop-up messages now appear.** About 30 screens' error and confirmation messages were never displayed; they are now.

### Login and access

- New staff can see the centre list and submit their access request.
- Base44 admins with no access level are treated as super admin.
- Remove / Decline now actually revokes centre access.
- Declined users can edit and resubmit their request.
- Pre-assigned access is applied on invite and first login, and email matching ignores case.
- The Pending screen refreshes itself.
- The mobile bottom bar only shows pages the person is allowed to open.
- Network errors show a Retry screen instead of a blank page, and a brief connection drop no longer logs anyone out.
- The Team table has permission rules.
- Staff pickers use the team list, so branch managers can see their own team.

### Staff logs, reports, forms, petty cash

- **Work logs:**
  - The admin board filters by date range with no silent limit, and shows a "Logged late" badge.
  - Logs lock after admin review; staff can no longer change them or the admin's remarks.
- **Daily reports:** drafts can be reopened, there is one report per centre per day, errors are shown, and a report locks once reviewed.
- **Safety forms:**
  - Autosave only reports "saved" after a real save, and retries on failure.
  - A copy is kept on the phone.
  - No duplicate submissions; weekly and monthly forms reopen the existing submission.
  - A form locks once submitted.
- **Weekly compliance grid:** now correctly Monday–Sunday.
- **Petty cash:** the balance carries forward from the last day-close, closed days are locked, and rejected expenses don't reduce the balance.
- **Time tracker:** survives closing the window or locking the phone.
- **Missing-report reminders:** counted correctly, and only approved staff are notified.

### Bookings

- The clash check uses Sri Lanka time (it was 5.5 hours off), and two people can't book the same slot at the same moment.
- Google Calendar:
  - Requested bookings are never synced. Tentative bookings sync as tentative; confirmed, in-progress and completed as confirmed.
  - No duplicate events.
  - All-day events land on the correct date.
- Multi-day bookings appear on every day they cover. The Timeline grid is fixed.
- Finance: currency is chosen from a list, a price is required, and negative values are rejected. The Jat Holdings booking figure is corrected (gross LKR 2,187,500).
- The public request form limits repeat requests and returns a clear error for impossible dates.
- Cancelling needs a reason. Reminders are sent once, and past bookings are marked completed automatically.

### Appointments and meetings

- Website visit requests now reach the right centre, with real dates and an assigned staff member where one is known.
  - 105 existing appointments were given their centre.
  - 48 free-text dates were converted to real dates.
- Follow-up emails and feedback requests go out once, not on every edit.
- Reminders use Sri Lanka time, go out once, and are logged.
- Appointments with no readable date appear in a "Needs a date" group instead of disappearing.
- The Follow-up button works. There is a new "No show" status.
- Staff pickers show the whole team. Assigning staff warns about clashes and notifies the assignee.
- Meeting minutes are visible to staff. The hard-coded email check (one address was missing its "@") is replaced by a role check.

### Inquiries, leads, tasks, messaging

- **Public inquiry webhooks:** input is validated, duplicates are dropped, and an optional `WEBHOOK_SECRET` can be set.
- **Duplicate alert automations switched off:** 5 of them, after live data confirmed the Inquiry Router handles both leads and submissions.
- **Alerts and escalations:** marked as sent only when a message actually goes out. Failures are recorded and retried.
- **WhatsApp:**
  - One shared sender. Every message, sent or failed, is logged in MessageLog.
  - Group sends go through the queue, so long lists aren't cut off.
- **Reports:** the monthly and mid-month reports now cover their proper periods. The follow-up generator no longer creates duplicate or unassigned tasks.
- **Recurring tasks:** they now actually recur (daily at 6 AM Colombo).
- **Lists:**
  - Leads, Submissions, Inquiries, Tasks and the Dashboard no longer silently cut off records.
  - Leads show a "Possible duplicate" badge, and mark-attended is clearer.
- **Notifications:** mark-all-read works, and the unread badge updates.
- **Staff status:** the online indicator writes less often, so it costs less.

## What the owner needs to do

1. **Press Publish** in the Base44 editor so these changes go live.
2. **Reconnect the Chatbiz WhatsApp sender (94761828838).** Every WhatsApp has failed since 23 Sep because that number is disconnected.
3. **After publishing, spot-check:**
   - as a centre staff user, create and edit a work log, save and submit a daily report, and fill in and submit a safety form
   - make a test booking
   - check the Activity Log page shows the changes
4. **Optional:** set a `WEBHOOK_SECRET` secret and send it from the website forms as the `x-webhook-secret` header.
5. **Security:** the internal function key sits in the automation files. If the app code is shared outside the team, rotate `INTERNAL_FUNCTION_KEY` and update the automations.
6. **Manual data review:**
   - 33 appointments with no centre
   - 26 appointments with unclear dates (e.g. "Feb 12-13")
   - 1 legacy booking (6ab39f5f…) with start = end
   - 1 requested booking (6ab62acf…) that still has an old Google Calendar event; press "Sync now" on it to remove it
7. **Approve the two users stuck on "approved + pending"** (they now appear under Access Management → Requests).
