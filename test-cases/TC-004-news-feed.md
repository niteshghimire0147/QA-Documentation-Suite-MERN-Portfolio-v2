# TC-004 — News Feed (Home Section + News Page)

**Feature:** Cybersecurity News — RSS feeds from The Hacker News + Palo Alto Networks
**Author:** Senior QA Engineer
**Date:** 2026-05-09
**Priority:** P1 — High

---

## Preconditions

- Backend running with `rss-parser` installed
- Internet accessible; both RSS feeds reachable:
  - `https://feeds.feedburner.com/TheHackersNews`
  - `https://www.paloaltonetworks.com/blog/feed/`
- `GET /api/news` endpoint responding

---

## HOME SECTION TESTS

### TC-F-029: News section loads on home page

**Priority:** High

**Test Steps:**
1. Navigate to `http://localhost:5173`
2. Scroll to the "// 07. CYBER NEWS" section

**Expected Results:**
- Section renders with title "Security News"
- Loading skeleton cards appear briefly (3 columns × 2 rows)
- Within 5 seconds, real articles replace the skeletons
- Exactly 6 articles are shown (hardcoded `slice(0, 6)`)
- No fallback/placeholder articles (FALLBACK array was removed)

---

### TC-F-030: News cards show correct data from RSS

**Priority:** High

**Test Steps:**
1. After articles load on home page, inspect each card

**Expected Results:**
- Each card shows:
  - Source badge: "The Hacker News" OR "Palo Alto Networks"
  - Time ago: e.g. "3h ago", "2d ago"
  - Title in bold white text
  - Short description (truncated to ~300 chars)
  - "Read more →" footer link
- Thumbnail image loads if available (no broken image icons)

---

### TC-F-031: News cards link to correct external URLs

**Priority:** High

**Test Steps:**
1. Click any news card on the home page

**Expected Results:**
- Opens the original article URL in a new browser tab (`target="_blank"`)
- URL is a valid `https://` link to thehackernews.com or paloaltonetworks.com
- `rel="noopener noreferrer"` is present (security requirement)

---

### TC-F-032: Refresh button fetches new articles

**Priority:** Medium

**Test Steps:**
1. Note the current 6 articles on the home news section
2. Click the "refresh --feed" button
3. Observe the button and article grid

**Expected Results:**
- Button shows spinner animation during fetch
- Button is disabled while loading (no double-click issue)
- After refresh, articles reload (may be same if cached within 15 min)
- No page error or crash

---

### TC-F-033: "View All News" button navigates to /news

**Priority:** High

**Test Steps:**
1. Scroll to the bottom of the home news section
2. Click "View All News →"

**Expected Results:**
- Browser navigates to `/news`
- Full news page renders with all articles from both feeds
- Page title area shows "// CYBER INTEL" and "Security News"

---

## NEWS PAGE TESTS (/news)

### TC-F-034: News page loads with all articles

**Priority:** High

**Test Steps:**
1. Navigate directly to `http://localhost:5173/news`
2. Wait for articles to load

**Expected Results:**
- Page title "Security News" with subtitle "live feed from thehackernews.com"
- All articles from both feeds displayed (not limited to 6)
- Each article shows source badge ("The Hacker News" / "Palo Alto Networks")
- Article count shows "X articles" above the grid
- Both source credits shown at the bottom

---

### TC-F-035: Search filters articles by title

**Priority:** High

**Test Steps:**
1. On the `/news` page, type "malware" in the search bar
2. Observe the article grid

**Expected Results:**
- Only articles containing "malware" in their title or description are shown
- Article count updates (e.g. "3 articles matching 'malware'")
- Articles not matching are hidden
- Clearing the search input restores all articles

---

### TC-F-036: Search filters articles by description

**Priority:** Medium

**Test Steps:**
1. Type a keyword known to be in an article description but not the title
2. Observe results

**Expected Results:**
- Articles containing the keyword in their description are shown
- Combined title + description search works correctly

---

### TC-F-037: Search with no matches shows empty state

**Priority:** Medium

**Test Steps:**
1. Type "xyzzyxyzzyxyzzy" in the search bar (guaranteed no match)

