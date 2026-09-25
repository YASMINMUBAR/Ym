# Curriculum audit, round 1: notes

App 69f71ad98aafbe26ce61e092 (finniped.com). This was a read-only audit: no files, entities, builder messages or checkpoints were changed. All counts come from paginated `query_entities` pulls (sorted by id) analysed with jq on 2026-09-25. File and line references point to the app sandbox (`/app`). The findings table is in `curriculum.csv` (C-01 to C-23).

## 1. Record counts and link integrity

| Entity | Records | Notes |
|---|---|---|
| AgeGroup | 14 | 5 canonical morning tiers, 5 merged morning duplicates (69f724cc…), 4 evening tiers all set to `status=merged` |
| CurriculumMonth | 115 | 60 (band, month) slots; 49 slots have duplicates, 55 records are surplus; all is_master=true; years 2025/2026/2027 are mixed |
| CurriculumWeek | 532 | 480 morning (all term_number null) + 52 evening (term_number 1–4, 13 weeks each) |
| CurriculumDay | 2,870 | 398 weeks have exactly 5 days; 82 weeks have 10/15/25 days |
| EveningSession | 208 | 52 weeks × 4 evening tiers, complete, but not read anywhere in src |
| EveningDay | 688 | 64 duplicates; evening_session_id empty on all 688 |
| Activity | 150 | 60 have status 'master' (off-enum); 10 have no age group; 1 is on a merged tier |
| Printable | 129 | 10 published with no file; 8 'Sample' drafts; 52 Tiny Roots month-pack rows (4 identical packs) |
| CalendarTheme | 6 | 2 use season wording |
| FinnipedResource | 440 | 182 published / 132 draft / 126 archived; 1 test row |
| FinnipedPlacement | 1,440 | 720 Numenature (import 2026-09-24) + 720 HeartWise (import 2026-09-25); all published |

Broken links checked. **None found** for:
- week.curriculum_month_id pointing to a missing month (0/480)
- day.curriculum_week_id pointing to a missing week (0/2,870)
- EveningDay.curriculum_week_id pointing to a missing week (0/688)
- placement.resource_id or resource_row_id pointing to a missing or archived resource (0/1,440; all 20 used resources are published and have a file_url)
- FinnipedResource.linked_week_ids pointing to a missing week (0 of 277 links)
- archived resource superseded_by pointing to a missing resource (0)

Test/sample records:
- FinnipedResource 6ab566dac64b719620a698cc "ZZ schema test (delete me)" (archived)
- Printable drafts 69f9b422a02bd71b58ccc26b … 69f9b42928c02c248bf70997 ('Sample Daily Plan', 'Phonido One-on-One Kit (Sample)', and others)
- None in months, weeks, days or placements. CurriculumDay titles such as "Testing with Water" are real content.

## 2. Numenature / HeartWise coverage per band

- Each of the 5 bands × 2 programmes has **36 week slots**, each with 3 session plans + 1 workbook page. That gives 108 + 36 = 144 placements per band per programme, or 1,440 in total. **There are 0 gaps inside the 36 slots.**
- The slots sit in **months 1, 2, 3, 5, 6, 7, 9, 10, 11** × week_in_month 1–4. This is *not* "months 1–9" as the brief says.
- April, August and December have weekly plans in CurriculumWeek (4 per band per month) but **no placements**. That is 12 weekly plans per band (60 in total) with no "This week in Heartwise/Numenature" panel. Examples: Tiny Roots April W1 69f7905a9c96d96ac1f4bf41; Little Sprouts April W3 69f7905a9c96d96ac1f4bf68.
- Every band has 48 unique (month, week) weekly-plan slots. There are extra duplicate records (Tiny Roots 124, Pre-Primary 68, others 96) because of the duplicate months and weeks.

## 3. Placements and "open at page"

- All 1,440 placements have page_from, page_to and of_pages.
- 0 have page_from/page_to greater than the resource's page count, 0 have page_to < page_from, and of_pages always equals FinnipedResource.pages.
- Page ranges never overlap within a book. session_no runs 1–108, contiguous per band.
- How the link is built:
  - `FinnipedWeekPanel.jsx:57` filters placements by (age_group_id, month, week_in_month, published).
  - The "View p.N" button (:123-125) passes `_page=page_from` to `PdfViewerDialog`.
  - That opens `FlipBook startPage` (1-based, `fws/components/FlipBook.jsx:51`) and prints with `file_url#page=N`.
  - This works when data is present.
