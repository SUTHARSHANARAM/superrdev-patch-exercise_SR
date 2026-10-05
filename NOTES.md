# Patch Notes

## Summary of changes

- Fixed task-search SQL precedence so archived tasks and the optional status filter apply to both title and description matches. Updated the H2 query, repository query, and Oracle reference artifact consistently.
- Moved pagination into the database through Spring Data `Page`/`Pageable` instead of loading all matching rows into memory and slicing in Java.
- Removed the artificial query-length-based `Thread.sleep` from the controller.
- Added 400 responses for invalid pagination and invalid task status values.
- Reset the UI to page 1 when search/status filters change. Added a short search debounce and protected state updates from stale/obsolete requests.

## What I chose not to change

I did not rewrite the API shape, introduce a new state-management library, or change the UI structure. I also left the Oracle package as a reference artifact and only aligned its search predicate with the application query.

## Biggest remaining risk

The endpoint still accepts an unrestricted `pageSize`, so a caller can request a very large page. In a production API I would add a maximum page size and likely move validation into a dedicated request/exception-handling layer.

## AI/tools used

I used ChatGPT to help inspect the code, identify likely defects, reason about SQL operator precedence and pagination, and draft focused code changes. I reviewed and adjusted the changes to fit the existing project structure and requirements, then verified the resulting files locally where possible.
