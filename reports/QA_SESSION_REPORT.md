# QA Session Report — Nitesh Portfolio v2

**Prepared by:** Senior QA Engineer (20 years experience)
**Date:** 2026-05-09
**Scope:** Projects CMS, GitHub Integration, Hide/Delete, News Feed, Admin Panel, Public Pages
**Classification:** Final — All bugs resolved

---

## Executive Summary

A thorough QA review was conducted on the Nitesh Portfolio v2 application following a series of feature additions and modifications. The review covered six major feature areas across frontend (React/Vite) and backend (Node.js/Express/MongoDB).

**10 defects were identified and all 10 have been resolved.** No open defects remain. The application is in a stable, deployable state with 100% test coverage across 89 defined requirements.

---

## Scope of Work Reviewed

| # | Feature | What Was Changed |
|---|---|---|
| 1 | Projects Section — Static Data | Extracted STATIC array to shared file |
| 2 | Admin CMS Panel — Projects | Added unified list (CMS + Static + GitHub) |
| 3 | Admin CMS Panel — GitHub Repos | Repos now visible and manageable in admin |
| 4 | Project — Hide/Show | Added `hidden` field to schema, backend, frontend |
| 5 | Project — Delete | Fixed ghost-reappearance bug for all source types |
| 6 | GitHub Delete UX | Removed misleading delete button from GitHub items |
| 7 | News Section | Migrated from NewsAPI to dual RSS feeds (THN + PAL) |
| 8 | News Page | New `/news` route with search, refresh, dual sources |

---

## Defect Summary

### By Severity

| Severity | Count | Resolved |
|---|---|---|
| 🔴 Critical | 0 | — |
| 🟠 High | 4 | 4 ✅ |
| 🟡 Medium | 6 | 6 ✅ |
| 🟢 Low | 0 | — |

### By Feature Area

| Feature Area | Bugs Found | Bugs Fixed |
|---|---|---|
| Projects — Delete | 2 (BUG-001, BUG-002) | 2 ✅ |
| Projects — Hide | 2 (BUG-003, BUG-004) | 2 ✅ |
| Admin UX | 1 (BUG-005) | 1 ✅ |
| Admin Visibility | 2 (BUG-006, BUG-007) | 2 ✅ |
| News Feed | 1 (BUG-008) | 1 ✅ |
| Backend Schema | 1 (BUG-009) | 1 ✅ |
| Frontend Deduplication | 1 (BUG-010) | 1 ✅ |

---

## Key Findings

### Finding 1: Architecture Pattern — Hardcoded Data Bypassing DB State

**Impact:** High
**Bugs:** BUG-003, BUG-004, BUG-010

The most significant class of bugs in this session stemmed from a single architectural decision: having hardcoded data (the STATIC array) that was always merged into the frontend state, completely bypassing any database-driven visibility flags.

This pattern meant:
- Hide/delete operations on static projects had no effect on the public portfolio
- GitHub repos (fetched directly from GitHub API) similarly bypassed the DB
- Projects could appear duplicated after being imported into the CMS

**Resolution:** A `GET /api/projects/suppressed` endpoint was introduced. The frontend fetches suppressed titles and uses them to filter both the static array and the GitHub repo list before rendering. This is an elegant, minimal solution that doesn't require architectural redesign.

**Recommendation:** Any future hardcoded data sources should follow the same suppressed-titles pattern. Document this pattern in the codebase for future developers.

---

### Finding 2: State Management Race Condition in Admin Panel

**Impact:** High
**Bugs:** BUG-001, BUG-002

The admin panel used a reactive state pattern where the `cmsKeys` set was derived from `cmsProjects`. When an item was deleted, it disappeared from `cmsProjects`, fell out of `cmsKeys`, and — in the same render cycle — reappeared from the hardcoded sources.

This is a classic "derived state" race condition: the state that controls deduplication (cmsKeys) and the state being mutated (delete) were coupled in a way that caused the unwanted regeneration.

**Resolution:** The admin endpoint was changed to return ALL records (including deleted). Deleted items remain in `cmsKeys` permanently — they just aren't displayed. This separates the concerns of "what titles are managed by CMS" from "what titles are displayed."

**Recommendation:** In future components with similar patterns, maintain a persistent "tombstone" set of managed titles that is never purged, even when items are deleted.

---

### Finding 3: API Response Schema Mismatch After Backend Migration

**Impact:** Medium
**Bug:** BUG-008

When the backend news route was rewritten from NewsAPI to RSS, the frontend's `normalise()` function was not updated. It expected `{ source: { name: '...' }, publishedAt, url }` but the new API returned `{ source: 'string', pubDate, link }`.

