# Worked PR examples

Concrete before/after examples. The "before" versions are the common failure modes;
the "after" versions show the practices in the SKILL applied.

## Example 1 — Bug fix (small)

**Weak:**
```
Title: Fixed bug

## Summary
Modified cache.py to fix the issue. Updated the eviction logic and changed how
timestamps are compared.

## Test plan
Tested locally.
```

**Good:**
```
Title: Fix race condition in session cache eviction

## Summary
Concurrent requests could evict a session entry that another request had just
refreshed, causing sporadic logouts under load. The eviction check now compares the
entry's refreshed-at timestamp under the same lock used to write it, closing the window.

Closes #482

## Test plan
- `pytest tests/cache/test_eviction.py` (added a test that reproduces the race with 50
  concurrent refresh/evict pairs — fails on main, passes here)
- Ran the staging load test (200 rps for 5 min); no spurious logouts vs. ~12/min before
```

Why it's better: the title is scannable, the summary leads with the *problem* and only
then the fix, and the test plan is specific and honest about how it was verified.

## Example 2 — Feature (medium)

**Weak:**
```
Title: Add export feature

## Summary
This PR adds a great new export feature. It modifies the reports module, adds a new
endpoint, updates the router, adds a service class, and updates the tests.

## Test plan
Everything works.
```

**Good:**
```
Title: Add CSV export for the reports dashboard

## Summary
Users have been copy-pasting report tables into spreadsheets by hand (top support
request this quarter). This adds a "Download CSV" action backed by a new
`GET /reports/{id}/export` endpoint. Streaming the rows so large reports don't buffer
the whole file in memory.

Out of scope: XLSX/PDF export — tracked in #517.

## Test plan
- [x] `pytest tests/reports/test_export.py`
- [x] Exported a 100k-row report locally; memory stayed flat (streamed)
- [x] Verified UTF-8 and embedded-comma cells quote correctly
- [ ] Needs a quick check on the staging dataset before merge
```

Why it's better: motivation is concrete, the summary describes behavior rather than
listing touched files, scope boundaries are explicit, and the checklist honestly marks
one item as not-yet-done.

## Example 3 — Tiny change

**Overcooked:**
```
Title: chore: documentation update

## Summary
This pull request updates the documentation to improve clarity and correctness for
end users reading the installation guide.

## Test plan
N/A — documentation only.
```

**Good:**
```
Title: Fix broken install link in README

Points the "getting started" link at /docs/install (the old /setup path 404s).
```

Why it's better: a one-line change deserves a one-line PR. The headers and "N/A" test
plan were pure ceremony.
