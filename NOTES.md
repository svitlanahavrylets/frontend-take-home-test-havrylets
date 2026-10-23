# My notes & thinking

## Bug 1: Infinite re-render loop and memory leak in polling

When I first ran the project, I immediately noticed that the page was flickering and glitching. I understood that this was caused by an excessive number of page re-renders. 

Since I hadn't worked with frontend for a while, I decided to refresh my knowledge and researched the common reasons behind page flickering in React. This led me to suspect an issue inside `useEffect`. 

I re-read documentation about dependency arrays, `setInterval`, and side effects in React. That's when I realized the effect was missing a cleanup function (`clearInterval`).

I initially added `return () => window.clearInterval(pollRef.current);` to stop the interval on re-renders.
After reading more about how closures work inside `useEffect`, I learned that relying on a mutable ref (`pollRef.current`) during cleanup can be risky if the ref value changes. I decided it was cleaner and safer to store the interval ID in a local variable (`const timerId = window.setInterval(...)`) to lock the value in the effect's closure.


## AI & Tool Usage
-  Used Google Search (with AI Overviews / AI search features) to refresh my knowledge about React concepts and research the common causes of page flickering, `useEffect` dependency array behavior, and `setInterval`.
- Used Gemini to refine and accurately translate my thoughts and technical explanations into clear English for documentation.
- Tested and verified all fixes locally in the browser window while running the project (`npm run dev`), checking both the UI stability and request logs in Chrome DevTools.

## Bug 2: Duplicate ID generation in `fakeApi.ts` breaks React list rendering

When I moved on to testing ticket creation and filtering, I noticed a strange bug: after attempting to filter the list, new tickets seemed to duplicate or appear unexpectedly. 

By clicking "View details" and checking the URL, I noticed that the ticket ID numbering was corrupted (there were duplicate IDs). 

After updating `let lastId = 3;` in the `createFakeTicketingApi` function (since there are 3 default tickets pre-loaded by default), the app started working properly without any duplication or rendering glitches.

## AI & Tool Usage
Since this was a deeper logic issue within the mock API setup, I delegated searching for the exact root cause in the code to AI. 


## Bug 3: `TicketDetailPage` stuck in infinite loading due to misplaced `AbortController`

While verifying the previous bug with ticket IDs, I noticed that the details page was not displaying any ticket content at all. It was stuck permanently showing only the "Loading..." message, even though the component code had full markup ready for rendering.

I started investigating the custom hook `useTicket.ts`, suspecting that `setLoading(false)` was not being called properly. 

I discovered that `new AbortController()` was defined outside the `useEffect` hook, in the main scope of `useTicket`. Because of this, every component re-render created a new controller instance, causing the cleanup function (`controller.abort()`) to execute immediately. Since the catch block ignored `AbortError` without updating the state, `loading` remained `true` forever.

I fixed this by moving `const controller = new AbortController();` inside `useEffect`. This ensured proper request scoping, resolved the infinite loading loop, and allowed the ticket details to display correctly.

## AI & Tool Usage
Since I hadn't worked with AbortController before, I researched how it works using Google Search. This link provides a very detailed explanation of it:
https://javascript.plainenglish.io/abortcontroller-and-why-we-even-need-it-fe7ec8465886


