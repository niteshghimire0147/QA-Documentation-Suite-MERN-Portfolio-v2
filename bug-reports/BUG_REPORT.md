# Bug Report — Nitesh Portfolio v2

**Author:** Senior QA Engineer
**Date:** 2026-05-09
**Total Bugs Found:** 10
**Open:** 0 | **Fixed:** 10 | **Won't Fix:** 0

---

## Severity Legend

| Severity | Description |
|---|---|
| 🔴 Critical | Application crash or data loss |
| 🟠 High | Core feature broken, no workaround |
| 🟡 Medium | Feature partially broken, workaround exists |
| 🟢 Low | Minor UI issue or inconvenience |

---

## BUG-001 — Deleted Static Projects Reappear in Admin Panel

| Field | Detail |
|---|---|
| **ID** | BUG-001 |
| **Severity** | 🟠 High |
| **Status** | ✅ Fixed |
| **Feature** | Projects CMS — Delete |
| **Found In** | `AdminProjects.jsx`, `GET /api/projects/admin` |

**Description:**
After deleting a static project (e.g. "Gumbili Studio") from the admin panel, the item disappeared momentarily then immediately reappeared in the list.

**Steps to Reproduce:**
1. Navigate to Admin → Projects
2. Find a static project (labelled with `Static` badge)
3. Click the 🗑 Delete button and confirm
4. Observe the project list refreshes

**Expected Result:** The project is permanently removed from the admin list.

**Actual Result:** The project reappeared immediately after `loadCms()` was called, because the deleted item was excluded from `cmsProjects`, causing it to fall back into `staticOnly`.

**Root Cause:**
`GET /api/projects/admin` filtered out `deleted: true` records. Once a static project was deleted, its title dropped out of `cmsKeys`. Since the STATIC array is hardcoded, the project was regenerated in `staticOnly` on the next render.

**Fix Applied:**
- Changed `GET /api/projects/admin` to return ALL records including `deleted: true`
- In the frontend, `cmsKeys` is now built from ALL CMS projects (including deleted)
- `displayedCms` filters out `deleted: true` items for rendering only
- Deleted titles permanently occupy `cmsKeys`, preventing regeneration from STATIC

---

## BUG-002 — Deleted GitHub Repos Reappear in Admin Panel

| Field | Detail |
|---|---|
| **ID** | BUG-002 |
| **Severity** | 🟠 High |
| **Status** | ✅ Fixed |
| **Feature** | Projects CMS — Delete (GitHub source) |
| **Found In** | `AdminProjects.jsx` |

**Description:**
Same root cause as BUG-001 but for GitHub-sourced repos. After deleting a GitHub repo entry, the item reappeared.

**Steps to Reproduce:**
1. Navigate to Admin → Projects
2. Find a GitHub repo (labelled with `GitHub` badge)
3. Click the 🗑 Delete button and confirm
4. Observe the project list refreshes

**Expected Result:** Repo disappears from admin list.

**Actual Result:** Repo reappears because `ghOnly` was recalculated from the live GitHub fetch without awareness of deleted DB records.

**Root Cause:** Same as BUG-001 — deleted titles fell out of `cmsKeys`, so GitHub repos with those titles reappeared in `ghOnly`.

**Fix Applied:** Same fix as BUG-001. The admin endpoint now returns deleted records and their titles stay in `cmsKeys` permanently.

---

## BUG-003 — Hidden Static Projects Still Visible on Portfolio Frontend

| Field | Detail |
|---|---|
| **ID** | BUG-003 |
| **Severity** | 🟠 High |
| **Status** | ✅ Fixed |
| **Feature** | Projects — Hide (Static source) |
| **Found In** | `ProjectsSection.jsx`, `GET /api/projects` |

**Description:**
Hiding a static project via the admin panel had no effect on the public portfolio. The project continued to display.