- Weak points:
  - Fetch errors are swallowed (:47, :67).
  - With no file_url the button silently disappears (:121).
  - The resource table is fully listed on each view (:64).
  - DailyPlan's fallback to week 1 (DailyPlan.jsx:192-197) sends the wrong week_in_month.
  - The Day page has no panel.
  - The panel is keyed on `tierId`/`selectedAgeGroup.id`, so a merged tier id gives nothing.

## 4. Wrong-hemisphere seasonal content (the main cause of season labels)

`base44/functions/generateCurriculumMonth/entry.ts:165` tells the AI: *"Southern Hemisphere — Sri Lanka / India / Australia / NZ"*, and :180/:183 ask for seasonal themes. The records it produced:

**7 CurriculumMonth (all Tiny Roots, 69f71f51e0cdaa01b360774a):**
- 69f83a4c73d928e7025d9a40 (Jan 2025 "Whispers of Summer…", mentions the NZ summer)
- 69f9b957c946f5452e99f020 (Feb 2026 "…The Last Days of Summer")
- 69f9bbf7ed9a20f525226552 (Jan 2027 "Summer Shade…")
- 69f9bc74dd185ff25ffd7116 (Feb 2027 "Sun-Warmed Seedlings", late summer)
- 69f9bcef96a092e3c7ace2bb (Mar 2027 "Autumn arrivals…", Autumn Equinox)
- 69f9c4a0eff18e22049f73bb (Apr 2027 "First Whispers of Autumn")
- 69f9c52654ec545fa81c50ed (May 2027, "cooler autumn mornings")

**28 CurriculumWeek under them:** 69f83a4c96e97cbbbaa8e5af, 69f83a4e5a150aaae8fc77c9, 69f83a511bc2a4b60eaa40ef, 69f83a51e74e165b0ff3b536, 69f9b9574bda4dc7f2a28d00, 69f9b9578cf7fa3ff3b70a5c, 69f9b957ade17fc66143b722, 69f9b957c1f08d64a8d3d99b, 69f9bbf71721a43fbe40f820, 69f9bbf7605167040ef4c6f5, 69f9bbf7895de1a41a22d945, 69f9bbf79116eedca1c29c2e, 69f9bc742e3716c5d07229dc, 69f9bc7432c34c2152994d2e, 69f9bc745e00b5c07474794f, 69f9bc749e73802dea42f31d, 69f9bcefb8767b1d1fde1926, 69f9bcefc3db92f066b776d1, 69f9bcefcf0d9305a8bd1b2f, 69f9bcefdbc679813c48a787, 69f9c4a016d587f65a6cfc36, 69f9c4a0835576e8f871f1fc, 69f9c4a0ba8e21972a20a1f9, 69f9c4a0d14be0f0f02ce0e8, 69f9c527a4e45d2aec8d297a, 69f9c527acbdc24a8c602d77, 69f9c527cccf9ff18e63272f, 69f9c527e4290dbd65b79330

**140 CurriculumDay** (20 per month; dates 2025-01, 2026-02, 2027-01 to 2027-05) sit under those weeks.

