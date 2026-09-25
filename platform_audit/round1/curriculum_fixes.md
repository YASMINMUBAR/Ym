# Curriculum fixes, round 1

App 69f71ad98aafbe26ce61e092 (finniped.com), sandbox `/app`. Code changes only: no checkpoint, no deploy, no builder messages, no schema, RLS or entity-record changes. The baseline commit before this pass is `b522330`, so `git diff b522330` shows every change (32 files).

Checks:
- `npx eslint --quiet <32 changed files>`: passes with 0 errors. Unused imports that were already in the touched files were removed with `--fix`.
- `vite build --outDir /tmp/fpbuild`: passes (exit 0). The only output is the existing Tailwind "ambiguous class" and Browserslist warnings.

## New shared files
- `src/lib/termLabel.js` exports these helpers:
  - `termOf`, `termWeek`, `termLabel`, `termMonthLabel`, `isConsolidation`, `monthName`
  - `notArchived(rows)`, which removes rows with `is_archived === true`
  - `CONSOLIDATION_LINE`

  Term 1 is months 1–4, Term 2 is 5–8 and Term 3 is 9–12. The week number is `(month index in term)*4 + week_number`, capped at 4 per month. April, August and December get " · Consolidation week".
- `src/components/curriculum/LoadError.jsx` shows "This page could not be loaded. Please try again." with a "Try again" button that calls `window.location.reload()`.

## Task 1: exclude archived months and weeks. Done
Rows are filtered on the client after each fetch (`notArchived` or `!r.is_archived`). Server query operators are not used for this.
- **Curriculum pages:**
  - `CurriculumTier.jsx` (limit raised from 50 to 200 so duplicates cannot crowd out real months)
  - `CurriculumMonth.jsx` (month and weeks)
  - `CurriculumWeekPage.jsx`
  - `CurriculumDayPage.jsx` (archived week/month are ignored for labels)
  - `CurriculumCalendar.jsx` (limit raised to 200)
  - `DailyPlan.jsx` (months, morning weeks, evening week)
  - `EveningTermPage`, `EveningYearOverview`, `EveningWeekPage`, `EveningDayPage`
- **Dashboards and parent pages:** `lib/dashboard/useTeacherToday.js`, `lib/parent/useParentChildren.js`, `useParentDashboard.js` and `useActiveChild.js`. `useActiveChild` now fetches 50 rows and filters them instead of taking 1.
- **Calendar exports:** `lib/calendarContent.js` covers weekly, monthly, termly, yearly and evening. `.get()` returns null when the row is archived.
- **Admin tools:** `CurriculumCanvas.jsx`, `BulkCurriculumGeneration.jsx`, `CurriculumOrchestrator.jsx` (counts) and `UltimateEnhanceModal.jsx` are filtered too. Archived rows should not be counted or targeted.
- **Other components:** `finniped/LibraryByWeek.jsx`, `kasvunpolku/control/ParentFeedHealth.jsx`.
- **Backend functions:** `exportCurriculumOutline`, `prepareNextWeekDailyPlans` (limit raised to 200) and `linkResourcesToWeeks`.
- **Not changed:**
  - `parentRecommendations` and `onParentActivityCompletion` read only the single highest week_number. That is an evening week, and evening weeks are not archived.
  - Generator functions (`generateCurriculum*`, `orchestrateCurriculumFull`, and similar) create records and do not list them for display.

## Task 2: term labels. Done
- **Tier:** each month card shows "Term N · Weeks a–b" (plus " · Consolidation" for April, August and December), then the theme, with the month name as small text.
- **Month:** the header chip shows "Term N · Weeks a–b" with the month name beside it. Each week card shows `termLabel` (for example "Term 2 · Week 7") with "June · week 3" as small text.
- **Week:** the header chip shows `termLabel`, for example "Term 1 · Week 13 · Consolidation week", with "April · week 1" as secondary text.
- **Day:** `DayHero` has a new optional `termText` prop that shows the term label before the date. The date is now secondary.
- **Calendar:** the month heading is "Term N · Weeks a–b — theme" with the month name underneath. The "Weekly Plans" heading and the week badges use term labels, with the month and week as small text.
- **DailyPlan (morning):** the header shows the term label, then the theme, then the month in small text. The intention card shows "Term N · Week k · title".
- **FinnipedWeekPanel:** when the month is April, August or December and no placements exist, it shows the line "Consolidation week: revisit favourite sessions from this term."
- Week titles stored in data (for example "January W1 · Open") are left unchanged. They are record content, not labels.

