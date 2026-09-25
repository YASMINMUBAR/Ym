# Access model: Finnish Pedagogy OS (app 69f71ad98aafbe26ce61e092)

Audit was read only. App files are at /app in the Base44 sandbox. Findings are in roles.csv (R-01 to R-22).

## How roles are determined
- The role is `User.role`, a custom enum in `base44/entities/User.jsonc` with the values `super_admin, admin, school_admin, teacher, parent, user`. The default is `user`. Other scoping fields are `school_id`, `classroom_id` and `child_ids` (parents).
- The current user comes from `useAuth()` in `src/lib/AuthContext.jsx`. A "View As" layer sits on top: `useImpersonation().effectiveUser` in `src/lib/ImpersonationContext.jsx`.
- Live counts (12 users): super_admin 2, teacher 3, parent 2, school_admin 1 (no school_id), user 4, admin 0.

## Route layers (src/App.jsx)
1. Public routes: `/parent-login`, `/register-training`, `/player`.
2. `<ParentGuard>` (`src/components/parent/ParentGuard.jsx`) lets in role=parent (with a soft-launch email allow-list) plus admin/super_admin:
```jsx
if (!user) return <Navigate to="/" replace />;
const isAdmin = user.role === "admin" || user.role === "super_admin";
if (user.role !== "parent" && !isAdmin) return <Navigate to="/" replace />;
if (user.role === "parent" && !allowed) return <SoftLaunchWaiting />;
```
3. `<AppShell>` (`src/pages/AppShell.jsx`) does **no** role check. It wraps `ImpersonationProvider` and `AppLayout`. `AppLayout` (`src/components/layout/AppLayout.jsx:41,80`) wraps every staff page in `<RoleRouteGuard>`. That is the **only central route guard**:
```jsx
// src/components/layout/RoleRouteGuard.jsx
const ADMIN_ONLY_PREFIXES = ["/admin/","/polaris","/orchestrator","/bulk-generation",
  "/users","/schools","/network","/kasvunpolku/admin","/curriculum-canvas"];
function isAdminLike(role) { return role === "super_admin" || role === "admin"; }
...
const isOpenInternalTool = path === "/admin/story-pdf-uploader" || path === "/admin/song-kit-uploader";
if (isAdminRoute && !isOpenInternalTool && role && !isAdminLike(role)) {
  const home = role === "parent" ? "/my-child" : "/";
  return <Navigate to={home} replace />;
}
```
   It uses `effectiveUser`, so View As is respected.

**Where to add guards:** `src/components/layout/RoleRouteGuard.jsx`. Replace the single prefix list with a per-prefix allowed-roles map. Add a rule that role=parent may not use any AppShell route (redirect to `/parent`), and fail closed when the role is missing. Also delete `/parent/admin` at `src/App.jsx:244`.

## Gaps in brief
- A parent or role=user can open by URL: `/children` (every child plus all message and observation inboxes), `/classrooms` (can create, edit and delete), `/enrollment` (every child), `/age-groups`, `/reports`, `/policies/*`, `/ai-assistant`, `/pro-studio` (Song.update), `/settings`, `/daily-plan`, `/curriculum/*`, `/music`, `/admin/story-pdf-uploader`, `/admin/song-kit-uploader`.
- Pages that guard themselves: KasvunpolkuControl (staff only, so `/parent/admin` bounces a parent to `/`), CalendarStudio, StoryStudio, BackgroundJobs (`/jobs`), MusicProviderSettings, FwsImport. UserManagement and Enrollment only gate their buttons.
- Nav mismatch: school_admin nav links to `/users`, `/schools`, `/kasvunpolku/admin`, `/network`, `/curriculum-canvas`, `/bulk-generation` and `/admin/music-room`, but RoleRouteGuard redirects school_admin away from all of them. Teacher nav links to `/kasvunpolku/admin`, which is also redirected.

## Row-level security (the real protection; client guards can be bypassed with the SDK)
- **No RLS** (every user can read and write): Child (medical, allergy, emergency-contact data), TeacherNote, Classroom, School, ParentDashboardSnapshot, ParentSubscriptionPreference, SavedQuestion, ActivityCompletion, AIContentLog, Printable, AgeGroup, Activity, DailySchedule and others. See `grep -L '"rls"' base44/entities/*`.
- **Open read (`"read": {}`)**, with create/update granted to all roles including parent and user: ParentMessage, ParentObservation, ChildGrowthRecord, ChildMilestoneObservation, ChildPortfolioEntry, FamilyEcho, WeeklyGrowthReport, ParentNotification (update {} too), SpecialTimeSession, PolicyDocument, PolicySignoff.
- **Properly owner-scoped:** AskFounderConversation, SOSSession, FindYourWayBackSession and ParentReflection (creator or super_admin); InvitedCoParent (creator); ImpersonationEvent, ParentPulseResponse, MusicProviderConfig and PolarisSettings (admin); TrainingRegistration (admin/school_admin); BackgroundJob (admin, or school_admin of the same school).

## View As
`ViewAsPill` only renders when `isSuperAdmin`, which is `realUser.role` super_admin **or admin** (ImpersonationContext.jsx:28). Stored impersonation is only rehydrated for those roles. Every session is logged to ImpersonationEvent. It is UI-only: server calls still use the admin's JWT, `ReadOnlyGuard` blocks writes by matching button text, and many pages read `useAuth()` rather than `effectiveUser`.

## Backend functions (140)
Handlers with no caller check that write through asServiceRole and trust the request body: onParentMessageCreated (sends emails), onActivityCompleted, onParentActivityCompletion, onParentMilestoneObserved, onSchoolCreatedClonePolicies, sendWeeklySummaryNotifications and sendWeeklyVoiceNote. generatePrintable makes auth optional. bookTrainingSession is public by design. Several generate/export functions check login but not role. Functions that return 403 were not each verified line by line.