**Steps to Reproduce:**
1. Navigate to Admin → Projects
2. Find a static project (e.g. "SQL Injection Lab")
3. Click the 👁 Eye (Hide) button
4. Navigate to the public portfolio at `/#projects`
5. Observe the Projects section

**Expected Result:** "SQL Injection Lab" is not visible on the portfolio.

**Actual Result:** "SQL Injection Lab" remains visible.

**Root Cause:**
`ProjectsSection.jsx` fetched visible CMS projects from `GET /api/projects` but always merged the entire `STATIC` array back in unconditionally:
```js
setProjects(sortProjects([...r.data, ...STATIC]));
```
Even if a static project was hidden in the DB, it was still present in the hardcoded STATIC array and got merged in.

**Fix Applied:**
1. Added new backend endpoint `GET /api/projects/suppressed` — returns lowercase titles of all `hidden: true` OR `deleted: true` projects
2. `ProjectsSection.jsx` fetches suppressed titles first, then filters STATIC array to exclude any title in the suppressed set
3. Static projects already in the DB (visible) use the DB version; those in suppressed set are completely excluded

---

## BUG-004 — Hidden GitHub Repos Still Visible on Portfolio Frontend

| Field | Detail |
|---|---|
| **ID** | BUG-004 |
| **Severity** | 🟠 High |
| **Status** | ✅ Fixed |
| **Feature** | Projects — Hide (GitHub source) |
| **Found In** | `ProjectsSection.jsx`, `GET /api/github/repos` |

**Description:**
Hiding a GitHub repo via the admin panel had no effect on the public portfolio. The repo continued to display under the GitHub tab.

**Steps to Reproduce:**
1. Navigate to Admin → Projects
2. Find a GitHub repo
3. Click the 👁 Eye (Hide) button
4. Navigate to the public portfolio, click the `GitHub` filter tab
5. Observe the Projects section

**Expected Result:** The hidden repo is not visible.

**Actual Result:** The repo remains visible.

**Root Cause:**
GitHub repos were fetched directly from the GitHub REST API in `ProjectsSection.jsx`. This fetch completely bypassed the portfolio's database. The `hidden` flag existed only in MongoDB and was never consulted when rendering GitHub repos.

**Fix Applied:**
- `ProjectsSection.jsx` now fetches suppressed titles from `GET /api/projects/suppressed`
- `visibleGithub` filters the GitHub repo list by excluding any repo whose title matches a suppressed title
- The GitHub count badge also uses `visibleGithub.length` (not total fetched)

---

## BUG-005 — Delete Button Shown for GitHub Repos (Misleading UX)

| Field | Detail |
|---|---|
| **ID** | BUG-005 |
| **Severity** | 🟡 Medium |
| **Status** | ✅ Fixed |
| **Feature** | Admin Panel — Projects list |
| **Found In** | `AdminProjects.jsx` |

**Description:**
The delete button (🗑) was shown for GitHub-sourced repos in the admin panel. Since GitHub repos cannot be deleted from GitHub via this UI, the button was misleading. Users could be confused about whether the action deletes from GitHub.

**Steps to Reproduce:**
1. Navigate to Admin → Projects
2. Find any item with the `GitHub` badge
3. Observe the action buttons on the right

**Expected Result:** No delete button for GitHub repos. Only the hide/show toggle should be available.

**Actual Result:** A red delete button was present, which (on click) would import the repo to the DB and then soft-delete it — confusing and unexpected behaviour.

**Fix Applied:**
Wrapped the delete button in a conditional: `{source !== 'github' && <button ...>Delete</button>}`. GitHub items now only show the 👁 hide/show toggle for visibility management.

---

## BUG-006 — Static Projects Not Visible or Manageable in Admin Panel

| Field | Detail |
|---|---|
| **ID** | BUG-006 |
| **Severity** | 🟡 Medium |
| **Status** | ✅ Fixed |
| **Feature** | Admin Panel — Projects |
| **Found In** | `AdminProjects.jsx` |

