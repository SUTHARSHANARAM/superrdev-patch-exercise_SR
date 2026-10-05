# Patch Exercise Notes

## Summary
Fixed the highest-value issues found across the backend, database query, and frontend.

### Bugs Fixed
1. **Task search filter precedence** — Fixed `AND`/`OR` precedence so archived tasks and status filters cannot be bypassed by description matches.
2. **Database pagination** — Replaced in-memory `subList` pagination with Spring Data database-level pagination and a count query.
3. **Artificial request delay** — Removed the controller's calculated `Thread.sleep()` delay that unnecessarily blocked request threads.
4. **Invalid pagination input** — Added validation for `page` and `pageSize` and return HTTP 400 for invalid values.
5. **Invalid status input** — Added handling for unsupported status values and return HTTP 400 instead of an unhandled exception.
6. **Frontend request state** — Reset pagination when filters change, added search debouncing, and prevented stale responses from overwriting newer results.

## What I Did Not Change
I did not rewrite the application architecture or replace the existing technologies. The Oracle SQL artifact was kept as a reference and updated consistently with the application query.

## Biggest Remaining Risk
The application still uses an in-memory H2 database, so data is not persistent across restarts. Production database behavior would also need separate verification.

## Tools / AI Used
Used AI assistance for debugging, identifying likely failure points, reviewing the existing implementation, and validating the fixes. Changes were manually tested through the application UI and API endpoints.
