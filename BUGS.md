# Bug Reports & Fixes

## Bug 1: Infinite re-render loop and memory leak in polling

### Root Cause
In `src/pages/TicketsListPage.tsx`, `window.setInterval` inside `useEffect` did not have a cleanup function (`clearInterval`). Every time `refetch` changed, a new timer was created without stopping the old one. This created multiple background timers, causing memory leaks, continuous state updates, and heavy UI flickering.

### Reproduction Steps
1. Open the original app (`npm run dev`) and go to the Ticket List page.
2. Observe the page for 5-10 seconds.
3. Notice that the UI starts flickering continuously, text flashes.

### Fix
1. Updated `useEffect` in `TicketsListPage.tsx` to store the timer ID in a local variable.
2. Added a cleanup function `return () => window.clearInterval(timerId);` to stop the timer when the component unmounts or before re-running the effect.

### Before/After Checks

- **Before Fix:** 
  - The UI kept re-rendering, flickering, and resetting state rapidly due to multiple uncleaned background timers running simultaneously.
- **After Fix:** 
  - UI is completely stable and stops flickering.
  - State updates happen predictably at regular 5-second intervals.
  - Manual check confirmed that navigating away and back to the page clears previous timers correctly without memory leaks.


## Bug 2: Duplicate ID generation in `fakeApi.ts` breaks React list rendering

### Root Cause
In `src/lib/fakeApi.ts`, the `lastId` counter was initialized to `2`, even though `defaultTickets` already contained a ticket with `id: 3`. When a user created a new ticket, `++lastId` assigned `id: 3` to the new item, creating two tickets with the exact same ID (`3`). 

Because React uses `ticket.id` as a unique `key` when rendering lists, this duplicate ID broken React's list reconciliation algorithm. When typing text into the Search input (for example, searching for the word "the"), React confused the items with matching keys, resulting in duplicated rows in the UI, incorrect text matching, and corrupted list states.

### Reproduction Steps
1. Open the origin app (`npm run dev`) and create a new ticket (e.g., "New custom ticket").
2. Type a common search term like "the" into the Search input.
3. Observe that existing tickets get duplicated or incorrectly mapped in the list because both the third default ticket and the newly created ticket share `key={3}`.
4. You can find the ticket ID by clicking 'View details' and checking the trailing number in the page URL.

### Fix
Updated `lastId` initialization in `src/lib/fakeApi.ts` from `2` to `3`, ensuring every newly created ticket receives a unique, incremental ID.

### Before/After Checks
- **Before Fix:** Creating a ticket assigned it a duplicate `id: 3`, which broke list rendering and caused duplicate rows to appear when filtering by search terms like "the".
- **After Fix:** Newly created tickets receive unique IDs (`4`, `5`, etc.). Searching and filtering works smoothly without duplicate items or React key collision warnings.

## Bug 3: `TicketDetailPage` stuck in infinite loading due to misplaced `AbortController`

### Root Cause
In `src/hooks/useTicket.ts`, `new AbortController()` was instantiated in the main body of the custom hook rather than inside `useEffect`. Because component re-renders created a new controller instance, the previous effect's cleanup function (`controller.abort()`) was immediately triggered. In the catch block, `if (err.name === 'AbortError') return;` returned early without calling `setLoading(false)`, leaving the component permanently stuck in the loading state.

### Reproduction Steps
1. Navigate to the main ticket list (`/`).
2. Click on "View details" for any ticket (e.g., ticket #1).
3. Observe that the page remains indefinitely on the `<p>Loading...</p>` state and never loads the details.

### Fix
1. Moved `const controller = new AbortController();` inside the `useEffect` scope so each effect execution manages its own lifecycle instance.
2. Verified that unmounting or ID changes abort requests properly without prematurely canceling active fetches on render.

### Before/After Checks
- **Before Fix:** Clicking "View details" caused an infinite loading screen due to immediate request abortion.
- **After Fix:** Details page briefly shows loading, then correctly resolves and displays the ticket information and status control options.