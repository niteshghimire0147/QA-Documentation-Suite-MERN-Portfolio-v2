# TC-003 — Hide / Delete Functionality

**Feature:** Project visibility management (hide/show) and deletion across all three sources
**Author:** Senior QA Engineer
**Date:** 2026-05-09
**Priority:** P1 — High (4 bugs were found and fixed in this area)

---

## Preconditions

- Backend running, MongoDB connected
- Admin logged in
- At least one project of each source type exists:
  - CMS project (manually added)
  - Static project (from `staticProjects.js`, not yet imported)
  - GitHub repo (not yet imported)
- Public portfolio open in a separate incognito window for real-time verification

---

## HIDE / SHOW TESTS

### TC-F-019: Hide a CMS project

**Priority:** High

**Test Steps:**
1. In Admin → Projects, find a CMS project (no source badge)
2. Note its title (e.g. "Test Project Alpha")
3. Click the 👁 Eye button (currently showing as visible)
4. In the incognito window, reload the public portfolio and scroll to Projects

**Expected Results:**
- Toast shows "Project hidden from portfolio"
- Project row in admin dims and shows "Hidden" badge
- Project does NOT appear on the public portfolio after reload
- `GET /api/projects` (public) does not return this project
- `GET /api/projects/admin` (authenticated) still returns it with `hidden: true`

---

### TC-F-020: Show (unhide) a CMS project

**Priority:** High
**Preconditions:** "Test Project Alpha" is hidden (from TC-F-019)

**Test Steps:**
1. In Admin → Projects, find "Test Project Alpha" (dimmed, "Hidden" badge)
2. Click the 👁‍🗨 Eye-Off button
3. Reload the public portfolio

**Expected Results:**
- Toast shows "Project visible on portfolio"
- "Hidden" badge disappears from admin list
- Project is visible on the public portfolio again
- `GET /api/projects` returns this project with `hidden: false`

---

### TC-F-021: Hide a Static project — frontend effect

**Priority:** High
**Regression:** Verifies BUG-003 fix

**Test Steps:**
1. Note which static projects are currently visible on the public portfolio (e.g. "SQL Injection Lab")
2. In Admin → Projects, find "SQL Injection Lab" (Static badge)
3. Click 👁 Hide
4. Reload the public portfolio in incognito

**Expected Results:**
- "SQL Injection Lab" disappears from the public portfolio
- `GET /api/projects/suppressed` returns `"sql injection lab"` in the array
- The static STATIC array entry is filtered out by the frontend
- Project does NOT appear in any filter tab (All, Cybersecurity)

---

### TC-F-022: Show (unhide) a Static project

**Priority:** High
**Preconditions:** "SQL Injection Lab" is hidden (from TC-F-021)

**Test Steps:**
1. In Admin → Projects, find "SQL Injection Lab" (now a CMS item, dimmed, "Hidden" badge)
2. Click 👁‍🗨 Show
3. Reload public portfolio

**Expected Results:**
- "SQL Injection Lab" reappears on the portfolio
- `GET /api/projects/suppressed` no longer includes `"sql injection lab"`
- Project is visible in its correct filter category (Cybersecurity)

---

### TC-F-023: Hide a GitHub repo — frontend effect

**Priority:** High
**Regression:** Verifies BUG-004 fix

**Test Steps:**
1. Note GitHub repos visible on the public portfolio GitHub tab
2. Pick one repo (e.g. "Nitesh Portfolio V2")
3. In Admin → Projects, find that GitHub repo (GitHub badge)
4. Click 👁 Hide
5. Reload the public portfolio, click GitHub filter tab

**Expected Results:**
- The repo is NOT visible on the GitHub tab
- The count badge on the GitHub filter button decreases by 1
- `GET /api/projects/suppressed` includes the repo title (lowercased)
- The repo still appears in Admin → Projects (dimmed, "Hidden" badge)

---

### TC-F-024: Suppressed endpoint returns correct titles

**Priority:** High

**Test Steps:**
1. Hide 2 projects (1 CMS, 1 static)
2. Delete 1 CMS project
3. Call `GET /api/projects/suppressed` directly

**Expected Results:**
- Response is a JSON array of lowercase strings
- Array contains titles of both hidden projects and the deleted project
- No other data (no IDs, descriptions) is exposed — titles only
- HTTP 200 with correct Content-Type: application/json

---

### TC-E-007: Hide and show toggle is idempotent

**Priority:** Medium

**Test Steps:**
1. Click 👁 Hide on a project
2. Immediately click 👁‍🗨 Show on the same project
3. Repeat 3 times rapidly

**Expected Results:**
- Final state matches the last click (visible if last click was Show)
- No duplicate DB entries
- No console errors
- `actionId` lock prevents concurrent conflicting actions on the same item

---

## DELETE TESTS

### TC-F-025: Delete a CMS project permanently removes it from admin

**Priority:** High
**Regression:** Verifies BUG-001 fix

**Test Steps:**
1. Create a new CMS project "Delete Me Test"
2. Click 🗑 Delete and confirm
3. Observe the admin list
4. Wait 3 seconds and observe again