**59 Activity records** have theme_id set to those months: 69f83a5352432432b8214838, 69f9b957cb449c51e29b399f, 69f9b9581397b97c54f13acc, 69f9b958956e55065dc71bd5, 69f9b9589be0fdace16ec4d6, 69f9b958c62e53b306b1d6c1, 69f9b958debf6b74a429cd93, 69f9b958edf46247cf331633, 69f9b959043b5797be8cbd24, 69f9b959b2c33bc5f670dc72, 69f9b959dbd0b63ab98f2a81, 69f9bbf710fc6f5a728d8ab5, 69f9bbf710fc6f5a728d8ab6, 69f9bbf728c7d891634712d2, 69f9bbf7352fcf54177cc17b, 69f9bbf7589ea3687bfeeb54, 69f9bbf76cefcd3937917919, 69f9bbf780b21410e1d3efc7, 69f9bbf7978d677bc825ce79, 69f9bbf79de932cf34b0590f, 69f9bbf7c5715aa3c5534161, 69f9bc740f89f937a8b1854f, 69f9bc7417c8259d15daa3f1, 69f9bc741cec463a5fbe6a71, 69f9bc742e3716c5d07229dd, 69f9bc7430fd9e8c9662aa5b, 69f9bc74476a178b1c5f6186, 69f9bc74617d63387d0c4d34, 69f9bc7491a9d834a03e9cea, 69f9bc74da493bb4d7a7b7c6, 69f9bc752fae4c37cdc2c64e, 69f9bcef17c8259d15daa40d, 69f9bcef2e3716c5d0722a10, 69f9bcef2fae4c37cdc2c679, 69f9bcef30fd9e8c9662aa70, 69f9bcef44c91bd6fd4ce49e, 69f9bcefdbc679813c48a788, 69f9bcefdbf592ac94f18471, 69f9bcefe1a5e5f0d1ab1944, 69f9c4a100a1b4a39a8e2b9f, 69f9c4a1344362972ec8475c, 69f9c4a1382ac100aae1af62, 69f9c4a1428d335a45c6a5bc, 69f9c4a1428d335a45c6a5bd, 69f9c4a15169ff71bf01f5fb, 69f9c4a17384aa79a8572fef, 69f9c4a1dd03682c442d2fde, 69f9c4a1dec0ed755423f0e2, 69f9c4a27384aa79a8572ff0, 69f9c52818b0ab5dd1e1a568, 69f9c52857d8115fac32b688, 69f9c52862c6b4a11b09e677, 69f9c52862eca36972b74c10, 69f9c52881a0fe56b2cf928c, 69f9c5289b6f8b460b72bcdc, 69f9c528a6fe5e262b5bcb55, 69f9c528a9f842b11a5d99f1, 69f9c528e4290dbd65b79331, 69f9c528f32c5a1d7a0151e1

**Other season words in canonical content, to rewrite:**
- Months:
  - 69f78aae5ea32b793e9f1fad "light spring breeze"
  - 69f7201a66cfacfd1a59487f "new year season"
  - 69f79e1303a060d3a4f915a4 "seasonal change"
  - 69f79eb503a060d3a4f915d3 "Four seasonal tree drawings"
- Week: 69f7905a9c96d96ac1f4bf43 "soft spring breeze"
- Days:
  - 69f848da465fa7b59b4bbc7e "What Was I Like in Autumn?"
  - 69f84bf8d54aee47b95c519c "Bodies in the Winter Garden"
  - 69f85c5530cde88bf13c35a6 "Soft Closing in the Spring Garden"
  - SH days 69f9b991a61520c2f808323d, 69f9b992064dbdc45cd51133, 69f9be3d2cd168160443d7b8, 69f9be4ab1b321cb48f47eb0 ("Summer")
- CalendarTheme: 6a05b4813b9a85b9d9ff7f76 ("every season"), 6a05b4813b9a85b9d9ff7f77 ("Spring terms… Spring and birthday weeks")
- Activity: 69f75709c60f6b930412bf39 "Leaf Printing with Autumn Colours". The same text is hard-coded in ActivityLibrary.jsx:78.

These uses of "fall" are verbs, not seasons, and need no change: placements "Friends sometimes fall out" (6ab6482db8c8adf88a3899fc, 6ab6486bff9662dd6afa6a91, 6ab6486c3f6553406d125d62) and days "Why Did It Fall?" and similar.

