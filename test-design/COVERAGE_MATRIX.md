# Test Coverage Matrix — Nitesh Portfolio v2

**Author:** Senior QA Engineer
**Date:** 2026-05-09

---

## Legend

| Symbol | Meaning |
|---|---|
| ✅ | Fully covered |
| ⚠️ | Partially covered |
| ❌ | Not covered |

---

## 1. Projects CMS

| Requirement | Test Case(s) | Coverage |
|---|---|---|
| Add project — required fields | TC-F-001, TC-ERR-004, TC-ERR-005 | ✅ |
| Add project — featured | TC-F-002 | ✅ |
| Add project — minimum fields | TC-E-001 | ✅ |
| Add project — URL validation | TC-E-003, TC-E-004 | ✅ |
| Edit project | TC-F-003, TC-F-052 | ✅ |
| Cancel edit | TC-F-004 | ✅ |
| Delete CMS project (no ghost) | TC-F-005, TC-F-025 | ✅ |
| Cancel delete dialog | TC-F-006, TC-ERR-008 | ✅ |
| Feature toggle | TC-F-007, TC-F-008 | ✅ |
| Display order | TC-F-009 | ✅ |
| Auth protection — POST | TC-ERR-001 | ✅ |
| Auth protection — PUT | TC-ERR-002 | ✅ |
| Auth protection — DELETE | TC-ERR-009 | ✅ |
| Invalid ObjectId | TC-ERR-002 | ✅ |
| Non-existent project delete | TC-ERR-003 | ✅ |

---

## 2. GitHub Integration

| Requirement | Test Case(s) | Coverage |
|---|---|---|
| Auto-fetch GitHub repos | TC-F-010, TC-E-005, TC-E-006 | ✅ |
| Filter: GitHub tab | TC-F-011 | ✅ |
| Repos visible in admin panel | TC-F-012 | ✅ |
| Auto-import on edit | TC-F-013 | ✅ |
| Auto-import on feature | TC-F-014 | ✅ |
| Filter fork repos | TC-F-015 | ✅ |
| Filter repos without descriptions | TC-F-016 | ✅ |
| Homepage URL → live link | TC-F-017 | ✅ |
| Count badge accuracy | TC-F-018 | ✅ |
| No delete button on GitHub items | TC-F-028, TC-ERR-006 | ✅ |
| Rate limit — graceful handling | TC-ERR-007 | ✅ |
| Empty repos response | TC-E-005 | ✅ |
| GitHub API unreachable | TC-E-006 | ✅ |

---

## 3. Hide / Delete Functionality

| Requirement | Test Case(s) | Coverage |
|---|---|---|
| Hide CMS project | TC-F-019 | ✅ |
| Show CMS project | TC-F-020 | ✅ |
| Hide static project → frontend gone | TC-F-021 | ✅ |
| Show static project → frontend visible | TC-F-022 | ✅ |
| Hide GitHub repo → frontend gone | TC-F-023 | ✅ |
| Suppressed endpoint data | TC-F-024 | ✅ |
| Idempotent hide/show toggle | TC-E-007 | ✅ |
| Delete CMS — no ghost | TC-F-025 | ✅ |
| Delete Static — no ghost | TC-F-026 | ✅ |
| Deleted project not on portfolio | TC-F-027 | ✅ |
| Full static lifecycle | TC-ST-001 | ✅ |
| Full GitHub lifecycle | TC-ST-002 | ✅ |
| Hidden field in Project schema | TC-F-019 (backend) | ✅ |
| Public GET excludes hidden | TC-F-021 verification | ✅ |
| Admin GET includes hidden | TC-F-049 | ✅ |

---

## 4. News Feed

| Requirement | Test Case(s) | Coverage |
|---|---|---|
| Home news section renders 6 articles | TC-F-029 | ✅ |
| Cards show correct RSS data | TC-F-030 | ✅ |
| Cards link to external articles | TC-F-031 | ✅ |
| Home refresh button | TC-F-032 | ✅ |
| View All → /news page | TC-F-033 | ✅ |
| /news page — all articles | TC-F-034 | ✅ |
| Search by title | TC-F-035 | ✅ |
| Search by description | TC-F-036 | ✅ |
| Search empty state | TC-F-037 | ✅ |
| /news refresh button | TC-F-038 | ✅ |
| Both sources represented | TC-F-039 | ✅ |
| Articles sorted by date | TC-F-040 | ✅ |
| API response structure | TC-F-041 | ✅ |
| 15-min cache | TC-F-042 | ✅ |
| THN feed unreachable → PAL loads | TC-E-008 | ✅ |
| Both feeds unreachable → stale cache | TC-E-009 | ✅ |
| Rate limiting /api/news | TC-ERR-010 | ✅ |

---

## 5. Admin Panel

| Requirement | Test Case(s) | Coverage |
|---|---|---|
| Login with valid credentials | TC-F-043 | ✅ |
| Login with invalid credentials | TC-F-044 | ✅ |
| Login rate limit (5 attempts) | TC-F-045 | ✅ |
| Protected routes block unauth | TC-F-046 | ✅ |
| Token expiry redirect | TC-F-047 | ✅ |
| Logout clears session | TC-F-048 | ✅ |
| Projects page — all sources | TC-F-049 | ✅ |
| Refresh reloads data | TC-F-050 | ✅ |
| Form resets after submit | TC-F-051 | ✅ |
| Edit form pre-fills | TC-F-052 | ✅ |
| Action row disables during op | TC-F-053 | ✅ |
| Admin nav links | TC-F-054 | ✅ |
| Unauthenticated API write | TC-ERR-011 | ✅ |
| 2FA token rejected | TC-ERR-012 | ✅ |
| Orphaned token rejected | TC-ERR-013 | ✅ |

---

## 6. Public Pages

| Requirement | Test Case(s) | Coverage |
|---|---|---|
| Home page all sections | TC-F-055 | ✅ |
| Active section highlight | TC-F-056 | ✅ |
| News nav link → /news | TC-F-057 | ✅ |
| Projects filter tabs | TC-F-058 | ✅ |
| Hidden projects not visible | TC-F-059 | ✅ |
| Blog list page | TC-F-060 | ✅ |
| Blog search | TC-F-061 | ✅ |
| CTF list page | TC-F-062 | ✅ |
| /news direct URL | TC-F-063 | ✅ |
| 404 page | TC-F-064 | ✅ |
| Security Disclosure | TC-F-065 | ✅ |
| PGP Key page | TC-F-066 | ✅ |
| Mobile navbar | TC-E-010 | ✅ |
| Responsive grid | TC-E-011 | ✅ |

---

## Overall Coverage Summary

| Area | Total Requirements | Covered | Coverage % |
|---|---|---|---|
| Projects CMS | 15 | 15 | **100%** |
| GitHub Integration | 13 | 13 | **100%** |
| Hide / Delete | 15 | 15 | **100%** |
| News Feed | 17 | 17 | **100%** |
| Admin Panel | 15 | 15 | **100%** |
| Public Pages | 14 | 14 | **100%** |
| **TOTAL** | **89** | **89** | **100%** |
