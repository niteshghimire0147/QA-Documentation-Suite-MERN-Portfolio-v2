# TC-001 — Projects CMS Management

**Feature:** Projects CMS (Add / Edit / Delete / Feature / Order)
**Author:** Senior QA Engineer
**Date:** 2026-05-09
**Priority:** P1 — High

---

## Preconditions (apply to all test cases in this file)

- Backend server running on `http://localhost:5000`
- Frontend running on `http://localhost:5173`
- Admin is logged in (valid JWT cookie present)
- MongoDB is connected and accessible
- Admin → Projects page is open

---

## TC-F-001: Add a new CMS project successfully

**Priority:** High
**Preconditions:** Admin is logged in and on the Projects page

**Test Steps:**
1. Fill in the form: Title = "Test Project Alpha", Description = "A test project for QA", Tech Stack = "React, Node.js", Category = "Development", GitHub URL = "https://github.com/test/alpha", Order = 1
2. Leave the Featured checkbox unchecked
3. Click "Add Project"

**Expected Results:**
- Toast notification shows "Project added!"
- New project "Test Project Alpha" appears in the unified project list with no source badge (CMS item)
- The project is saved to MongoDB (`deleted: false`, `hidden: false`, `featured: false`)

**Postconditions:** "Test Project Alpha" exists in DB and admin list

---

## TC-F-002: Add project with Featured checked

**Priority:** High

**Test Steps:**
1. Fill form with valid data, check ✅ "★ Featured project"
2. Click "Add Project"

**Expected Results:**
- Project appears in admin list with "★ Featured" badge
- On the public portfolio, the project appears before non-featured projects

---

## TC-F-003: Edit an existing CMS project

**Priority:** High
**Preconditions:** At least one CMS project exists

**Test Steps:**
1. Click ✏️ Edit on an existing CMS project
2. Verify form is pre-filled with existing values
3. Change the title to "Updated Project Alpha"
4. Change tech stack to "Vue.js, Python"
5. Click "Update Project"

**Expected Results:**
- Toast shows "Updated!"
- Project list shows new title "Updated Project Alpha"
- Tech stack shows "Vue.js, Python"
- Form resets to blank "New Project" state
- MongoDB record reflects updated values

---

## TC-F-004: Cancel editing a project

**Priority:** Medium

**Test Steps:**
1. Click ✏️ Edit on any project
2. Modify the title field
3. Click "Cancel"

**Expected Results:**
- Form resets to empty `EMPTY` state
- Header changes from "// EDIT PROJECT" back to "// NEW PROJECT"
- The project in the list is unchanged

---

## TC-F-005: Delete a CMS project

**Priority:** High
**Preconditions:** "Test Project Alpha" CMS project exists

**Test Steps:**
1. Locate "Test Project Alpha" in the admin list
2. Click 🗑 Delete
3. Confirm the browser `confirm()` dialog

**Expected Results:**
- Project immediately disappears from the admin list
- Does NOT reappear after the list refreshes
- MongoDB record has `deleted: true`, `deletedAt` is set
- Project is NOT visible on the public portfolio

**Regression:** Verifies BUG-001 fix — deleted CMS projects must not reappear

---

## TC-F-006: Cancel delete confirmation dialog

**Priority:** Medium

**Test Steps:**
1. Click 🗑 Delete on any CMS project
2. Click "Cancel" in the browser confirm dialog

**Expected Results:**
- Project remains in the admin list unchanged
- No API call is made
- No toast notification appears

---

## TC-F-007: Feature toggle — mark project as featured

**Priority:** High

**Test Steps:**
1. Find a non-featured CMS project (☆ star is grey)
2. Click the ☆ star button

**Expected Results:**
- Toast shows "★ Marked as featured"
- Star turns yellow (★)
- "★ Featured" badge appears next to the project title
- On the public portfolio, the project appears before non-featured ones

---

## TC-F-008: Feature toggle — unfeature a project

**Priority:** High

**Test Steps:**
1. Find a featured CMS project (★ star is yellow)
2. Click the ★ star button

**Expected Results:**
- Toast shows "Removed from featured"
- Star turns grey (☆)
- "★ Featured" badge is removed
- On the public portfolio, the project moves to after featured projects

