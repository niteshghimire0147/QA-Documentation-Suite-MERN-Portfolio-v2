# Test Plan — Nitesh Portfolio v2

**Document Version:** 1.0
**Author:** Senior QA Engineer
**Date:** 2026-05-09
**Status:** Final

---

## 1. Introduction

This test plan covers the QA activities for the Nitesh Portfolio v2 web application — a MERN stack (MongoDB, Express, React, Node.js) portfolio site with a full CMS, GitHub integration, and a live cybersecurity news feed.

---

## 2. Scope

### 2.1 In Scope

| Feature Area | Description |
|---|---|
| Projects CMS | Add, edit, delete, feature, hide projects via admin panel |
| GitHub Integration | Auto-fetch public GitHub repos and display in projects section |
| Static Projects | Hardcoded projects manageable via CMS |
| Hide / Delete | Hide projects from portfolio without deletion; delete from CMS |
| News Feed | Live RSS feed from The Hacker News + Palo Alto Networks |
| News Page | Standalone `/news` page with search and refresh |
| Admin Authentication | JWT-based login with 2FA and httpOnly cookie |
| Admin Panel | Dashboard, projects, blogs, CTF, testimonials, site config |
| Public Pages | Home, Blog, CTF, News, Security Disclosure, PGP |
| Navbar | Navigation links, active section highlighting |

### 2.2 Out of Scope

- Performance / load testing
- Penetration testing of the admin auth system
- Mobile responsiveness (visual-only, no automated tests)
- Third-party CMS platforms (Contentful, Sanity)
- Email delivery of contact form

---

## 3. Test Objectives

1. Verify all CRUD operations for projects work correctly for all three sources (CMS, Static, GitHub)
2. Confirm hide/delete functionality affects the frontend correctly and immediately
3. Validate that GitHub repos cannot be accidentally deleted from the UI
4. Ensure the news feed correctly fetches, merges, caches and displays articles from both RSS sources
5. Verify admin authentication guards all write operations
6. Confirm the public portfolio correctly reflects CMS state

---

## 4. Test Approach

### 4.1 Testing Types

| Type | Description | Priority |
|---|---|---|
| Functional Testing | All user-facing features against requirements | P1 |
| Integration Testing | Frontend ↔ Backend ↔ MongoDB ↔ External APIs | P1 |
| Regression Testing | Re-test after every bug fix | P1 |
| Boundary Testing | Min/max inputs, empty states, pagination limits | P2 |
| Negative Testing | Invalid inputs, unauthorized access, malformed data | P1 |
| UI / State Testing | Loading states, error states, empty states | P2 |
| Security Testing | Auth bypass attempts, unauthenticated API calls | P1 |

### 4.2 Test Levels

- **Unit:** Backend route handlers (manual API testing via curl/Postman)
- **Integration:** Frontend + Backend communication (browser-based)
- **End-to-End:** Full user flows from browser to database

---

## 5. Test Environment

| Component | Value |
|---|---|
| Frontend | React 18 + Vite + Tailwind CSS |
| Backend | Node.js + Express 4 |
| Database | MongoDB (local: `mongodb://localhost:27017/nitesh-portfolio`) |
| Auth | JWT (HS256) via httpOnly cookie + Bearer header fallback |
| External APIs | GitHub REST API, THN RSS, Palo Alto Networks RSS |
| Browser | Chrome (latest), Firefox (latest) |
| Base URL | `http://localhost:5173` (frontend), `http://localhost:5000` (API) |

---

## 6. Entry and Exit Criteria

### Entry Criteria
- Backend server is running and connected to MongoDB
- Frontend dev server is running
- Admin account exists in the database
- GitHub API accessible (no rate-limit block)
- RSS feeds reachable

### Exit Criteria
- All P1 (High priority) test cases pass
- Zero open High/Critical bugs
- All previously identified bugs confirmed fixed
- Coverage matrix shows 100% requirement coverage

---

## 7. Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| GitHub API rate limit during tests | Medium | Medium | Use cached responses; test in batches |
| RSS feed unavailable | Low | Medium | Backend serves stale cache gracefully |
| MongoDB connection drop | Low | High | Test error handling paths separately |
| JWT secret rotation between tests | Low | High | Keep test session within one server run |
| Static project reappearing after delete | Confirmed Bug | High | Fixed — regression test included |

---

## 8. Test Deliverables

| Deliverable | Location |
|---|---|
| Test Plan (this document) | `qa/TEST_PLAN.md` |
| Bug Report | `qa/bug-reports/BUG_REPORT.md` |
| Test Strategy | `qa/test-design/TEST_STRATEGY.md` |
| Coverage Matrix | `qa/test-design/COVERAGE_MATRIX.md` |
| Test Cases — Projects CMS | `qa/test-cases/TC-001-projects-cms.md` |
| Test Cases — GitHub Integration | `qa/test-cases/TC-002-github-integration.md` |
| Test Cases — Hide/Delete | `qa/test-cases/TC-003-hide-delete.md` |
| Test Cases — News Feed | `qa/test-cases/TC-004-news-feed.md` |
| Test Cases — Admin Panel | `qa/test-cases/TC-005-admin-panel.md` |
| Test Cases — Public Pages | `qa/test-cases/TC-006-public-pages.md` |
| QA Session Report | `qa/reports/QA_SESSION_REPORT.md` |

---

## 9. Test Schedule

| Phase | Activity | Duration |
|---|---|---|
| Phase 1 | Smoke test — all pages load, API responds | 30 min |
| Phase 2 | Projects CMS CRUD + GitHub integration | 2 hours |
| Phase 3 | Hide/Delete functionality across all sources | 1.5 hours |
| Phase 4 | News feed — both RSS sources, caching, search | 1 hour |
| Phase 5 | Admin panel — auth, all sections | 1.5 hours |
| Phase 6 | Public pages — navigation, data display | 1 hour |
| Phase 7 | Regression on all fixed bugs | 1 hour |

**Total Estimated Duration:** ~8.5 hours

---

## 10. Roles and Responsibilities

| Role | Responsibility |
|---|---|
| QA Engineer | Execute test cases, report bugs, verify fixes |
| Developer | Fix reported bugs, provide fix confirmation |
| Product Owner | Review bug severity classifications |