**Description:**
The 8 hardcoded static projects (Gumbili Studio, Portfolio Website, etc.) were completely invisible in the CMS admin panel. Administrators had no way to edit, feature, hide or delete them without modifying source code.

**Steps to Reproduce:**
1. Navigate to Admin → Projects
2. Observe the project list

**Expected Result:** All projects visible on the portfolio (including static ones) are also listed in the admin panel.

**Actual Result:** Only database-stored CMS projects were listed. Static projects were absent.

**Root Cause:**
The STATIC array was defined only in `ProjectsSection.jsx` and never shared with `AdminProjects.jsx`.

**Fix Applied:**
1. Extracted STATIC array to `frontend/src/data/staticProjects.js`
2. Both `ProjectsSection.jsx` and `AdminProjects.jsx` import from this shared file
3. `AdminProjects.jsx` builds a unified list: CMS projects + static not-yet-in-DB + GitHub repos not-yet-in-DB

---

## BUG-007 — GitHub Repos Not Visible in Admin Panel

| Field | Detail |
|---|---|
| **ID** | BUG-007 |
| **Severity** | 🟡 Medium |
| **Status** | ✅ Fixed |
| **Feature** | Admin Panel — Projects |
| **Found In** | `AdminProjects.jsx` |

**Description:**
GitHub repos that appeared on the public portfolio were not visible in the admin panel, making it impossible for the admin to manage them.

**Steps to Reproduce:**
1. Navigate to the public portfolio and observe GitHub repos in the Projects section
2. Navigate to Admin → Projects
3. Observe the admin project list

**Expected Result:** GitHub repos appear in the admin panel with full management options.

**Actual Result:** Admin panel showed only CMS projects.

**Root Cause:**
`AdminProjects.jsx` only called `GET /api/projects` (CMS database). It never fetched from `GET /api/github/repos`.

**Fix Applied:**
Admin panel now fetches both `GET /api/projects/admin` and `GET /api/github/repos` on mount. All items are merged into a unified list with source badges (`CMS`, `Static`, `GitHub`).

---

## BUG-008 — News Section Format Mismatch After RSS Migration

| Field | Detail |
|---|---|
| **ID** | BUG-008 |
| **Severity** | 🟡 Medium |
| **Status** | ✅ Fixed |
| **Feature** | News Section (Home Page) |
| **Found In** | `NewsSection.jsx`, `mapArticle()` |

**Description:**
After the backend news route was rewritten to use RSS feeds instead of the NewsAPI, the home page news section displayed stale fallback articles instead of live RSS content. Cards showed placeholder data.

**Steps to Reproduce:**
1. Start the application
2. Navigate to the home page
3. Scroll to the "// 07. CYBER NEWS" section

**Expected Result:** Live articles from The Hacker News and Palo Alto Networks RSS feeds.

**Actual Result:** Static fallback articles were displayed (Krebs on Security, CISA placeholder content).

**Root Cause:**
The `normalise()` function in `NewsSection.jsx` expected the old NewsAPI response format:
```js
{ source: { name: '...' }, publishedAt: '...', url: '...' }
```
But the new RSS backend returns:
```js
{ source: 'The Hacker News', pubDate: '...', link: '...', categories: [...] }
```
The field mismatch caused the news array to map incorrectly. Additionally, the old `FALLBACK` static array was shown when the API returned data that didn't pass validation.

**Fix Applied:**
- Removed the FALLBACK static array entirely
- Replaced `normalise()` with `mapArticle()` that correctly maps `pubDate → pubDate`, `link → url`, `source (string) → source`
- Added loading skeleton UI instead of fallback content

---

## BUG-009 — News Section Missing `hidden` Field in Project Schema

| Field | Detail |
|---|---|
| **ID** | BUG-009 |
| **Severity** | 🟡 Medium |
| **Status** | ✅ Fixed |
| **Feature** | Backend — Project Model |
| **Found In** | `backend/models/Project.js`, `backend/routes/projects.js` |

