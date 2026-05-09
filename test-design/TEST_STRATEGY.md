# Test Strategy — Nitesh Portfolio v2

**Author:** Senior QA Engineer (20 years experience)
**Date:** 2026-05-09

---

## 1. Testing Philosophy

This project follows a **risk-based testing** approach. Features with the highest likelihood of defects or the highest impact on user experience are tested first and most thoroughly. Testing is requirement-driven — every test case maps back to a specific user story or functional requirement.

### Core Principles

1. **Test behaviour, not implementation** — Test what the user sees and does, not how the code works internally
2. **Fail fast** — Run smoke tests before deep functional tests
3. **Regression is non-negotiable** — Every bug fix gets a regression test case
4. **Boundary conditions always bite** — Edge cases (empty arrays, null fields, API failures) get dedicated test cases
5. **Security is a first-class citizen** — Authentication and authorization are tested as rigorously as functional paths

---

## 2. Testing Levels

### 2.1 API Testing (Backend)

All backend routes are tested directly using HTTP requests (curl or Postman) to verify:
- Correct HTTP status codes
- Response body shape and data types
- Auth middleware (`protect`) blocks unauthenticated requests
- Rate limiting returns `429` after threshold
- Input validation rejects bad data
- Soft delete doesn't physically remove records

**Key API Endpoints Under Test:**

| Endpoint | Method | Auth Required |
|---|---|---|
| `/api/projects` | GET | No |
| `/api/projects/admin` | GET | Yes |
| `/api/projects/suppressed` | GET | No |
| `/api/projects` | POST | Yes |
| `/api/projects/:id` | PUT | Yes |
| `/api/projects/:id` | DELETE | Yes |
| `/api/github/repos` | GET | No |
| `/api/news` | GET | No |
| `/api/auth/login` | POST | No |

### 2.2 Integration Testing (Frontend + Backend)

Tested via the browser, verifying:
- Admin panel actions correctly reflect in the public portfolio
- CMS state changes (hide, delete, feature) are immediately visible after page refresh
- GitHub repos fetched from GitHub API are filtered using DB-stored suppressed titles
- News fetched from two RSS feeds are merged, sorted by date, and rendered

### 2.3 End-to-End Testing (Full User Flows)

Complete user journeys tested manually:

| Flow | Steps |
|---|---|
| Admin adds a project | Login → Projects → Fill form → Submit → Verify on portfolio |
| Admin hides a project | Login → Projects → Click Hide → Navigate to portfolio → Verify gone |
| Admin deletes a project | Login → Projects → Click Delete → Confirm → Verify gone |
| Admin features a project | Login → Projects → Click ★ → Verify appears first on portfolio |
| Visitor views news | Navigate to `/news` → Verify articles load from both sources |
| Visitor searches news | Navigate to `/news` → Type search term → Verify filtered results |

---

## 3. Test Design Techniques

### 3.1 Equivalence Partitioning
Used for input fields (title, description, URL fields):
- Valid input: well-formed strings, valid URLs
- Invalid input: empty strings, non-URL format for URL fields, excessively long strings

### 3.2 Boundary Value Analysis
Applied to:
- `pageSize` parameter: test with 1, 50 (default), 100 (max), 101 (over max)
- `order` field: 0 (min), 99 (default), large numbers
- News article count: test with 0, 6 (slice limit), >6 articles from feed
- Cache TTL: test fresh response vs. cached response within 15 minutes

### 3.3 State Transition Testing
Key state machines under test:

**Project visibility states:**
```
[Static/GitHub, not in DB]
        ↓ (first edit/hide/feature action)
    [In DB, visible]
        ↓ (hide)
    [In DB, hidden]
        ↓ (show)
    [In DB, visible]
        ↓ (delete)
    [In DB, deleted]  ← terminal state
```

**Admin authentication states:**
```
[Unauthenticated]
        ↓ (valid credentials)
    [Authenticated, no 2FA]
        ↓ (2FA code)
    [Authenticated, 2FA passed]
        ↓ (logout / token expiry)
    [Unauthenticated]
```

### 3.4 Error Guessing
Based on 20 years of experience, the following common failure modes were specifically targeted:
- Array merge operations producing duplicates (→ found BUG-010)
- Async state where deleted items fall out of a key set (→ found BUG-001, BUG-002)
- API response format changes breaking consumers (→ found BUG-008)
- Missing schema fields silently ignored by MongoDB (→ found BUG-009)
- Hardcoded data bypassing database-driven visibility (→ found BUG-003, BUG-004)

---

## 4. Test Data Strategy

### 4.1 Static Projects (Hardcoded)
Used directly from `src/data/staticProjects.js`. These 8 projects serve as baseline test data for all static-source tests.

### 4.2 CMS Projects (Created During Testing)
Test cases that require CMS projects create them during setup and clean up (delete) during teardown.

### 4.3 GitHub Repos
Real data fetched from `https://api.github.com/users/niteshghimire0147/repos`. Tests account for the live nature of this data by checking structure rather than specific repo names.

### 4.4 News Articles
Real RSS data from:
- `https://feeds.feedburner.com/TheHackersNews`
- `https://www.paloaltonetworks.com/blog/feed/`

Tests verify structural integrity (title, link, pubDate, source) rather than specific article content.

---

## 5. Test Execution Order

Run test suites in this order to minimise interference:

1. **Smoke tests** — All pages load (200 OK), API health check passes
2. **Auth tests** — Login, token validation, logout
3. **Read-only tests** — GET endpoints, public pages
4. **Write tests** — POST/PUT/DELETE via admin
5. **State transition tests** — Hide → verify → Show → verify → Delete → verify
6. **Integration tests** — Frontend reflects backend changes
7. **Regression tests** — All 10 bug scenarios

---

## 6. Tooling

| Tool | Purpose |
|---|---|
| Browser DevTools | Network tab, console errors, cookie inspection |
| curl / Postman | Direct API endpoint testing |
| MongoDB Compass | Verify DB state directly after operations |
| Browser (incognito) | Verify public pages without admin session |
| Git diff | Verify code changes match stated fixes |
