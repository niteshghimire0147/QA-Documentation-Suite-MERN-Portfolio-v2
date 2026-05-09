# QA Documentation — Nitesh Portfolio v2

**Author:** Senior QA Engineer (20 years experience)
**Project:** Nitesh Portfolio — MERN Stack + Vite + Tailwind CSS
**Repository:** `nitesh-portfolio-v2`
**Date:** 2026-05-09
**QA Coverage:** Projects CMS, GitHub Integration, Hide/Delete Features, News Feed, Admin Panel, Public Pages

---

## Folder Structure

```
qa/
├── README.md                        ← You are here
├── TEST_PLAN.md                     ← Overall test plan, scope, strategy
│
├── bug-reports/
│   └── BUG_REPORT.md               ← All bugs found, severity, status
│
├── test-design/
│   ├── TEST_STRATEGY.md             ← Testing approach, methodologies, tooling
│   └── COVERAGE_MATRIX.md          ← Requirement → test case traceability
│
├── test-cases/
│   ├── TC-001-projects-cms.md      ← CMS project CRUD operations
│   ├── TC-002-github-integration.md ← GitHub repo sync & display
│   ├── TC-003-hide-delete.md       ← Hide/show and delete functionality
│   ├── TC-004-news-feed.md         ← News section & RSS feeds
│   ├── TC-005-admin-panel.md       ← Admin authentication & panel
│   └── TC-006-public-pages.md      ← Public-facing portfolio pages
│
└── reports/
    └── QA_SESSION_REPORT.md        ← Full QA session findings & recommendations
```

---

## Quick Summary

| Area | Test Cases | Bugs Found | Bugs Fixed |
|---|---|---|---|
| Projects CMS | 22 | 3 | 3 |
| GitHub Integration | 14 | 2 | 2 |
| Hide / Delete | 18 | 4 | 4 |
| News Feed | 16 | 1 | 1 |
| Admin Panel | 20 | 0 | — |
| Public Pages | 14 | 0 | — |
| **Total** | **104** | **10** | **10** |

> All bugs identified during this QA session have been resolved. Zero open defects at time of writing.

---

## How to Use This Folder

1. Start with `TEST_PLAN.md` for scope and approach
2. Review `bug-reports/BUG_REPORT.md` for all found defects
3. Run test cases from `test-cases/` against the application
4. Verify coverage with `test-design/COVERAGE_MATRIX.md`
5. Read `reports/QA_SESSION_REPORT.md` for executive summary and recommendations
