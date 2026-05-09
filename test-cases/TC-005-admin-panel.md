# TC-005 — Admin Panel

**Feature:** Admin Authentication + CMS Dashboard
**Author:** Senior QA Engineer
**Date:** 2026-05-09
**Priority:** P1 — High

---

## Preconditions

- Backend running, MongoDB connected
- Admin user account exists in DB

---

## AUTHENTICATION TESTS

### TC-F-043: Admin login with valid credentials

**Priority:** High

**Test Steps:**
1. Navigate to `/<ADMIN_PATH>/login`
2. Enter valid admin email and password
3. Click "Login"

**Expected Results:**
- Redirected to admin dashboard `/<ADMIN_PATH>`
- `token` httpOnly cookie is set
- Dashboard renders without errors
- No 401 or 403 errors in network tab

---

### TC-F-044: Admin login with invalid password

**Priority:** High

**Test Steps:**
1. Navigate to admin login
2. Enter valid email + wrong password
3. Click "Login"

**Expected Results:**
- Error message shown: "Invalid credentials" or similar
- User stays on login page
- No cookie set
- HTTP 401 from `POST /api/auth/login`

---

### TC-F-045: Admin login rate limiting (5 attempts)

**Priority:** High

**Test Steps:**
1. Attempt login with wrong password 6 times from the same IP

**Expected Results:**
- First 5 attempts: 401 Unauthorized with "Invalid credentials"
- 6th attempt within 15 minutes: 429 Too Many Requests with "Too many login attempts. Try again in 15 minutes."
- Login form is blocked for 15 minutes

---

### TC-F-046: Protected admin routes block unauthenticated access

**Priority:** High

**Test Steps:**
1. Clear all cookies (log out)
2. Navigate directly to `/<ADMIN_PATH>/projects`

**Expected Results:**
- Redirected to `/<ADMIN_PATH>/login`
- Admin Projects page is not rendered
- No project data is exposed
- API interceptor fires the redirect (401 → login page)

---

### TC-F-047: Token expiry redirects to login

**Priority:** High

**Test Steps:**
1. Log in as admin
2. Manually delete the `token` cookie from browser DevTools
3. Try to perform any admin action (e.g. add a project)

**Expected Results:**
- API call returns 401
- `api.interceptors.response` catches the 401
- Browser redirects to `/<ADMIN_PATH>/login`
- `isRedirecting` flag prevents multiple simultaneous redirects

---

### TC-F-048: Admin logout clears session

**Priority:** High

**Test Steps:**
1. Log in as admin
2. Click Logout (wherever available in the UI)
3. Try to navigate to any protected admin page

**Expected Results:**
- `token` cookie is cleared
- Protected routes redirect to login
- No stale data visible

---

## ADMIN PROJECTS PANEL TESTS

### TC-F-049: Admin projects page displays all sources

**Priority:** High

**Test Steps:**
1. Log in and navigate to Admin → Projects

**Expected Results:**
- Page shows "// ALL PROJECTS (N)" heading
- CMS projects appear without source badge
- Static projects appear with `Static` blue badge
- GitHub repos appear with `GitHub` teal badge and GitHub icon
- Total count N = CMS + unimported Static + unimported GitHub

---

### TC-F-050: Refresh button reloads both CMS and GitHub data

**Priority:** Medium

**Test Steps:**
1. Click the "Refresh" button in the admin projects header

**Expected Results:**
- `loadCms()` is called — fresh fetch from `GET /api/projects/admin`
- `loadGithub()` is called — fresh fetch from `GET /api/github/repos`
- "fetching GitHub..." indicator shows briefly
- List updates with fresh data

---

### TC-F-051: Add project form clears after submit

**Priority:** Medium

**Test Steps:**
1. Fill in the add project form completely
2. Submit the form
3. Observe the form after successful submission

**Expected Results:**
- All form fields reset to empty/default values
- Editing state is cleared (`editing = null`)
- Form header reads "// NEW PROJECT"

---

### TC-F-052: Edit form pre-fills all fields correctly

**Priority:** High

**Test Steps:**
1. Click ✏️ Edit on a CMS project that has all fields filled
2. Inspect the form

**Expected Results:**
- Title, Description, Category, GitHub URL, Live URL, Order all pre-filled
- Tech Stack shows comma-separated string (e.g. "React, Node.js")
- Featured checkbox reflects the project's current state
- Hidden checkbox reflects the project's current state
- Form header reads "// EDIT PROJECT"
- Page scrolls to the top of the form

---

### TC-F-053: Working action disables buttons on that row

**Priority:** Medium

**Test Steps:**
1. Click ★ Feature on a GitHub repo (which requires auto-import)
2. Immediately observe the row buttons

**Expected Results:**
- Row dims (opacity-50)
- All action buttons on that row are `disabled`
- Other rows remain fully interactive
- After action completes, row re-enables

---

## ADMIN DASHBOARD / NAVIGATION

### TC-F-054: All admin nav links are accessible

**Priority:** Medium

**Test Steps:**
1. Log in to admin
2. Click each nav item: Dashboard, Projects, Blogs, CTF, Testimonials, Site Config, Settings

**Expected Results:**
- Each page loads without errors
- Correct content renders for each section
- No broken links or 404 pages

---

### TC-ERR-011: Direct API write without token

**Priority:** High

**Test Steps:**
1. Send `POST /api/projects` with valid JSON payload but no Authorization header or cookie

**Expected Results:**
- HTTP 401
- `{ "message": "Not authorized" }`
- Database unchanged

---

### TC-ERR-012: 2FA token rejected for regular API calls

**Priority:** High

**Test Steps:**
1. Obtain a 2FA intermediate JWT (the one issued before 2FA completion, with `step: '2fa'`)
2. Use it to call `GET /api/projects/admin`

**Expected Results:**
- HTTP 401
- `{ "message": "Not authorized" }`
- Middleware correctly rejects `step === '2fa'` tokens

---

### TC-ERR-013: Deleted user token rejected

**Priority:** High

**Test Steps:**
1. Create a test admin user, log in, get a token
2. Delete that user from MongoDB directly
3. Use the obtained token to call `GET /api/projects/admin`

**Expected Results:**
- HTTP 401
- `{ "message": "Not authorized" }`
- `logSecurityEvent('ORPHAN_TOKEN', ...)` is called in the backend

---

## Coverage Matrix

| Requirement | Test Cases |
|---|---|
| Login with valid credentials | TC-F-043 |
| Login with invalid credentials | TC-F-044 |
| Login rate limiting | TC-F-045 |
| Protected route blocks unauthenticated access | TC-F-046 |
| Token expiry redirect | TC-F-047 |
| Logout clears session | TC-F-048 |
| Admin projects page — all sources | TC-F-049 |
| Refresh reloads data | TC-F-050 |
| Form resets after submit | TC-F-051 |
| Edit form pre-fills | TC-F-052 |
| Working action disables row | TC-F-053 |
| Admin nav links | TC-F-054 |
| Unauthenticated API write | TC-ERR-011 |
| 2FA token rejected | TC-ERR-012 |
| Orphaned token rejected | TC-ERR-013 |
