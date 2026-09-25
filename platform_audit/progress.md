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