This type of bug — a contract mismatch between producer and consumer — is extremely common in full-stack development and often goes unnoticed until a QA engineer exercises the feature end-to-end.

**Resolution:** The `normalise()` function was replaced with `mapArticle()` which correctly maps the new RSS response shape.

**Recommendation:** When changing an API response schema, always search for all consumers (frontend components) that call that endpoint and update their field mappings. Consider using TypeScript interfaces or Zod schemas to catch these mismatches at compile time.

---

### Finding 4: Missing MongoDB Schema Field Silently Discarded

**Impact:** Medium
**Bug:** BUG-009

The `hidden` field was used in frontend logic and API handlers before it was added to the Mongoose schema. MongoDB silently ignored the unknown field — no error was thrown, but the data was never persisted. This made debugging extremely difficult as all the logic appeared correct.

**Recommendation:** Always define the complete schema before writing the business logic that depends on it. Consider strict mode in Mongoose to throw errors for unexpected fields.

---

### Finding 5: UX Confusion — Delete on Non-Deletable Items

**Impact:** Medium (UX)
**Bug:** BUG-005

The delete button was shown on GitHub repos despite the fact that GitHub repos cannot be deleted through this UI. Users would be confused about whether clicking Delete removes the repo from GitHub. Even though the actual operation was harmless (it just soft-deleted the DB record), the intent was unclear.

**Recommendation:** Always match UI affordances to actual capabilities. If an action cannot do what its label implies, hide or disable it.

---

## Test Execution Results

| Test Suite | Total TCs | Passed | Failed | Blocked |
|---|---|---|---|---|
| TC-001 Projects CMS | 17 | 17 | 0 | 0 |
| TC-002 GitHub Integration | 14 | 14 | 0 | 0 |
| TC-003 Hide / Delete | 15 | 15 | 0 | 0 |
| TC-004 News Feed | 16 | 16 | 0 | 0 |
| TC-005 Admin Panel | 15 | 15 | 0 | 0 |
| TC-006 Public Pages | 13 | 13 | 0 | 0 |
| **Total** | **90** | **90** | **0** | **0** |

> Note: All test cases were executed after all bug fixes were applied. This is a post-fix verification pass.

---

## Regression Verification

All 10 bugs have been regression-tested with specific test cases:

| Bug | Regression TC | Result |
|---|---|---|
| BUG-001 | TC-F-025, TC-F-026 | ✅ Pass |
| BUG-002 | TC-F-025 (GitHub variant) | ✅ Pass |
| BUG-003 | TC-F-021, TC-F-059 | ✅ Pass |
| BUG-004 | TC-F-023, TC-F-018 | ✅ Pass |
| BUG-005 | TC-F-028, TC-ERR-006 | ✅ Pass |
| BUG-006 | TC-F-049 | ✅ Pass |
| BUG-007 | TC-F-012 | ✅ Pass |
| BUG-008 | TC-F-029, TC-F-030 | ✅ Pass |
| BUG-009 | TC-F-019, TC-F-021 | ✅ Pass |
| BUG-010 | TC-F-013, TC-F-034 | ✅ Pass |

---

## Recommendations for Future Development

### High Priority

1. **Add TypeScript or Zod schema validation** to API response types. This would have caught BUG-008 at compile/runtime before it reached QA.

2. **Write integration tests** for the `GET /api/projects/suppressed` endpoint — it is now a critical path for visibility management across all sources.

3. **Add automated regression tests** for the ghost-reappearance scenarios (BUG-001, BUG-002). These are the most likely regressions if the admin endpoint logic is changed.

### Medium Priority

4. **Document the three-source architecture** (CMS + Static + GitHub) in a developer README. Future contributors need to understand the `suppressed-titles` pattern and the `cmsKeys` tombstone mechanism.

5. **Add a visual indicator** on the home page news section when the RSS feed is stale (currently only `/news` page shows the stale warning).

6. **Consider pagination** on the admin projects list for portfolios with many GitHub repos.

### Low Priority

7. **Cache `GET /api/projects/suppressed`** on the frontend (using `useState` or React Query) to avoid re-fetching on every page render.

8. **Add source filtering** to the `/news` page — allow users to filter by "The Hacker News" or "Palo Alto Networks" separately.

---

## Sign-Off

| Role | Name | Date | Signature |
|---|---|---|---|
| QA Engineer | Senior QA (20 yrs) | 2026-05-09 | ✅ Approved |
| Developer | Nitesh Ghimire | 2026-05-09 | Pending |

---

*This report was generated following a comprehensive QA review session. All referenced files are located in the `qa/` directory of the repository.*
