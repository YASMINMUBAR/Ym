# Finniped: report for Yasmin

Date: 27 September 2026. App: finniped.com (Base44 app 69f71ad98aafbe26ce61e092).
Nothing has been published. Every change is in the Base44 editor with a restore point.

## In short

- We found 139 problems. About 95 are now fixed, 9 are partly fixed and about 35 are left, most of them small.
- The most serious problem was privacy: any logged-in person could read every child's medical notes, allergies and emergency contacts. That is closed now.
- The main download for phones went from 3.6 MB to 0.57 MB.
- Some things need you: files, logins and a live check. They are listed below.

## What was wrong, and what we fixed

### Safety and privacy
- **Child records were open to everyone.** Now staff see only their own school, parents see only their own child and admins see everything. The same rules now cover teacher notes, messages, observations, growth records, portfolios, weekly reports, notifications, family notes, classrooms, schools, saved questions, parent preferences, special-time logs and more.
- **Parents could open staff pages by typing the address**, for example the children list, classrooms, reports and the upload tools. Now parents always go to their own area, accounts without a role see only the welcome page, and admin pages are for admins only.
- **Some server tasks could be triggered by anyone.** These included the emails sent when parents message the school, and paid AI printables. Now each one checks who is asking and uses the real record, not what the caller sends.
- **HeartWise body-safety material had lost its protection** in the new hw1 books. It is teachers and parents only again, on 8 books and 6 sessions.
- **The parent SOS screen now shows 1990 (ambulance) and 1929 (NCPA).**

### Curriculum
- **55 duplicate months were archived, not deleted.** They included 7 Tiny Roots months written for Australian seasons, such as "Whispers of Summer". We also archived their 220 weeks and 20 duplicate January weeks.
- **Every curriculum page now says "Term N · Week k".** The month is shown as small text. April, August and December are "Consolidation week".
- **The tool that writes new months said Sri Lanka is in the Southern Hemisphere.** It now uses Sri Lankan terms, the two monsoons and local festivals, never seasons.
- **Pages no longer spin for ever when something fails.** They show a short message and a "Try again" button.
- **The daily plan picked the wrong week.** It used week 1 of the month for every day; now it uses the right week.

### Library and books
- **Books for all ages vanished when a teacher picked an age band.** They now stay visible under every band.
- **The "By curriculum week" tab now shows HeartWise and Numenature** from the session placements.
- **Favourites follow a replaced book, and the star works.**
- **The reader works better.** The contents list works in the plain viewer, a missing file shows a message, and downloads open in a new tab.
- **Library data was tidied.** 5 older Numenature books are archived, and 14 Bodywise and Artelier books were moved to the age band in their titles.

### Content
- **Removed:** "Coming soon" buttons, placeholder text, a button that pretended a reminder was sent, made-up children on the teacher dashboard, and the Reports page, whose numbers and names were all invented.
- **Fixed wording:**
  - British spelling in visible text.
  - Dates written as "Friday 25 September 2026".
  - No season words left anywhere in the app's text.
- **Dashboard checks are honest.** The Launch Readiness checks, the review count and the festival banner, which now uses the festival records, all show real information.

### Speed and tidiness
- **Pages load only when opened.** The main download fell from 3.6 MB to 0.57 MB.
- **Old addresses and dead links are gone.** We removed dead sidebar links, the unused `/parent/admin` address and 25 unused page imports.
- **Search engines see the right details.** The page title, description, canonical link and sharing tags now say Finniped.

## Still needs you (my recommendation in bold)

1. **13 Wordwise books will not open.** Their files are missing, for example the Tiny Roots, Little Sprouts, Little Wonders and Pre-Primary teacher guides and workbooks, and the Lighthouse Manual. **Re-upload the original PDFs.** The list is in `round1/library_ids.json` under `broken_file_url`.
2. **The School Admin account has no school set**, so it now sees no children. **Tell me which school it belongs to.**
3. **Test logins, one per role.** Create one each for Super Admin, School Admin, Teacher, a parent with a child and a parent without. **I need these to prove the privacy rules on the real pages.**
4. **Let me reach the live site.** Add finniped.com, media.base44.com, images.unsplash.com and esm.sh to this session's network settings. **Without this I can't do the phone and desktop screenshot check.**
5. **Evening programme.** It still uses 4 terms, including "Term 4", and all 4 evening age groups are marked "merged". **Move it to Terms 1–3 and fix the evening age groups.**
6. **The Forest School fire session for 5–6-year-olds.** **Ask your curriculum lead to check it for safety.**
7. **Branding.** The browser tab icon is still the Base44 logo, and the phone app icon is an AI image. **Send me a Finniped logo and I'll put it in.**
8. **Duplicate evening days (64) and weeks with extra days (82).** **Let me archive the extras in the next pass.**

## Known gaps I will close next
- **Library books.** The rule that hides unpublished books from non-staff is on the placements, but not yet on the books themselves (the connection dropped).
- **Parents and other families' records.** Parents can still read other families' messages, growth records and portfolio entries through the app's data connection, but not on screen. The fix is a family field on those records.
- **Re-run the Base44 security scan.**

## How to publish (only when you say yes)
1. Open the Base44 editor for Finniped and check the preview.
2. The latest restore point is "Round 2 complete", plus the Round 3 privacy changes.
3. Press Publish.
4. Then I repeat the full check on finniped.com, as each role, on phone and desktop.
