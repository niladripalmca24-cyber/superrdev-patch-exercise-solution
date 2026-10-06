# Notes

### Summary of Changes
- **SQL Operator Precedence**: Wrapped `(LOWER(title) LIKE :term OR LOWER(description) LIKE :term)` with parentheses in `TaskRepository.java`, `search_tasks.sql`, and `task_search_package.sql`. This prevents archived tasks from leaking when descriptions match and ensures the status filter is not bypassed.
- **Backend Latency & Robustness**: Removed the artificial blocking `Thread.sleep` in `TaskController.java`. Added defensive pagination boundary checks (`safePage >= 1`, bounded `safePageSize`) to eliminate `IndexOutOfBoundsException`, and safe status parsing to prevent unhandled 500 errors.
- **Frontend Race Conditions & State Cleanup**: In `useTasks.js`, implemented an active flag cancellation pattern in `useEffect` to ignore out-of-order stale network responses. Ensured `loading` always resets to `false` on error and cleared stale errors on new requests.
- **UX & Performance**: Added a reusable `useDebounce` hook (300ms) to throttle search keystrokes and avoid redundant network requests. Reset `page` to 1 whenever search query or status filter changes in `App.jsx`.

### What Was Not Changed & Why
- Maintained the existing controller response structure and in-memory pagination (`subList`) rather than refactoring to Spring Data `Pageable` to maintain contract stability and focus on high-impact bug fixes.
- Preserved existing CSS styling without a visual overhaul, prioritizing search correctness, race condition elimination, and stability within the timebox.

### Biggest Remaining Risk
- **In-Memory Pagination Scalability**: Fetching all matching records into memory before slicing causes substantial JVM memory pressure and DB latency as the dataset scales.
- **Missing Authentication & Rate Limiting**: The `/api/tasks` endpoint is open without rate limits or authorization, leaving the service vulnerable to DoS attacks.

### Tools & AI Used
- Used AI to assist in identifying subtle SQL operator precedence and drafting the `useDebounce` hook.
- Manually reviewed and verified the H2/Oracle query logic, adjusted debounce timing (300ms), and added pagination boundary validation.
