# Content fixes, round 1 (Finnish Pedagogy OS, app 69f71ad98aafbe26ce61e092)

Baseline commit: 2f7381e. Edits auto-commit in the Base44 sandbox. There is no checkpoint and nothing has been deployed.
Checks: `npx eslint --quiet` on all 49 changed JS/JSX files passed. `vite build --outDir /tmp/fpbuild` exited 0; the only output was the existing Tailwind `duration-[…]` ambiguity warnings.
Diff (excluding base44/): 52 files, +143 / -608.

Note: `git diff 2f7381e` also shows changes to 13 `base44/entities/*.jsonc` files (Child, TeacherNote, ParentNotification and others). They were made at 13:20–13:43 in commits that touch only entity files. I did not make them. They look like another person's or agent's RLS/schema work, so check before the next checkpoint.

| ID | Status | File(s) | Notes |
|---|---|---|---|
| T-01 | fixed | src/components/kasvunpolku/control/SundayDigestCard.jsx | Removed "Send test to me" alert button. Preview and Generate still work. I also switched the "Next digest" date to "EEEE d MMMM". |
| T-02 | fixed | src/components/kasvunpolku/control/ParentJourneySection.jsx | Removed "Send to unwelcomed" button. The subline now reads "N not yet welcomed" (dropped "transactional cost minimal"). |
| T-03 | fixed | same | Removed "Send gentle re-engage" button. Invite parent still works. |
| T-04 | fixed | src/components/kasvunpolku/control/TeamProgressTable.jsx | Removed fake "reminder sent" bell button. |
| T-05 | fixed | src/components/children/ChildFamilyTab.jsx | Removed whole Communication Log card (placeholder text plus a "Draft message" button with no handler). |
| T-06 | fixed | src/pages/parent/ParentThisMonth.jsx | Now "No milestones for this month yet." |
| T-07 | partly | src/components/parent/hw/heartwise/ChapterToolbar.jsx | I did not hide the button: with no audio it runs `generateChapterAudio` and prepares audio on demand, so it works. Removed "Audio coming soon" and the muted icon. The label is "Listen", or "Preparing…" while busy. |
| T-08 | fixed | src/components/music/player/FullPlayerDialog.jsx | Removed disabled Share button and the "placeholders" comment. |
| T-09 | fixed | src/pages/parent/ParentSettings.jsx, src/pages/parent/HWMe.jsx | LANGS is now English only. Locale files are kept. The unused `settings.coming_soon` key is still in en.json. |
| T-10 | fixed | src/pages/kasvunpolku/MyAccount.jsx | Removed the si/ta/fi options. |
| T-11 | fixed | src/pages/kasvunpolku/MyAccount.jsx | Removed the dead "Export observations as JSON" row because no real export exists nearby. |
| T-12 | fixed | src/components/onboarding/CalmOnboarding.jsx | Now "The daily structure is built into the platform, so every school follows the same curriculum rhythm." |
| T-13 | fixed | src/pages/TeacherDashboard.jsx | TwoChildrenWidget no longer rendered (file kept). |
| T-14 | fixed | src/pages/TeacherDashboard.jsx | HeartBearScript no longer rendered (file kept). |
| T-24 | fixed | src/pages/SchoolAdminDashboard.jsx | Uses `school.school_name` and `school.city`. The School schema has `city` but no `location`. |
| T-25 | fixed | src/pages/Reports.jsx, src/pages/SuperAdminDashboard.jsx | Page is now just the "Reports & Progress" heading and one honest line. All fake numbers, names, charts and the CSV button are gone. Removed the Quick Actions link. The /reports route is kept. |
| T-26 | fixed | src/pages/ActivityLibrary.jsx | Deleted SAMPLE_ACTIVITIES. Empty list shows "No activities yet." and a failed load shows "Activities could not be loaded. Please try again." |
| T-27 | fixed | src/pages/ChildrenNotes.jsx | Heading is now "Your children" with a real count; "Six children…" removed. Card line uses the real `current_age_band` and `enrolment_year` and is hidden when both are empty. The Child record has only `classroom_id`, so no room name is shown. |
| T-28 | fixed | src/components/children/ChildOverviewTab.jsx, ChildProfileModal.jsx | Hard-coded line replaced with "Enrolled {enrolment_year}", shown only when set. |
| T-47 | fixed | src/pages/ChildrenNotes.jsx | Empty state "No children to show yet." |
| T-29 | fixed | src/pages/ForestSchool.jsx | "Spring / Summer", "Spring / Autumn" and "Autumn / Winter" are now "All Year", matching existing tags. I did not map them to a Term: that would be a guess. |
| T-45 | fixed (tag only) | src/pages/ForestSchool.jsx | Fire session is now tagged "All Year". Content is unchanged. It still needs a curriculum-lead safety review. |
| T-30 | fixed | src/pages/ActivityLibrary.jsx | Removed along with the sample activities. |
| T-31 | fixed | src/lib/parent/useDailyMomentPhoto.js, src/components/parent/hw/today/DailyMomentCard.jsx | SeasonalMoment query and currentSeason() removed. It returns null when there is no recent teacher photo. Kicker now "A quiet moment". |
| T-32 | fixed | src/components/kasvunpolku/parent/SeasonHero.jsx | Labels changed from "…season" to "…stage" (e.g. "the first-light stage"). Code identifiers unchanged. |
| T-33 | fixed | src/pages/parent/ParentMilestoneLibrary.jsx, ParentFirstMoments.jsx | "Every child grows at their own pace." and "Every first comes in its own time." |
| T-34 | fixed | src/lib/songCategories.js | Label "Weather and nature" 🌦️. The key `concept_seasons` is unchanged. |
| T-35 | fixed | src/components/layout/Sidebar.jsx | "Programmes" (x2). |
| T-36 | fixed | src/components/canvas/ProgramSegment.jsx, CanvasMonthGrid.jsx | "Morning Programme" and "Evening Programme". |
| T-37 | fixed | src/pages/Schools.jsx, src/components/schools/EditSchoolDialog.jsx, src/lib/policies.js | "Licence plan", "Licence and limits", "Licence start", "Licence end", "licensing and inspection bodies". |
| T-38 | fixed | src/locales/en.json | "practised". |
| T-39 | fixed | src/components/finniped/OrganizeLibraryButton.jsx | "Organise with AI", "Organising…", "Library organised", "Couldn't finish organising". |
| T-40 | fixed | src/pages/kasvunpolku/MyChild.jsx, src/components/dashboard/WeatherCard.jsx, src/lib/languageGuard.js | "towards". |
| T-41 | fixed | src/components/dashboard/FestivalBanner.jsx | "acknowledgement". |
| T-42 | fixed | VoiceCatalogEditor.jsx, StoryAlbumEditor.jsx (x2), MusicProviderCard.jsx | Visible "voice catalogue" only. Identifiers and the `voice_catalog` field are unchanged. |
| T-43 | fixed | TeacherDashboard, HeroHeader, ParentPhotoAlbum (x2), ParentFirstMoments, MilestoneDetailCard (+2 "d MMM"), SummaryHero, SharedCalendars, ParentCalendars, MyReflections | "EEEE d MMMM yyyy" and "d MMM yyyy". |
| T-44 | fixed | src/components/parent/hw/sos/SOSOverlay.jsx | Fixed footer on every step with the 1990 and 1929 line. "Find Your Way Back" was not changed because it was not in scope. |
| T-46 | fixed | src/pages/curriculum/EveningCurriculumHome.jsx | Added .catch. Shows "Evening age groups could not be loaded. Please try again." on error and "No evening age groups set up yet." when the list is empty. |
| T-49 | fixed | index.html | lang="en-GB", title "Finniped · Finnish Pedagogy OS", meta description, og:title, og:description, apple-mobile-web-app-title "Finniped". I did not add og:image or og:url. The favicon is still the Base44 logo (https://base44.com/logo_v2.svg). At runtime, RouteTitle.jsx and HWRouteTitle.jsx replace document.title with their own BRAND ("Finnish World School" and others), so the new title shows only before the app loads. |
| T-50 | fixed (name) | public/manifest.json | name and short_name are "Finniped". Icons are unchanged: they still use the AI `generated_image.png` for both 192 and 512 and need proper icons. |
| T-53 | fixed | src/pages/TeacherDashboard.jsx | "Good evening" from 17:00. |

## Other changes

- Lint cleanup: `eslint --fix` removed unused imports that were already in files I touched: FestivalBanner, Sidebar, ActivityLibrary, SchoolAdminDashboard, Schools, SuperAdminDashboard, TeacherDashboard and MyAccount. Only import lines changed.

## Not in scope, noticed in passing

- Teacher dashboard "Print" button (Today's Rhythm) has no onClick.
- ChildFamilyTab "Family Contributions" card is a static placeholder.