**Season words in src:**
- ForestSchool.jsx:55,70,85 ("Spring / Summer", "Spring / Autumn", "Autumn / Winter")
- ParentThisMonth.jsx:103 ("Milestones for this season")
- DailyMomentCard.jsx:28 ("A quiet season")
- useDailyMomentPhoto.js:73-78 (Northern-Hemisphere meteorological seasons)
- CanvasGeneratePanel.jsx:145
- BulkCurriculumGeneration.jsx:52
- generateMonthPrintablePack/entry.ts:26
- The kasvunpolku/* "seasons" are an age-stage metaphor, outside curriculum scope.

## 5. Duplicate records

**Canonical month set.** The 60 records created 2026-05-03 10:14 (Bright Bloomers) and 17:5x (other bands) cover all 60 slots exactly once. FinnipedPlacement.app_pillar was matched against these records.

Keep: 69f7201a66cfacfd1a59487c, 69f7201a66cfacfd1a59487d, 69f7201a66cfacfd1a59487e, 69f7201a66cfacfd1a59487f, 69f7201a66cfacfd1a594880, 69f7201a66cfacfd1a594881, 69f7201a66cfacfd1a594882, 69f7201a66cfacfd1a594883, 69f7201a66cfacfd1a594884, 69f7201a66cfacfd1a594885, 69f7201a66cfacfd1a594886, 69f7201a66cfacfd1a594887, 69f78aae5ea32b793e9f1faa, 69f78aae5ea32b793e9f1fab, 69f78aae5ea32b793e9f1fac, 69f78aae5ea32b793e9f1fad, 69f78aae5ea32b793e9f1fae, 69f78aae5ea32b793e9f1faf, 69f78aae5ea32b793e9f1fb0, 69f78aae5ea32b793e9f1fb1, 69f78aae5ea32b793e9f1fb2, 69f78aae5ea32b793e9f1fb3, 69f78aae5ea32b793e9f1fb4, 69f78aae5ea32b793e9f1fb5, 69f78b0e4ed61d3b1fb3ec3c, 69f78b0e4ed61d3b1fb3ec3d, 69f78b0e4ed61d3b1fb3ec3e, 69f78b0e4ed61d3b1fb3ec3f, 69f78b0e4ed61d3b1fb3ec40, 69f78b0e4ed61d3b1fb3ec41, 69f78b0e4ed61d3b1fb3ec42, 69f78b0e4ed61d3b1fb3ec43, 69f78b0e4ed61d3b1fb3ec44, 69f78b0e4ed61d3b1fb3ec45, 69f78b0e4ed61d3b1fb3ec46, 69f78b0e4ed61d3b1fb3ec47, 69f78b759aa53b04cb8d795d, 69f78b759aa53b04cb8d795e, 69f78b759aa53b04cb8d795f, 69f78b759aa53b04cb8d7960, 69f78b759aa53b04cb8d7961, 69f78b759aa53b04cb8d7962, 69f78b759aa53b04cb8d7963, 69f78b759aa53b04cb8d7964, 69f78b759aa53b04cb8d7965, 69f78b759aa53b04cb8d7966, 69f78b759aa53b04cb8d7967, 69f78b759aa53b04cb8d7968, 69f78bedba33dc8d97b29512, 69f78bedba33dc8d97b29513, 69f78bedba33dc8d97b29514, 69f78bedba33dc8d97b29515, 69f78bedba33dc8d97b29516, 69f78bedba33dc8d97b29517, 69f78bedba33dc8d97b29518, 69f78bedba33dc8d97b29519, 69f78bedba33dc8d97b2951a, 69f78bedba33dc8d97b2951b, 69f78bedba33dc8d97b2951c, 69f78bedba33dc8d97b2951d

Surplus (55, review then archive; includes the 7 Southern-Hemisphere months): 69f83a4c73d928e7025d9a40, 69f9bbf7ed9a20f525226552, 69f79e1303a060d3a4f915a2, 69f9b957c946f5452e99f020, 69f9bc74dd185ff25ffd7116, 69f79e1303a060d3a4f915a3, 69f9bcef96a092e3c7ace2bb, 69f79e1303a060d3a4f915a4, 69f9c4a0eff18e22049f73bb, 69f79e1303a060d3a4f915a5, 69f9c52654ec545fa81c50ed, 69f79e1303a060d3a4f915a6, 69f79e1303a060d3a4f915a7, 69f79e1303a060d3a4f915a8, 69f79e1303a060d3a4f915a9, 69f79e1303a060d3a4f915aa, 69f79e1303a060d3a4f915ab, 69f79e1303a060d3a4f915ac, 69f79e1303a060d3a4f915ad, 69f79e1303a060d3a4f915ae, 69f79e1303a060d3a4f915af, 69f79e1303a060d3a4f915b0, 69f79e1303a060d3a4f915b1, 69f79e1303a060d3a4f915b2, 69f79e1303a060d3a4f915b3, 69f79e1303a060d3a4f915b4, 69f79e1303a060d3a4f915b5, 69f79e1303a060d3a4f915b6, 69f79e1303a060d3a4f915b7, 69f79e1503a060d3a4f915b8, 69f79e1503a060d3a4f915b9, 69f79e1503a060d3a4f915ba, 69f79e1503a060d3a4f915bb, 69f79e1503a060d3a4f915bc, 69f79e1503a060d3a4f915bd, 69f79e1503a060d3a4f915be, 69f79e1503a060d3a4f915bf, 69f79e1503a060d3a4f915c0, 69f79e1503a060d3a4f915c1, 69f79e1503a060d3a4f915c2, 69f79eb503a060d3a4f915c9, 69f79eb503a060d3a4f915ca, 69f79eb503a060d3a4f915cb, 69f79eb503a060d3a4f915cc, 69f79eb503a060d3a4f915cd, 69f79eb503a060d3a4f915ce, 69f79eb503a060d3a4f915cf, 69f79eb503a060d3a4f915d0, 69f79eb503a060d3a4f915d1, 69f79eb503a060d3a4f915d2, 69f79eb503a060d3a4f915d3, 69f79eb903a060d3a4f915dc, 69f79eb903a060d3a4f915dd, 69f79eb903a060d3a4f915de, 69f79eb903a060d3a4f915df

**Duplicate January weeks** (20, batches 2026-05-03 19:15/19:20, attached to the kept January months): 69f79eb703a060d3a4f915d4, 69f79eb703a060d3a4f915d5, 69f79eb703a060d3a4f915d6, 69f79eb703a060d3a4f915d7, 69f79eb703a060d3a4f915d8, 69f79eb703a060d3a4f915d9, 69f79eb703a060d3a4f915da, 69f79eb703a060d3a4f915db, 69f7a00603a060d3a4f915ee, 69f7a00603a060d3a4f915ef, 69f7a00603a060d3a4f915f0, 69f7a00603a060d3a4f915f1, 69f7a00603a060d3a4f915f2, 69f7a00603a060d3a4f915f3, 69f7a00603a060d3a4f915f4, 69f7a00603a060d3a4f915f5, 69f7a00603a060d3a4f915f6, 69f7a00603a060d3a4f915f7, 69f7a00603a060d3a4f915f8, 69f7a00603a060d3a4f915f9

**Weeks with more than 5 CurriculumDay** (week id and day count; the Week page shows at most 10): 69f7905a9c96d96ac1f4bf7e (10), 69f7905a9c96d96ac1f4bf7f (10), 69f7905a9c96d96ac1f4bf80 (10), 69f7905a9c96d96ac1f4bf81 (10), 69f7905a9c96d96ac1f4bf82 (10), 69f7915877fe3e83e1a7dfb6 (10), 69f7915877fe3e83e1a7dfb7 (10), 69f7915877fe3e83e1a7dfb8 (10), 69f7915877fe3e83e1a7dfb9 (10), 69f7915877fe3e83e1a7dfba (10), 69f7915877fe3e83e1a7dfbb (10), 69f7915877fe3e83e1a7dfbc (10), 69f7915877fe3e83e1a7dfbd (10), 69f7915877fe3e83e1a7dfbe (10), 69f7915877fe3e83e1a7dfbf (10), 69f7915877fe3e83e1a7dfc0 (10), 69f7915877fe3e83e1a7dfc1 (10), 69f7915877fe3e83e1a7dfc2 (10), 69f7915877fe3e83e1a7dfc3 (10), 69f7915877fe3e83e1a7dfc4 (10), 69f7915877fe3e83e1a7dfc5 (10), 69f7915877fe3e83e1a7dfc6 (10), 69f7915877fe3e83e1a7dfc7 (10), 69f7915877fe3e83e1a7dfc8 (10), 69f7920ce4dd821ccd8a12a3 (10), 69f7920ce4dd821ccd8a12a4 (10), 69f7920ce4dd821ccd8a12a5 (10), 69f79eb703a060d3a4f915d6 (25), 69f79eb703a060d3a4f915d7 (25), 69f838c37ebad4869d41fd17 (10), 69f838c93bab35eaef9e322d (10), 69f838c951a281375c73413d (10), 69f838c96acf8880bafbae54 (10), 69f838ca3dc7f42cc483500c (10), 69f838cd14f8a8bca43bc1b9 (10), 69f838cd367ab3048ebc476a (10), 69f838cd7bd1dd11f9e79033 (10), 69f838cedd5daba0461cb312 (10), 69f838d06adaa2270de0e285 (10), 69f838d1091ccb82cbb5c36b (10), 69f838d13bab35eaef9e322e (10), 69f838d1dbfe82f191cacfea (10), 69f838d2a7ec0c8995be8d75 (10), 69f838d2cd0fd541846922fb (10), 69f838d2f754e5b0625680ff (10), 69f838d39dcc4200fc319104 (10), 69f83fa200f4879d11f89e5e (10), 69f83fa20d9b3ffc77288463 (15), 69f83fa2146fff871dcbde8f (10), 69f83fa216bcff9cc41ba3b3 (10), 69f83fa21cb48eedc4767f42 (10), 69f83fa2241063e79d19dbe7 (10), 69f83fa2241063e79d19dbe8 (10), 69f83fa2362f59e4d50cd064 (10), 69f83fa237b50034b02ae085 (10), 69f83fa23c0f9f1d92b64a69 (10), 69f83fa241b6b4c5e1e62be1 (15), 69f83fa241b6b4c5e1e62be2 (10), 69f83fa241e81ffedebcbab7 (10), 69f83fa24921eb17cab23af3 (10), 69f83fa25dbf4a6ca168b82f (10), 69f83fa25f5b463403e4ba2e (15), 69f83fa26221912ca937aec0 (15), 69f83fa2641a81dedc1d21d6 (15), 69f83fa2648551c13d51cbdb (15), 69f83fa2699a64d717a6232e (10), 69f83fa270ff649b2ecc2fac (10), 69f83fa27d0d7f3454d91ff6 (10), 69f83fa27f51fa921d84508f (10), 69f83fa29bb0ec73ace9480d (10), 69f83fa2a148a107c45ecab8 (10), 69f83fa2a288b756eb2f887e (10), 69f83fa2afffc88e3eab1a3f (10), 69f83fa2b1ece930ec0a430f (10), 69f83fa2c13394eb276b9cf0 (10), 69f83fa2c13394eb276b9cf1 (10), 69f83fa2d8266280ab199b19 (10), 69f83fa2dcca3dc9085c0bb0 (10), 69f83fa2e24e1f295ce98ba4 (10), 69f83fa2fdb19f55f2e20e7a (10), 69f83fa2ff8df4efa8a09957 (10), 69f83fa337a616d2bccdf977 (10)

**Duplicate EveningDay** (64; the later id of each (age, week, day_number) pair): 6a035895dfdc05075f80e208, 6a035895dfdc05075f80e209, 6a03521ce930c7c140251d54, 6a03521ce930c7c140251d55, 6a035e38ef55b3d37a9fa19d, 6a035e38ef55b3d37a9fa19e, 6a035e38ef55b3d37a9fa1a5, 6a035e38ef55b3d37a9fa1a6, 6a035e38ef55b3d37a9fa1ad, 6a035e38ef55b3d37a9fa1ae, 6a035e85c80eae86fa19d515, 6a035e85c80eae86fa19d516, 6a035e85c80eae86fa19d51d, 6a035e85c80eae86fa19d51e, 6a035e85c80eae86fa19d525, 6a035e85c80eae86fa19d526, 6a035895dfdc05075f80e20a, 6a035895dfdc05075f80e20b, 6a03521ce930c7c140251d56, 6a03521ce930c7c140251d57, 6a035e38ef55b3d37a9fa19f, 6a035e38ef55b3d37a9fa1a0, 6a035e38ef55b3d37a9fa1a7, 6a035e38ef55b3d37a9fa1a8, 6a035e38ef55b3d37a9fa1af, 6a035e38ef55b3d37a9fa1b0, 6a035e85c80eae86fa19d517, 6a035e85c80eae86fa19d518, 6a035e85c80eae86fa19d51f, 6a035e85c80eae86fa19d520, 6a035e85c80eae86fa19d527, 6a035e85c80eae86fa19d528, 6a035b5476240517d7e0034d, 6a035b8a8545eef5fe5f9ad8, 6a0356dd5dadc3a4715031d5, 6a0356dd5dadc3a4715031d6, 6a035e38ef55b3d37a9fa1a1, 6a035e38ef55b3d37a9fa1a2, 6a035e38ef55b3d37a9fa1a9, 6a035e38ef55b3d37a9fa1aa, 6a035e38ef55b3d37a9fa1b1, 6a035e38ef55b3d37a9fa1b2, 6a035e85c80eae86fa19d519, 6a035e85c80eae86fa19d51a, 6a035e85c80eae86fa19d521, 6a035e85c80eae86fa19d522, 6a035e85c80eae86fa19d529, 6a035e85c80eae86fa19d52a, 6a035bc16686afe23a4a16a9, 6a035bf486408000d73effb7, 6a0356dd5dadc3a4715031d7, 6a0356dd5dadc3a4715031d8, 6a035e38ef55b3d37a9fa1a3, 6a035e38ef55b3d37a9fa1a4, 6a035e38ef55b3d37a9fa1ab, 6a035e38ef55b3d37a9fa1ac, 6a035e38ef55b3d37a9fa1b3, 6a035e38ef55b3d37a9fa1b4, 6a035e85c80eae86fa19d51b, 6a035e85c80eae86fa19d51c, 6a035e85c80eae86fa19d523, 6a035e85c80eae86fa19d524, 6a035e85c80eae86fa19d52b, 6a035e85c80eae86fa19d52c

## 6. Month vs term: what teachers see today

| Surface | Label shown now |
|---|---|
| /curriculum/tier/:id | Grid of 12 month cards: "Jan" + theme + small "Term 1" |
| Month page | Chip "January · Term 1"; weeks listed as "Week 1…4" |
| Week page | Chip "Week 1" (week of the month); title "January W1 · Open"; no term |
| Day page | Day name and title; no term, no week-of-term |
| /curriculum/calendar | "January — theme", "Weekly Plans — January", "Week N"; never shows a term |
| /daily-plan (morning) | Date + month theme + "Week N · title" |
| Finniped panel | "This week in Heartwise · Session 2 · pages 12–13"; no term or week |
| Evening pages | "Term 1…**4**" (13 weeks each) and "Week 1…52"; calendar week counted from 1 Jan |
| Data | CurriculumMonth.term = 1 (Jan–Apr), 2 (May–Aug), 3 (Sep–Dec); CurriculumWeek.term_number is null on all 480 morning weeks; FinnipedPlacement uses month + week_in_month |

So morning is labelled month-first (12-month year) with the term as a footnote. Evening uses 4 terms. The Numenature/HeartWise material is really 3 × 12 teaching weeks.

## 7. Recommendation: one rule

**The label is "Term N · Week k", with the calendar month and dates as secondary text only.**

- **Term:** Sri Lankan school terms. Term 1 = January–April, Term 2 = May–August, Term 3 = September–December. This matches the existing CurriculumMonth.term.
- **Week k counts from the start of the term.** Formula: `term_week = (month − term_start_month) × 4 + week_in_month`, where term_start_month is 1, 5 or 9.
  - Weeks 1–12 are teaching weeks. They hold the 36 Numenature/HeartWise weeks exactly, because placements sit in months 1–3, 5–7 and 9–11.
  - Weeks 13–16 (April, August, December) are labelled "Term N · Consolidation week" and show a review card instead of an empty session panel.
- **Evening:** re-map the 4 × 13 structure onto Terms 1–3 and never show "Term 4".
- **Store the values; don't derive them in each page.** Add `term_number` and `term_week` to CurriculumWeek and FinnipedPlacement. Backfill them from month and week_in_month. Show a single helper label, e.g. "Term 2 · Week 5 (June, week 1)", on the Tier, Month, Week, Day, Calendar, DailyPlan and Finniped panel.
- **Wording:** never use spring, summer, autumn/fall, winter or "this season". Where nature timing matters, use monsoon terms (south-west/north-east monsoon, inter-monsoon) or festivals (Avurudu, Vesak, Poson, Deepavali).

**Order of fixes:**
1. Correct the generator prompt (C-01).
2. Retire the 7 Southern-Hemisphere months and their children (C-02).
3. Collapse to the 60 canonical months and dedupe January weeks and days (C-03, C-05, C-06).
4. Backfill term and week, and switch the labels (C-14, C-15).
5. Fix the error and empty states (C-12).
