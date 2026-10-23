// Custom hook — centralises ticket data and related UI state
It is better to pass data from the top down (via props) or through a well-defined global state. Also, it is best to avoid using `unknown[]`, as this will lead to runtime bugs.

// TicketCard — renders a single ticket row
Inserting unsanitized HTML in this way is dangerous, as attackers could inject their own text and execute operations of their choosing.


## AI & Tool Usage
While reviewing the code, I used AI—since I lacked prior experience with `new TicketEventBus()` and unsanitized HTML—which helped me identify an error and provided information on these topics.