**Description:**
The `hidden` field did not exist in the MongoDB Project schema. API calls to set `hidden: true` silently failed — MongoDB ignored the unknown field. The public `GET /api/projects` also did not filter hidden projects.

**Steps to Reproduce:**
1. Add a project via admin panel
2. Click the 👁 Hide button
3. Navigate to the public portfolio
4. Observe the project is still visible

**Expected Result:** Hidden projects do not appear on the public portfolio.

**Actual Result:** All projects appeared regardless of intended hidden state.

**Fix Applied:**
- Added `hidden: { type: Boolean, default: false, index: true }` to `Project.js` schema
- Added `hidden` to `pickProjectFields()` in `projects.js`
- Updated `GET /api/projects` filter: `{ deleted: { $ne: true }, hidden: { $ne: true } }`

---

## BUG-010 — Static Projects Duplicated When CMS Also Returns Them

| Field | Detail |
|---|---|
| **ID** | BUG-010 |
| **Severity** | 🟡 Medium |
| **Status** | ✅ Fixed |
| **Feature** | Projects Section (Frontend) |
| **Found In** | `ProjectsSection.jsx` |

**Description:**
When a static project was imported into the CMS (e.g. via the Edit action), the project appeared twice on the public portfolio — once from the DB and once from the hardcoded STATIC array.

**Steps to Reproduce:**
1. Navigate to Admin → Projects
2. Click Edit on a static project (e.g. "Portfolio Website") — this imports it to DB
3. Save the project
4. Navigate to the public portfolio `/#projects`
5. Observe the Projects grid

**Expected Result:** "Portfolio Website" appears exactly once.

**Actual Result:** "Portfolio Website" appeared twice (one from DB, one from hardcoded STATIC).

**Root Cause:**
`ProjectsSection.jsx` merged `[...r.data, ...STATIC]` unconditionally. Once a static project was imported to DB it appeared in both arrays.

**Fix Applied:**
```js
const dbTitles = new Set(r.data.map(p => p.title.toLowerCase().trim()));
const staticToShow = STATIC.filter(
  s => !dbTitles.has(s.title.toLowerCase().trim()) &&
       !suppressedSet.has(s.title.toLowerCase().trim())
);
setProjects(sortProjects([...r.data, ...staticToShow]));
```
Static projects are now only shown if they are NOT already present in the DB response.

---

## Summary Table

| ID | Title | Severity | Status | File(s) Affected |
|---|---|---|---|---|
| BUG-001 | Deleted static projects reappear in admin | 🟠 High | ✅ Fixed | `AdminProjects.jsx`, `routes/projects.js` |
| BUG-002 | Deleted GitHub repos reappear in admin | 🟠 High | ✅ Fixed | `AdminProjects.jsx` |
| BUG-003 | Hidden static projects visible on frontend | 🟠 High | ✅ Fixed | `ProjectsSection.jsx`, `routes/projects.js` |
| BUG-004 | Hidden GitHub repos visible on frontend | 🟠 High | ✅ Fixed | `ProjectsSection.jsx` |
| BUG-005 | Delete button shown for GitHub repos | 🟡 Medium | ✅ Fixed | `AdminProjects.jsx` |
| BUG-006 | Static projects not in admin panel | 🟡 Medium | ✅ Fixed | `AdminProjects.jsx`, `staticProjects.js` |
| BUG-007 | GitHub repos not in admin panel | 🟡 Medium | ✅ Fixed | `AdminProjects.jsx` |
| BUG-008 | News section format mismatch | 🟡 Medium | ✅ Fixed | `NewsSection.jsx` |
| BUG-009 | `hidden` field missing from Project schema | 🟡 Medium | ✅ Fixed | `models/Project.js`, `routes/projects.js` |
| BUG-010 | Static projects duplicated after CMS import | 🟡 Medium | ✅ Fixed | `ProjectsSection.jsx` |