**Expected Results:**
- Empty state shown with 📡 emoji and "// No articles match your search."
- Article count shows "0 articles matching 'xyzzyxyzzyxyzzy'"
- No errors or crashes

---

### TC-F-038: Refresh button on /news page

**Priority:** Medium

**Test Steps:**
1. Click the "Refresh" button on the `/news` page
2. Observe the spinner and response

**Expected Results:**
- FiRefreshCw icon spins during request
- Button disabled while loading
- Articles reload after completion
- "last fetched X ago" timestamp updates

---

### TC-F-039: Both sources represented in article list

**Priority:** High

**Test Steps:**
1. Load the `/news` page
2. Scan all visible article source badges

**Expected Results:**
- At least one article with badge "The Hacker News"
- At least one article with badge "Palo Alto Networks"
- Articles are sorted newest-first across both sources

---

### TC-F-040: Articles sorted by date (newest first)

**Priority:** High

**Test Steps:**
1. Load the `/news` page
2. Check the "time ago" and date values on the first 5 articles

**Expected Results:**
- Articles appear in descending date order
- Most recent article is at position 1
- Older articles follow in order
- No date inversion between adjacent cards

---

## BACKEND API TESTS

### TC-F-041: GET /api/news returns correct structure

**Priority:** High

**Test Steps:**
1. Call `GET /api/news` directly

**Expected Results:**
- HTTP 200
- Response body: `{ articles: [...], sources: [...], fetchedAt: "ISO string" }`
- `articles` is an array
- Each article has: `title`, `link`, `pubDate`, `description`, `thumbnail`, `categories`, `source`
- `sources` array: `["The Hacker News", "Palo Alto Networks"]`

---

### TC-F-042: Cache serves stale data within TTL

**Priority:** Medium

**Test Steps:**
1. Call `GET /api/news` — note `fetchedAt` timestamp
2. Immediately call `GET /api/news` again

**Expected Results:**
- Second response is instant (from cache)
- `fetchedAt` timestamp is identical to first response
- Backend does not make a new RSS fetch within 15-minute TTL

---

### TC-E-008: One RSS feed unreachable — other still loads

**Priority:** High

**Test Steps:**
1. Block outbound requests to `feeds.feedburner.com` only
2. Call `GET /api/news`

**Expected Results:**
- API returns `200 OK` (not a 500 error)
- `articles` array contains Palo Alto Networks articles only
- THN articles are absent but no error is thrown
- `fetchedAt` is set normally

---

### TC-E-009: Both RSS feeds unreachable — stale cache returned

**Priority:** High

**Test Steps:**
1. Load news once to populate cache
2. Block both RSS feeds
3. Wait for cache to expire (>15 min) or manually clear cache
4. Call `GET /api/news`

**Expected Results:**
- If stale cache exists: returns `{ ...cached_data, stale: true }`
- If no cache at all: returns `{ articles: [], sources: [], error: 'Feed unavailable' }` with HTTP 200
- Frontend shows empty state gracefully (no crash)

---

### TC-ERR-010: Rate limit on /api/news

**Priority:** Medium

**Test Steps:**
1. Send 31 rapid requests to `GET /api/news` within 10 minutes

**Expected Results:**
- First 30 requests: HTTP 200
- 31st request: HTTP 429 with `{ "message": "Too many news requests. Try again later." }`

---

## Coverage Matrix

| Requirement | Test Cases |
|---|---|
| News section on home page loads | TC-F-029 |
| Cards show correct RSS data | TC-F-030 |
| Cards link to external articles | TC-F-031 |
| Refresh button | TC-F-032, TC-F-038 |
| View All link | TC-F-033 |
| News page — all articles | TC-F-034 |
| Search by title | TC-F-035 |
| Search by description | TC-F-036 |
| Search empty state | TC-F-037 |
| Both sources represented | TC-F-039 |
| Sorted by date | TC-F-040 |
| API response structure | TC-F-041 |
| Cache behaviour | TC-F-042 |
| Partial feed failure | TC-E-008 |
| Full feed failure | TC-E-009 |
| Rate limiting | TC-ERR-010 |
