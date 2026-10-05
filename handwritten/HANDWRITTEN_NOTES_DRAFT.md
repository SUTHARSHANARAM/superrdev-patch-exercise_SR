# Patch Exercise — Handwritten Notes Draft

## 1. SQL Search Filter AND/OR Precedence

- **Where**: `TaskRepository.java` (`searchTasks` query), `db/queries/search_tasks.sql`, and `db/oracle/task_search_package.sql`.
- **How Discovered**: Filtered by status while searching terms that matched descriptions of archived or other status tasks; noticed archived and wrong-status tasks appeared in results.
- **Root Cause**: SQL evaluates `AND` before `OR`. Without parentheses around title and description matches, description matches bypassed `archived = FALSE` and status filters.
- **Fix**: Added parentheses `(LOWER(title) LIKE :term OR LOWER(description) LIKE :term)` around title and description conditions in all SQL query files.
- **Why Chosen**: Ensures search term matching is evaluated as a single group so archived and status filters apply to all search results.

---

## 2. Database-Level Pagination

- **Where**: `TaskRepository.java` and `TaskController.java` (`searchTasks` method).
- **How Discovered**: Code inspection showed `TaskRepository` returned all matching tasks in a `List`, and `TaskController` paginated them in Java using `subList()`.
- **Root Cause**: Loading all records from the database into Java memory to trim results causes high memory usage and poor performance as data grows.
- **Fix**: Replaced `List<Task>` return type with Spring Data `Page<Task>`, added `Pageable` parameter and `countQuery` to repository, and used `PageRequest.of(page - 1, pageSize)` in controller.
- **Why Chosen**: Lets the SQL database handle `LIMIT`, `OFFSET`, and total count, returning only the requested page items to the application.

---

## 3. Removal of Artificial Request Delay

- **Where**: `TaskController.java` (`searchTasks` method).
- **How Discovered**: Testing API endpoints and inspecting controller code revealed a calculated `Thread.sleep()` block.
- **Root Cause**: An artificial `Thread.sleep()` was intentionally added to delay response times and block request threads.
- **Fix**: Deleted the `Thread.sleep()` code block completely from `TaskController.java`.
- **Why Chosen**: Removing artificial sleeping frees worker threads immediately and eliminates unnecessary request latency.

---

## 4. Validation of Invalid Page and PageSize

- **Where**: `TaskController.java` (`searchTasks` method).
- **How Discovered**: Testing requests with `page=0` or invalid/non-positive values triggered internal exceptions when building page requests.
- **Root Cause**: Controller accepted integer parameters without checking that `page` and `pageSize` were positive values (`>= 1`).
- **Fix**: Added `if (page < 1 || pageSize < 1)` check that returns an HTTP 400 Bad Request response with error message JSON.
- **Why Chosen**: Prevents invalid values from reaching Spring Data and returns clear HTTP 400 responses instead of 500 server errors.

---

## 5. Handling of Invalid Task Status Values

- **Where**: `TaskController.java` and `TaskStatus.java` enum.
- **How Discovered**: Passing an unknown status string (e.g., `status=INVALID`) caused an uncaught `IllegalArgumentException` from `TaskStatus.valueOf()`, resulting in an HTTP 500 server error.
- **Root Cause**: `TaskStatus.valueOf()` threw an exception for invalid status strings without a `try-catch` block.
- **Fix**: Wrapped status enum lookup in a `try-catch` block catching `IllegalArgumentException` and returning an HTTP 400 Bad Request response.
- **Why Chosen**: Safely handles invalid user input and returns meaningful HTTP 400 error messages to clients.

---

## 6. Frontend Pagination Reset, Debouncing, Stale Response Protection & States

- **Where**: `useTasks.js` hook and `App.jsx` component.
- **How Discovered**: Code inspection showed that stale responses could overwrite newer results, and testing showed changing search/status while on page 2 left the UI on page 2.
- **Root Cause**: Keystrokes sent immediate API calls without delay, async callbacks lacked a flag to ignore stale responses, and filter controls did not reset `page` state.
- **Fix**: Added 300ms `setTimeout` debounce and `cancelled` boolean flag in `useEffect` cleanup in `useTasks.js`, managed loading/error state, and added `setPage(1)` to filter handlers in `App.jsx`.
- **Why Chosen**: Reduces redundant API calls, ignores stale responses, and ensures search results start on page 1.

---

## Testing Performed

- Verified UI status filter change resets pagination back to page 1
- Verified search bar text filtering correctly returns matching tasks
- Tested `page=0` request and confirmed API returns HTTP 400 error response
- Tested invalid status value and confirmed API returns HTTP 400 error response
- Tested normal pagination (`page=1`, `page=2`) and verified correct page items and totals returned
- Verified archived tasks are not returned in search results through corrected search filtering
