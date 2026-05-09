# TC-002 — GitHub Integration

**Feature:** GitHub Repos — Auto-fetch, display, admin management
**Author:** Senior QA Engineer
**Date:** 2026-05-09
**Priority:** P1 — High

---

## Preconditions

- Backend server running
- GitHub API accessible (not rate-limited)
- `niteshghimire0147` GitHub account has at least one public non-forked repo with a description
- Admin is logged in for admin-panel test cases

---

## TC-F-010: GitHub repos auto-load on portfolio page

**Priority:** High

**Test Steps:**
1. Open the public portfolio at `http://localhost:5173`
2. Scroll to the Projects section
3. Wait for the "fetching github repos…" indicator to disappear

**Expected Results:**
- GitHub repos appear in the project grid
- Each GitHub card shows the `GitHub` badge
- Cards display title (title-cased repo name), description, star count, fork count
- GitHub icon links to the correct repo URL

---

## TC-F-011: GitHub tab filter shows only GitHub repos

**Priority:** High

**Test Steps:**
1. On the Projects section, click the `GitHub` filter button

**Expected Results:**
- Only cards with the `GitHub` badge are shown
- Static and CMS projects disappear from the grid
- The count badge next to "GitHub" matches the number of visible cards

---

## TC-F-012: GitHub repos appear in admin panel

**Priority:** High

**Test Steps:**
1. Navigate to Admin → Projects
2. Observe the unified project list

**Expected Results:**
- GitHub repos appear in the list with the `GitHub` badge and GitHub icon
- GitHub repos that are not yet in the CMS DB appear with the `GitHub` badge
- No delete button (🗑) is visible on GitHub items
- Eye (👁) and Star (☆) and Edit (✏️) buttons are present on GitHub items

---

## TC-F-013: GitHub repo — first edit auto-imports to CMS

**Priority:** High
**Preconditions:** A GitHub repo exists that is NOT yet in the CMS DB

**Test Steps:**
1. In Admin → Projects, find a GitHub-badged item
2. Click ✏️ Edit

**Expected Results:**
- A brief loading state appears (row dims)
- Form is pre-filled with the GitHub repo's data
- Header reads "// EDIT PROJECT"
- The item now appears in the CMS list (no longer `GitHub` badge — now CMS entry)
- `techStack` is pre-filled with the repo's primary language

---

## TC-F-014: GitHub repo — feature toggle auto-imports

**Priority:** High
**Preconditions:** A GitHub repo exists that is NOT yet in CMS DB

**Test Steps:**
1. In Admin → Projects, find a GitHub-badged item with ☆ (unfeatured)
2. Click the ☆ button

**Expected Results:**
- Row dims briefly (action in progress)
- Toast shows "★ Marked as featured"
- Item moves to top of list / shows "★ Featured" badge
- Item is now stored in DB with `featured: true`
- On public portfolio, the item appears before non-featured projects

---

## TC-F-015: Fork repos are excluded from GitHub display

**Priority:** Medium

**Test Steps:**
1. Verify that `niteshghimire0147` account has at least one forked repo
2. Check the portfolio Projects section GitHub tab
3. Check the admin panel GitHub items

**Expected Results:**
- Forked repos (where `fork: true` in GitHub API) are not shown anywhere
- Only original repos (`fork: false`) are displayed

---

## TC-F-016: Repos without descriptions are excluded

**Priority:** Medium

**Test Steps:**
1. Check if `niteshghimire0147` has any repo with an empty description
2. Observe the portfolio GitHub tab

**Expected Results:**
- Repos with empty/null description are filtered out
- Only repos with meaningful descriptions appear

---

## TC-F-017: GitHub repo with homepage shows live link

**Priority:** Low

**Test Steps:**
1. Find a GitHub repo in the portfolio that has a `homepage` URL set on GitHub
2. Observe the project card

**Expected Results:**
- The `FiExternalLink` icon is visible on the card
- Clicking it opens the homepage URL in a new tab

---

## TC-F-018: GitHub tab count badge reflects visible repos

**Priority:** Medium

**Test Steps:**
1. Note the count shown in the GitHub filter badge (e.g. "GitHub (5)")
2. Hide two GitHub repos via admin panel
3. Refresh the portfolio page
4. Observe the GitHub count badge

**Expected Results:**
- Count badge decreases by 2 (e.g. "GitHub (3)")
- The hidden repos are no longer in the visible grid

**Regression:** Verifies BUG-004 fix and correct `visibleGithub.length` in badge

---

## TC-E-005: GitHub API returns empty array

**Priority:** Medium

**Test Steps:**
1. Temporarily mock `GET /api/github/repos` to return `[]`
2. Load the portfolio page

**Expected Results:**
- No GitHub cards appear in the All tab
- GitHub tab count badge does not appear (or shows 0)
- "fetching github repos…" message disappears
- Static and CMS projects still display normally

---

## TC-E-006: GitHub API is unreachable (network error)

**Priority:** Medium

**Test Steps:**
1. Block outbound requests to `api.github.com` (via hosts file or network rule)
2. Load the portfolio page

**Expected Results:**
- No GitHub cards appear — fails silently
- Static and CMS projects render normally
- No uncaught console errors break the page
- Admin panel GitHub section shows "No public repos found" or empty list

---

## TC-ERR-006: No delete button on GitHub items in admin

**Priority:** High
**Regression:** Verifies BUG-005 fix

**Test Steps:**
1. Navigate to Admin → Projects
2. Find any item with the `GitHub` badge that is NOT yet imported to CMS

**Expected Results:**
- 🗑 Delete button is NOT present
- Only ✏️ Edit, 👁 Hide/Show, ★ Feature, and 🔗 GitHub link are visible

---

## TC-ERR-007: GitHub API rate limit — graceful handling

**Priority:** Medium

**Test Steps:**
1. Exhaust GitHub API rate limit (60 requests/hour unauthenticated) or mock a 403/429 response
2. Load the portfolio page

**Expected Results:**
- Page does not crash
- GitHub section shows no repos (fails silently)
- Static and CMS projects still display
- Backend serves from cache if a previous successful fetch was cached

---

## Coverage Matrix

| Requirement | Test Cases |
|---|---|
| Auto-fetch public GitHub repos | TC-F-010, TC-E-005, TC-E-006 |
| Filter: show only GitHub repos | TC-F-011 |
| GitHub repos visible in admin | TC-F-012 |
| Auto-import on admin action | TC-F-013, TC-F-014 |
| Fork filtering | TC-F-015 |
| Description filtering | TC-F-016 |
| Homepage link | TC-F-017 |
| Count badge accuracy | TC-F-018 |
| No delete button on GitHub items | TC-ERR-006 |
| Rate limit / error handling | TC-ERR-007, TC-E-006 |