**Expected Results:**
- "Delete Me Test" disappears immediately from the list
- Does NOT reappear after `loadCms()` refresh
- `GET /api/projects/admin` returns the record with `deleted: true` (soft delete)
- `GET /api/projects` (public) does NOT return the record

---

### TC-F-026: Delete a Static project removes it from admin permanently

**Priority:** High
**Regression:** Verifies BUG-001 fix (static variant)

**Test Steps:**
1. Find a static project (Static badge) — e.g. "Krishi Guru App"
2. Click 🗑 Delete and confirm
3. Observe the admin list (wait for refresh)
4. Scroll through entire list

**Expected Results:**
- "Krishi Guru App" disappears from the admin list
- Does NOT reappear even though it is in the hardcoded `STATIC_PROJECTS` array
- The deleted title remains in `cmsKeys` (DB-stored, `deleted: true`) — blocking regeneration from static
- NOT visible on the public portfolio

---

### TC-F-027: Deleted project not visible on public portfolio

**Priority:** High

**Test Steps:**
1. Note the title of any project currently visible on the public portfolio
2. Delete it via Admin → Projects
3. Reload the public portfolio in incognito

**Expected Results:**
- The deleted project is NOT visible in any filter tab
- `GET /api/projects` does not include it (filtered by `deleted: { $ne: true }`)
- `GET /api/projects/suppressed` includes its title (deleted projects are suppressed)

---

### TC-F-028: No delete button on GitHub-sourced items

**Priority:** High
**Regression:** Verifies BUG-005 fix

**Test Steps:**
1. Navigate to Admin → Projects
2. Inspect every item with a `GitHub` badge
3. Check for the presence of a 🗑 Delete button

**Expected Results:**
- Zero GitHub-badged items have a delete button
- Only 👁 Eye, ✏️ Edit, ★ Star, and 🔗 GitHub link are shown for GitHub items

---

### TC-ERR-008: Cancel delete on browser confirm dialog

**Priority:** Medium

**Test Steps:**
1. Click 🗑 Delete on any project
2. Click "Cancel" on the browser `confirm()` dialog

**Expected Results:**
- Project remains in the list unchanged
- No API request is made (`DELETE /api/projects/:id` not called)
- No toast notification appears

---

### TC-ERR-009: Unauthenticated delete attempt via API

**Priority:** High

**Test Steps:**
1. Log out of admin panel
2. Using curl, send `DELETE /api/projects/<valid-id>` with no auth token

**Expected Results:**
- API returns `401 Unauthorized`
- `{ "message": "Not authorized" }`
- Project record is unchanged in DB (`deleted` remains `false`)

---

### TC-ST-001: Full lifecycle — Static project (import → hide → show → delete)

**Priority:** High (state transition test)

**Test Steps:**
1. **Verify initial state:** "Hotel Booking System" is visible on portfolio and in admin (Static badge)
2. **Hide:** Click 👁 Hide → auto-imported to DB → portfolio hides it
3. **Verify hidden:** Public portfolio does not show it; suppressed endpoint includes it
4. **Show:** Click 👁‍🗨 Show → portfolio shows it again
5. **Verify visible:** Public portfolio shows it; suppressed endpoint does not include it
6. **Delete:** Click 🗑 Delete and confirm
7. **Verify deleted:** Admin list no longer shows it; portfolio does not show it; item does not reappear

**Expected Results:** Each state transition above behaves exactly as stated

---

### TC-ST-002: Full lifecycle — GitHub repo (hide → show → feature → unfeature)

**Priority:** High (state transition test)

**Test Steps:**
1. **Verify:** A GitHub repo is visible on portfolio GitHub tab and in admin (GitHub badge)
2. **Hide:** Click 👁 Hide → auto-imported to DB → repo disappears from portfolio
3. **Verify hidden:** Portfolio GitHub tab does not show it; count badge decreases
4. **Show:** Click 👁‍🗨 Show → repo reappears on portfolio
5. **Feature:** Click ★ → repo appears first in portfolio
6. **Unfeature:** Click ★ → repo moves to normal position

**Expected Results:** Each transition correct with no ghosting or duplication

---

## Coverage Matrix

| Requirement | Test Cases |
|---|---|
| Hide CMS project | TC-F-019 |
| Show CMS project | TC-F-020 |
| Hide static project (frontend effect) | TC-F-021 |
| Show static project | TC-F-022 |
| Hide GitHub repo (frontend effect) | TC-F-023 |
| Suppressed endpoint | TC-F-024 |
| Idempotent toggle | TC-E-007 |
| Delete CMS project (no ghost) | TC-F-025 |
| Delete static project (no ghost) | TC-F-026 |
| Delete effect on public portfolio | TC-F-027 |
| No delete button on GitHub items | TC-F-028 |
| Cancel delete dialog | TC-ERR-008 |
| Unauthenticated delete | TC-ERR-009 |
| Full static lifecycle | TC-ST-001 |
| Full GitHub lifecycle | TC-ST-002 |
