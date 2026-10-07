# Notes on Patch & Architecture

### Summary of Changes
- **SQL Operator Precedence**: Fixed missing parentheses around `LOWER(title) LIKE ... OR LOWER(description) LIKE ...` in `TaskRepository.java`, `search_tasks.sql`, and `task_search_package.sql`. This prevents archived tasks from leaking into results and ensures the status filter is respected on title matches.
- **Backend Thread Blocking & Validation**: Removed artificial `Thread.sleep` in `TaskController.java` that blocked worker threads for 1s on short queries. Added safe handling for invalid status strings (returning HTTP 400 instead of 500) and guarded pagination parameters against negative bounds.
- **Frontend Race Conditions & State Reset**: Added a subscription cancellation flag in `useTasks.js` to prevent stale asynchronous responses from overwriting newer search results. Ensured `loading` is set to `false` and error state clears properly on errors.
- **Search Debounce & Pagination Reset**: Debounced query updates (300ms) in `App.jsx` to reduce redundant network requests, and reset `page` to 1 whenever search query or status filter changes so users never get trapped on empty pages.

### What I Chose Not to Change
- **In-Memory Slicing**: Retained Java-level pagination (`subList`) instead of rewriting the repository to use SQL `LIMIT`/`OFFSET` or Spring Data `Pageable` to keep the patch focused and preserve existing API response contracts without risking query regressions.
- **Styling & Structure**: Avoided adding CSS frameworks or refactoring standard UI components, preserving the original folder structure and design.

### Biggest Remaining Risk
- **Scalability with In-Memory Pagination**: Loading the entire matching dataset into JVM memory before slicing will cause high memory pressure and slow queries once the tasks table grows beyond thousands of records. Database-level pagination with appropriate composite indexes on `(archived, status, created_at)` is necessary for production scale.

### Tools & AI Used
- Used AI to locate the SQL precedence bug across native queries and reference PL/SQL package, review race conditions in `useTasks`, and draft the debouncing logic in `App.jsx`.