---

## TC-F-009: Set display order

**Priority:** Medium

**Test Steps:**
1. Create two projects: Project A with order = 2, Project B with order = 1
2. Both projects are non-featured

**Expected Results:**
- On the public portfolio, Project B (order 1) appears before Project A (order 2)
- Featured projects still appear before both

---

## TC-E-001: Add project with minimum required fields only

**Priority:** Medium

**Test Steps:**
1. Fill only required fields: Title = "Minimal Project", Description = "Only required fields"
2. Leave Tech Stack, GitHub URL, Live URL, Order all empty/default
3. Click "Add Project"

**Expected Results:**
- Project is saved successfully
- `techStack: []`, `githubUrl: ''`, `liveUrl: ''`, `order: 0` in DB
- No GitHub or ExternalLink icons shown in admin list (empty URLs)

---

## TC-E-002: Add project with very long title (200+ characters)

**Priority:** Low

**Test Steps:**
1. Enter a title that is 250 characters long
2. Click "Add Project"

**Expected Results:**
- Project saves without error (backend does not enforce title length limit in schema)
- Title displays truncated in the admin list row (CSS `truncate` class)
- Full title visible in the edit form

---

## TC-E-003: Add project with invalid GitHub URL

**Priority:** Medium

**Test Steps:**
1. Enter "not-a-url" in the GitHub URL field
2. Click "Add Project"

**Expected Results:**
- Browser native `type="url"` validation prevents form submission
- Error indicator shown on the GitHub URL field
- No API call made

---

## TC-E-004: Add project with non-HTTPS GitHub URL (http://)

**Priority:** Medium

**Test Steps:**
1. Enter "http://github.com/test" in the GitHub URL field
2. Click "Add Project"

**Expected Results:**
- Project saves (backend `sanitizeUrl` accepts `http://` as valid)
- GitHub URL stored as "http://github.com/test"
- GitHub icon link renders in the admin list

---

## TC-ERR-001: Attempt to add project without authentication

**Priority:** High

**Test Steps:**
1. Clear cookies / log out
2. Using curl or Postman, POST to `/api/projects` with valid project payload and no Authorization header

**Expected Results:**
- API returns `401 Unauthorized`
- Response body: `{ "message": "Not authorized" }`
- No project is created in DB

---

## TC-ERR-002: Attempt to update project with invalid MongoDB ID

**Priority:** Medium

**Test Steps:**
1. Send PUT request to `/api/projects/invalid-id-format` with valid auth token

**Expected Results:**
- API returns `400 Bad Request`
- Response body: `{ "message": "Invalid ID" }`

---

## TC-ERR-003: Attempt to delete non-existent project

**Priority:** Medium

**Test Steps:**
1. Send DELETE to `/api/projects/507f1f77bcf86cd799439011` (valid ObjectId format but not in DB) with valid auth

**Expected Results:**
- API returns `404 Not Found`
- Response body: `{ "message": "Project not found" }`

---

## TC-ERR-004: Submit add project form with missing required fields

**Priority:** High

**Test Steps:**
1. Leave the Title field empty
2. Fill in all other fields
3. Click "Add Project"

**Expected Results:**
- Browser native `required` validation fires
- Form does not submit
- Title field is highlighted as invalid
- No API call made

---

## TC-ERR-005: Submit add project form with missing description

**Priority:** High

**Test Steps:**
1. Fill in Title but leave Description empty
2. Click "Add Project"

**Expected Results:**
- Browser `required` validation fires on description textarea
- Form does not submit

---

## Coverage Matrix

| Requirement | Test Cases |
|---|---|
| Add project via admin form | TC-F-001, TC-F-002, TC-E-001, TC-E-002, TC-E-003, TC-E-004 |
| Edit existing project | TC-F-003, TC-F-004 |
| Delete project | TC-F-005, TC-F-006 |
| Feature/unfeature project | TC-F-007, TC-F-008 |
| Set display order | TC-F-009 |
| Auth protection on write operations | TC-ERR-001 |
| Input validation | TC-E-003, TC-ERR-004, TC-ERR-005 |
| Error handling | TC-ERR-002, TC-ERR-003 |
