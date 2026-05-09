# TC-006 — Public Pages

**Feature:** All public-facing pages of the portfolio
**Author:** Senior QA Engineer
**Date:** 2026-05-09
**Priority:** P2 — Medium

---

## Preconditions

- Application running
- Open browser in incognito (no admin session)
- MongoDB has sample blog posts and CTF writeups (if any)

---

## HOME PAGE SECTIONS

### TC-F-055: Home page renders all sections

**Priority:** High

**Test Steps:**
1. Navigate to `http://localhost:5173`
2. Scroll through the entire page

**Expected Results:**
- All sections render in order: Hero, About, Skills, Experience, Projects, Certifications, Arsenal, Hall of Fame, Testimonials, News, Contact
- No broken layouts or missing sections
- Cyber grid background visible and not blocking content
- Scroll progress bar appears at top

---

### TC-F-056: Navbar highlights correct active section on scroll

**Priority:** Medium

**Test Steps:**
1. On the home page, slowly scroll through each section
2. Observe the navbar links

**Expected Results:**
- Active section's nav link is highlighted in primary colour (`#00d4ff`)
- Active underline glow appears under the link
- Link becomes inactive when its section leaves viewport
- Only one section is highlighted at a time

---

### TC-F-057: Navbar News link navigates to /news page

**Priority:** High

**Test Steps:**
1. Click "News" in the navbar

**Expected Results:**
- Browser navigates to `/news`
- Full news page renders
- News link is highlighted as active in the navbar
- Back button returns to home page

---

### TC-F-058: Projects section filter tabs work correctly

**Priority:** High

**Test Steps:**
1. On home page, scroll to Projects
2. Click each filter tab: All, GitHub, Cybersecurity, Development, Academic

**Expected Results:**
- "All" shows all projects from all sources
- "GitHub" shows only GitHub-badged projects (with count badge)
- "Cybersecurity" shows only cybersecurity-category projects
- "Development" shows only development-category projects
- "Academic" shows only academic-category projects
- Switching tabs is instant (no loading state)

---

### TC-F-059: Projects section respects hidden flag

**Priority:** High
**Regression:** Verifies BUG-003 and BUG-004 fix

**Test Steps:**
1. Ensure at least one static project and one GitHub repo are hidden via admin
2. Open public portfolio in incognito
3. Check All tab and each filter tab in Projects section

**Expected Results:**
- Hidden static projects do not appear in any tab
- Hidden GitHub repos do not appear in the GitHub tab or All tab
- No console errors about missing data

---

## BLOG PAGE

### TC-F-060: Blog list page loads

**Priority:** Medium

**Test Steps:**
1. Navigate to `/blog`

**Expected Results:**
- Page shows "// WRITE-UPS" heading
- Blog posts listed (if any exist in DB)
- Search input is functional
- Tag filters appear if tags exist
- Empty state shown if no posts: "// No posts yet. Check back soon!"

---

### TC-F-061: Blog search filters posts

**Priority:** Medium

**Test Steps:**
1. Navigate to `/blog`
2. Type a keyword in the search bar

**Expected Results:**
- Only posts matching the keyword (in title, excerpt, or category) are shown
- Result count updates
- Clearing search restores all posts

---

## CTF PAGE

### TC-F-062: CTF list page loads

**Priority:** Medium

**Test Steps:**
1. Navigate to `/ctf`

**Expected Results:**
- CTF writeups page renders
- Writeups listed (if any exist)
- Empty state if no writeups
- No console errors

---

## NEWS PAGE

### TC-F-063: News page accessible from direct URL

**Priority:** High

**Test Steps:**
1. Navigate directly to `http://localhost:5173/news`

**Expected Results:**
- Page renders correctly with full Navbar and Footer
- News articles load from both RSS feeds
- Search bar is functional
- Both source credits shown at bottom

---

## OTHER PAGES

### TC-F-064: 404 page for unknown routes

**Priority:** Medium

**Test Steps:**
1. Navigate to `http://localhost:5173/this-does-not-exist`

**Expected Results:**
- NotFound page renders (not a blank page or browser 404)
- Navigation back to home is available
- Navbar and Footer are present

---

### TC-F-065: Security Disclosure page

**Priority:** Low

**Test Steps:**
1. Navigate to `/security-disclosure`

**Expected Results:**
- Page renders with security disclosure content
- Navbar and Footer present
- No broken links

---

### TC-F-066: PGP Key page

**Priority:** Low

**Test Steps:**
1. Navigate to `/pgp`

**Expected Results:**
- Page renders with PGP key content
- Navbar and Footer present

---

## MOBILE / RESPONSIVE

### TC-E-010: Mobile navbar toggle works

**Priority:** Medium

**Test Steps:**
1. Resize browser to mobile width (<768px) or use DevTools device mode
2. Observe the navbar

**Expected Results:**
- Desktop nav links are hidden
- Hamburger menu icon (FiMenu) is visible
- Clicking it reveals the mobile menu
- Clicking a link in mobile menu closes the menu
- Clicking X icon (FiX) closes the menu

---

### TC-E-011: Projects grid responsive layout

**Priority:** Medium

**Test Steps:**
1. View Projects section at mobile width

**Expected Results:**
- Grid switches to 1-column layout on mobile
- 2-column layout on tablet (md)
- 3-column layout on desktop (lg)
- Cards are not overflowing or clipped

---

## Coverage Matrix

| Requirement | Test Cases |
|---|---|
| Home page all sections render | TC-F-055 |
| Navbar active section highlight | TC-F-056 |
| News nav link | TC-F-057 |
| Projects filter tabs | TC-F-058 |
| Hidden projects not visible | TC-F-059 |
| Blog list page | TC-F-060, TC-F-061 |
| CTF list page | TC-F-062 |
| News page direct URL | TC-F-063 |
| 404 page | TC-F-064 |
| Security Disclosure | TC-F-065 |
| PGP Key page | TC-F-066 |
| Mobile navbar | TC-E-010 |
| Responsive grid | TC-E-011 |