## Task 3: evening programme. Checked, no change
`programType.js` EVENING_TERMS and the evening pages contain no season words. **Needs a decision:** the evening programme still uses 4 terms of 13 weeks, including "Term 4 · Completing & Celebrating", and counts calendar weeks from 1 January. It does not match the morning Term 1–3 rule.

## Task 4 (C-12): error states. Done
- **Pages with a new error state:** Week, Month, Tier, Calendar (age groups, months and weeks), DailyPlan, all four Evening pages and Printables. Each fetch now has a `.catch` or `try/catch` that shows `LoadError` (with reload). DailyPlan shows an error for the first load and an inline error card when a day fails to load.
- **FinnipedWeekPanel:**
  - Load errors at :47 and :67 (and the resource list) now set an error flag. The panel shows a quiet italic line: "Some curriculum files could not be loaded. Please refresh the page to try again."
  - A placement whose book has no `file_url` now shows "File not available yet" instead of hiding the View button.

## Task 5: DailyPlan week fallback. Done
If no day matches the date exactly, the page uses the week whose `week_number` equals `ceil(day/7)`, capped at the highest week number in the month. The "week 1" fallback and the "any week with this weekday" fallback are both gone. If there is still no day, the page shows: "There is no day plan for Friday 25 September (Term 3 · Week 4) in "<theme>" yet." The week is kept, so labels and the Finniped panel still use the correct week_in_month. The same fix is in `useTeacherToday.js`, which had the same week-1 fallback.

## Task 6: merged age groups. Done, with one caveat
The DailyPlan picker now leaves out AgeGroups with `status === "merged"`. **Caveat:** all 4 evening tiers have `status=merged` in the data (see notes §1). Excluding them would leave the evening Daily Plan empty. So evening merged tiers are hidden only if at least one non-merged evening tier exists; today they all still show. The evening tier status needs fixing in data (decision or data task).

## Task 7 (C-01): generator prompt. Done
In `base44/functions/generateCurriculumMonth/entry.ts`, the context line now says:
- Sri Lankan school Term N (Jan–Apr, May–Aug, Sep–Dec)
- Colombo is in the Northern Hemisphere and tropical, with no spring, summer, autumn or winter
- the south-west monsoon runs about May–September and the north-east monsoon about December–February, with inter-monsoon rains
- local festivals: Thai Pongal, Sinhala and Tamil New Year (Avurudu) in April, Vesak, Poson, Deepavali and Christmas

The `theme`, `nature_focus` and `festival_observances` fields and the closing line ("Be local to Sri Lanka… never use spring/summer/autumn/fall/winter") replace the old seasonal instructions. The rest of the function is unchanged.

## Task 8: leftovers. Done
- `TeacherDashboard.jsx`: the Print button next to "Today's Rhythm" did nothing and has been **removed**. The dashboard has no print layout. The "Full Plan" link beside it opens `/daily-plan`, which has a working Print button.
- `children/ChildFamilyTab.jsx`: the static "Family Contributions" placeholder card showed no data and has been **removed**, along with the unused `MessageCircle` import.

## Open items and decisions
1. Evening terms: the evening programme still uses 4 × 13 weeks and a "Term 4" (task 3).
2. The evening AgeGroups are all `status=merged`, so the picker exclusion does not apply to them yet (task 6).
3. `CurriculumWeek.term_number` and `term_week` are still not stored. Labels are calculated in `termLabel.js` from month and week_number.
4. Season words remain in src outside the curriculum pages, as listed in notes §4:
   - `ForestSchool.jsx`, `ParentThisMonth.jsx`, `DailyMomentCard.jsx`, `useDailyMomentPhoto.js`, `CanvasGeneratePanel.jsx`, `BulkCurriculumGeneration.jsx:52`
   - the `generateMonthPrintablePack` prompt

   None of these were in scope for this pass.
