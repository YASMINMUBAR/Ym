# Finniped hardening: progress

App: Base44 69f71ad98aafbe26ce61e092 (finniped.com)
Plan: three rounds of audit -> fix -> verify. Approved by Yasmin on 2026-09-25.

## Round 1
- [x] Base44 checkpoint "Before platform hardening audit (Round 1)"
- [ ] Area audits (roles, library, curriculum, content/dashboard, technical)
- [ ] Merge findings into 02_findings.csv
- [ ] Fix batch 1 (critical)
- [ ] Fix batch 2 (major)
- [ ] Verify

## Blockers
- FinnipedBuild/ folder not in this repo (needed for book rebuilds).
- No test logins per role yet (needed to prove data privacy between schools/families).

### Update (Round 1)
- [x] Five area audits done: round1/{roles,library,curriculum,content,technical}.csv (139 findings: 9 critical).
- [x] Merged into 02_findings.csv, ranked, with a status column.
- [x] Batch 1 fixed + Base44 checkpoint "Round 1 batch 1": RoleRouteGuard (parents -> /parent, no-role accounts only see "/", uploaders admin-only, /jobs admin-only); Library band filter keeps all-band books; favourites star follows replaced books; plain viewer contents jump + error note; download opens new tab; load error message.
- [ ] Batch 2 (content: placeholders, fake data, seasons, British spelling, SOS numbers, SEO) — in progress.
- Needs Yasmin: RLS on child data (R-01..R-04), safeguarding gate for hw1 body-safety (L-02), re-upload 13 files (L-01), duplicate curriculum months (C-03..C-06), 7 wrong-hemisphere months (C-02).
- [x] Yasmin approved (2026-09-25): child-data RLS, hw1 body-safety gate, archive duplicate months, Term N · Week k labels.
- [x] RLS applied on Child, TeacherNote, Classroom, ParentObservation, FamilyEcho, ParentNotification, WeeklyGrowthReport, ParentMessage, ChildGrowthRecord, ChildMilestoneObservation, ChildPortfolioEntry. Role "user" removed from all create/update lists.
- [x] hw1 safeguarding: 8 FinnipedResource + 6 FinnipedPlacement set safeguarding_gated=true, audience=teacher_parent_only.
- [x] Added is_archived to CurriculumMonth/Week; archived 55 surplus months, their 220 weeks, and 20 duplicate January weeks (is_master=false). Nothing deleted.
- [ ] Code: curriculum pages filter is_archived, Term N · Week k labels, loading/error states, generator prompt (C-01).
- Known gap: parents can still read ParentMessage/growth/milestone/portfolio/TeacherNote rows of other children via the API (no parent field on those rows). Round 2: add parent_user_ids and backfill.
- Known gap: School Admin account has no school_id, so it now sees no children.
- [x] Batch 2 content fixes done + checkpoint 'Round 1 batch 2'. See round1/content_fixes.md.
- [x] Batch 3 curriculum code done + checkpoint 'Round 1 batch 3'. See round1/curriculum_fixes.md. ROUND 1 COMPLETE.

## Round 2
- [ ] Security/routing/speed fixes (backend function auth, sidebar dead links, lazy-loaded pages, /parent/admin removed) — in progress.
- [ ] Remaining curriculum/library/dashboard/season fixes — in progress.
- [ ] Round 2 re-audit (read-only) after both finish.
- [x] Yasmin gave standing approval for all edits (2026-09-25); still no deleting, no publishing, no passwords.
- [x] L-06: archived 5 older Numenature books. L-05: moved 14 Bodywise/Artelier books to the age band in their titles.
