### TC-001: Book one available event successfully
**Category**: Happy Path
**Priority**: P0
**Preconditions**: Authenticated user; an event has at least one available seat.
**Steps**: 1. Open the events list. 2. Select an event with a visible Book Now action. 3. Enter a valid name, email, phone, and quantity of 1. 4. Confirm the booking.
**Expected Results**: A confirmed booking is created, a unique booking reference is displayed, and the confirmation view offers View My Bookings and Browse Events actions.
**Business Rule**: A booking requires valid customer data and a quantity from 1 through 10.
**Suggested Layer**: E2E

### TC-002: Display a newly created booking in the bookings list
**Category**: Happy Path
**Priority**: P0
**Preconditions**: Authenticated user has a confirmed booking.
**Steps**: 1. Open My Bookings. 2. Locate the booking by its reference.
**Expected Results**: The card shows the event, customer details, quantity, total price, confirmed status, and booking reference.
**Business Rule**: Users can view all of their own bookings, including event details.
**Suggested Layer**: E2E

### TC-003: Open full booking details
**Category**: Happy Path
**Priority**: P1
**Preconditions**: Authenticated user has a confirmed booking.
**Steps**: 1. Open My Bookings. 2. Select View Details for a booking.
**Expected Results**: The detail page shows the booking reference, event details, customer details, quantity, total paid, status, and refund eligibility control.
**Business Rule**: A booking detail lookup returns the booking only to its owner.
**Suggested Layer**: E2E

### TC-004: Navigate to bookings from the booking confirmation
**Category**: Happy Path
**Priority**: P1
**Preconditions**: A booking has just been confirmed.
**Steps**: 1. Select View My Bookings on the confirmation card.
**Expected Results**: The user is taken to the bookings list and the new booking is visible.
**Business Rule**: After booking, users can reach their bookings from the confirmation view.
**Suggested Layer**: E2E

### TC-005: Look up a booking by its reference through the API
**Category**: Happy Path
**Priority**: P2
**Preconditions**: Authenticated user owns a booking and has its reference.
**Steps**: 1. Send GET `/api/bookings/ref/:ref` with the bearer token and reference. 2. Compare the returned booking with the created booking.
**Expected Results**: The API returns HTTP 200 and the matching booking with its event.
**Business Rule**: Booking references uniquely identify bookings.
**Suggested Layer**: API

### TC-006: List bookings with pagination
**Category**: Happy Path
**Priority**: P1
**Preconditions**: Authenticated user owns more bookings than the requested page limit.
**Steps**: 1. Request GET `/api/bookings?page=1&limit=2`. 2. Request the next page. 3. Inspect the response metadata.
**Expected Results**: Each page contains no more than the requested limit, results are ordered newest first, and pagination reports total, page, limit, and totalPages.
**Business Rule**: Bookings are returned as a paginated user-scoped collection.
**Suggested Layer**: API

### TC-007: Filter bookings by event
**Category**: Happy Path
**Priority**: P2
**Preconditions**: Authenticated user has bookings for at least two events.
**Steps**: 1. Request GET `/api/bookings?eventId=<id>`. 2. Inspect every returned booking.
**Expected Results**: Every result belongs to the requested event and pagination total reflects only matching bookings.
**Business Rule**: The bookings endpoint supports filtering by event ID.
**Suggested Layer**: API

### TC-008: Filter bookings by status
**Category**: Happy Path
**Priority**: P2
**Preconditions**: Authenticated user has bookings available to the status filter.
**Steps**: 1. Request GET `/api/bookings?status=confirmed`. 2. Inspect every returned booking.
**Expected Results**: Every result has confirmed status and no unrelated user booking is returned.
**Business Rule**: The bookings endpoint supports status filtering.
**Suggested Layer**: API

### TC-009: Cancel one booking from its detail page
**Category**: Happy Path
**Priority**: P0
**Preconditions**: Authenticated user owns one confirmed booking.
**Steps**: 1. Open the booking detail page. 2. Select Cancel Booking. 3. Confirm the cancellation dialog.
**Expected Results**: The booking is deleted, a success message is shown, the user returns to the bookings list, and the cancelled booking is absent.
**Business Rule**: Individual booking cancellation deletes the booking.
**Suggested Layer**: E2E

### TC-010: Clear all bookings
**Category**: Happy Path
**Priority**: P0
**Preconditions**: Authenticated user owns one or more bookings.
**Steps**: 1. Open My Bookings. 2. Select Clear All Bookings. 3. Confirm the browser dialog.
**Expected Results**: All of the user’s bookings are deleted and the empty state with Browse Events is displayed.
**Business Rule**: Clear All Bookings removes all bookings for the authenticated user in one action.
**Suggested Layer**: E2E

### TC-100: Calculate total price from quantity
**Category**: Business Rule
**Priority**: P0
**Preconditions**: An event has a known per-ticket price and enough seats.
**Steps**: 1. Book the event with quantity 1. 2. Repeat with a valid quantity greater than 1. 3. Compare each booking total with event price multiplied by quantity.
**Expected Results**: `totalPrice` equals `event.price x quantity` for every booking.
**Business Rule**: Price is stored per ticket and total price is calculated server-side.
**Suggested Layer**: API

### TC-101: Generate a correctly formatted booking reference
**Category**: Business Rule
**Priority**: P0
**Preconditions**: An event title begins with a known letter and a booking can be created.
**Steps**: 1. Create a booking. 2. Validate the reference format and prefix.
**Expected Results**: The reference matches `[EVENT_TITLE_FIRST_LETTER]-[6 uppercase alphanumeric characters]` and is unique.
**Business Rule**: The first reference character is the uppercase first character of the event title.
**Suggested Layer**: Unit

### TC-102: Reduce available seats after booking
**Category**: Business Rule
**Priority**: P0
**Preconditions**: Authenticated user can inspect an event with known availability.
**Steps**: 1. Record the event availability. 2. Book quantity N. 3. Reload the event detail.
**Expected Results**: The same user sees availability reduced by N immediately after confirmation.
**Business Rule**: Booking quantity reduces available seats immediately.
**Suggested Layer**: E2E

### TC-103: Compute dynamic event seats per user
**Category**: Business Rule
**Priority**: P0
**Preconditions**: User A owns a dynamic event with known total seats; User B can access the same shared event where applicable.
**Steps**: 1. User A books quantity N. 2. User A reloads the event. 3. User B inspects the event and books independently.
**Expected Results**: User A’s availability is total seats minus User A’s bookings; another user’s calculation is not reduced by User A’s bookings.
**Business Rule**: Dynamic event availability is computed from the current user’s booking quantities.
**Suggested Layer**: API

### TC-104: Enforce the nine-booking FIFO limit
**Category**: Business Rule
**Priority**: P0
**Preconditions**: Authenticated user has nine bookings and their creation order is known.
**Steps**: 1. Create a tenth booking. 2. List the user’s bookings.
**Expected Results**: The newest booking exists, the total remains at nine, and the oldest booking is automatically removed.
**Business Rule**: Each user may hold a maximum of nine bookings; overflow prunes the oldest booking.
**Suggested Layer**: API

### TC-105: Prefer pruning a different event at the booking limit
**Category**: Business Rule
**Priority**: P1
**Preconditions**: User has nine bookings, including an older booking for another event and a booking for the event being booked.
**Steps**: 1. Create another booking for the existing event. 2. List bookings and inspect seat effects.
**Expected Results**: The oldest booking from a different event is pruned first; the same-event fallback is used only when no other event booking exists.
**Business Rule**: Booking-limit FIFO pruning prefers an oldest booking excluding the target event.
**Suggested Layer**: Unit

### TC-106: Restore effective availability after cancellation
**Category**: Business Rule
**Priority**: P1
**Preconditions**: User has a booking for a dynamic event.
**Steps**: 1. Record availability. 2. Cancel the booking. 3. Reload the event detail.
**Expected Results**: The cancelled quantity no longer contributes to the user’s booked quantity and availability increases accordingly.
**Business Rule**: Booking deletion immediately frees seats.
**Suggested Layer**: API

### TC-107: Determine single-ticket refund eligibility
**Category**: Business Rule
**Priority**: P1
**Preconditions**: User owns a booking with quantity 1.
**Steps**: 1. Open booking details. 2. Select Check Refund Eligibility. 3. Wait for the eligibility result.
**Expected Results**: A spinner is shown during processing, then the result says the booking is eligible for a full refund.
**Business Rule**: Single-ticket bookings qualify for a full refund using client-side logic.
**Suggested Layer**: Component

### TC-108: Reject multi-ticket refund eligibility
**Category**: Business Rule
**Priority**: P1
**Preconditions**: User owns a booking with quantity greater than 1.
**Steps**: 1. Open booking details. 2. Select Check Refund Eligibility. 3. Wait for the result.
**Expected Results**: A spinner is shown, then the result says group bookings are non-refundable and includes the ticket count.
**Business Rule**: Bookings with more than one ticket are not refundable.
**Suggested Layer**: Component

### TC-109: Keep booking status confirmed after creation
**Category**: Business Rule
**Priority**: P1
**Preconditions**: A booking request succeeds.
**Steps**: 1. Inspect the create response. 2. Open the booking in the UI.
**Expected Results**: The booking status is confirmed in both API and UI.
**Business Rule**: Newly created bookings have confirmed status.
**Suggested Layer**: API

### TC-110: Preserve static-event seat behavior
**Category**: Business Rule
**Priority**: P2
**Preconditions**: A seeded static event is available.
**Steps**: 1. Record the static event’s displayed availability. 2. Book tickets. 3. Inspect the event from another user.
**Expected Results**: Static event availability follows its fixed database value and is not changed by per-user booking calculations.
**Business Rule**: Static event seats are fixed and shared; static events are immutable.
**Suggested Layer**: API

### TC-200: Require authentication to list bookings
**Category**: Security
**Priority**: P0
**Preconditions**: No bearer token is supplied.
**Steps**: 1. Send GET `/api/bookings` without authentication. 2. Open `/bookings` in a logged-out browser.
**Expected Results**: The API returns HTTP 401 Unauthorized and the UI requires login rather than exposing booking data.
**Business Rule**: All booking endpoints require a bearer token.
**Suggested Layer**: API

### TC-201: Prevent cross-user booking detail access
**Category**: Security
**Priority**: P0
**Preconditions**: User A owns a booking; User B has a valid account but does not own it.
**Steps**: 1. Capture User A’s booking ID and reference. 2. Authenticate as User B. 3. Request both the ID and reference endpoints.
**Expected Results**: Both requests return HTTP 403 Forbidden or an Access Denied response; User B cannot see booking data.
**Business Rule**: Cross-user booking access is forbidden.
**Suggested Layer**: API

### TC-202: Isolate booking lists between users
**Category**: Security
**Priority**: P0
**Preconditions**: User A and User B each have at least one booking.
**Steps**: 1. List bookings as User A. 2. List bookings as User B. 3. Compare references and IDs.
**Expected Results**: Each response contains only the authenticated user’s bookings.
**Business Rule**: Each user operates in an isolated booking sandbox.
**Suggested Layer**: API

### TC-203: Prevent cross-user booking cancellation
**Category**: Security
**Priority**: P0
**Preconditions**: User A owns a booking; User B is authenticated.
**Steps**: 1. User B sends DELETE `/api/bookings/:id` using User A’s booking ID. 2. User A lists bookings.
**Expected Results**: User B receives HTTP 403 or not-found authorization behavior and User A’s booking remains intact.
**Business Rule**: Users may cancel only their own bookings.
**Suggested Layer**: API

### TC-204: Restrict clear-all to the authenticated user
**Category**: Security
**Priority**: P0
**Preconditions**: User A and User B each own bookings.
**Steps**: 1. User A calls DELETE `/api/bookings`. 2. List bookings as both users.
**Expected Results**: Only User A’s bookings are deleted; User B’s bookings remain.
**Business Rule**: Clear All Bookings deletes records for the current user only.
**Suggested Layer**: API

### TC-205: Prevent tampering with another user’s dynamic event booking
**Category**: Security
**Priority**: P1
**Preconditions**: User A owns a dynamic event; User B has a valid token.
**Steps**: 1. User B submits a booking request using User A’s event ID. 2. Inspect the response and both users’ bookings.
**Expected Results**: User B cannot book an event that is not visible in their sandbox, and no unauthorized booking is created.
**Business Rule**: Dynamic events and bookings are user-isolated; static events are the shared exception.
**Suggested Layer**: API

### TC-300: Reject missing booking fields
**Category**: Negative
**Priority**: P1
**Preconditions**: Authenticated user and a valid event ID.
**Steps**: 1. Submit booking requests with each required field omitted in turn. 2. Inspect each response.
**Expected Results**: Each request returns HTTP 400 with validation details naming the missing field; no booking is created.
**Business Rule**: Event ID, customer name, email, phone, and quantity are required.
**Suggested Layer**: API

### TC-301: Reject invalid customer name
**Category**: Negative
**Priority**: P1
**Preconditions**: Authenticated user and a valid event.
**Steps**: 1. Submit a one-character name. 2. Submit a blank or whitespace-only name.
**Expected Results**: Each request returns HTTP 400 with the minimum-length or required-field validation error.
**Business Rule**: Customer name must contain at least two characters after trimming.
**Suggested Layer**: API

### TC-302: Reject invalid customer email
**Category**: Negative
**Priority**: P1
**Preconditions**: Authenticated user and a valid event.
**Steps**: 1. Submit malformed email values. 2. Inspect the response.
**Expected Results**: HTTP 400 identifies customerEmail as invalid and no booking is created.
**Business Rule**: Customer email must use valid email format.
**Suggested Layer**: API

### TC-303: Reject invalid customer phone
**Category**: Negative
**Priority**: P1
**Preconditions**: Authenticated user and a valid event.
**Steps**: 1. Submit fewer than 10 characters. 2. Submit disallowed alphabetic or punctuation characters.
**Expected Results**: HTTP 400 identifies the phone validation failure and no booking is created.
**Business Rule**: Phone must be at least 10 characters and contain only digits, plus, hyphen, spaces, or parentheses.
**Suggested Layer**: API

### TC-304: Reject quantity below one
**Category**: Negative
**Priority**: P1
**Preconditions**: Authenticated user and a valid event.
**Steps**: 1. Submit quantity 0. 2. Submit a negative quantity.
**Expected Results**: HTTP 400 reports that quantity must be an integer between 1 and 10.
**Business Rule**: The minimum booking quantity is one ticket.
**Suggested Layer**: API

### TC-305: Reject quantity above ten
**Category**: Negative
**Priority**: P1
**Preconditions**: Authenticated user and a valid event.
**Steps**: 1. Submit quantity 11. 2. Submit a larger quantity.
**Expected Results**: HTTP 400 reports the maximum quantity violation and no booking is created.
**Business Rule**: The maximum booking quantity is ten tickets.
**Suggested Layer**: API

### TC-306: Reject non-integer quantity and event ID
**Category**: Negative
**Priority**: P1
**Preconditions**: Authenticated user.
**Steps**: 1. Submit a decimal or text quantity. 2. Submit a decimal, zero, or text event ID.
**Expected Results**: HTTP 400 identifies the integer or positive-ID validation failure.
**Business Rule**: Quantity is an integer from 1 to 10 and event ID is a positive integer.
**Suggested Layer**: API

### TC-307: Reject booking for a nonexistent event
**Category**: Negative
**Priority**: P1
**Preconditions**: Authenticated user; event ID does not exist or is inaccessible.
**Steps**: 1. Submit an otherwise valid booking with that event ID.
**Expected Results**: The API returns HTTP 404 and no booking is created.
**Business Rule**: A booking can reference only an event visible to the authenticated user.
**Suggested Layer**: API

### TC-308: Reject booking when requested seats exceed availability
**Category**: Negative
**Priority**: P0
**Preconditions**: An event has fewer available seats than the requested quantity.
**Steps**: 1. Submit the booking. 2. Inspect the response and event availability.
**Expected Results**: HTTP 400 reports insufficient seats, no booking is created, and availability is not reduced.
**Business Rule**: Requested quantity cannot exceed the user-specific available seats.
**Suggested Layer**: API

### TC-309: Handle unknown booking ID
**Category**: Negative
**Priority**: P1
**Preconditions**: Authenticated user; booking ID does not exist.
**Steps**: 1. Request the booking detail endpoint. 2. Attempt cancellation with the same ID.
**Expected Results**: Both operations return a not-found response and do not alter any booking.
**Business Rule**: Detail and cancellation operations require an existing booking.
**Suggested Layer**: API

### TC-310: Handle unknown booking reference
**Category**: Negative
**Priority**: P2
**Preconditions**: Authenticated user; reference does not exist.
**Steps**: 1. Request GET `/api/bookings/ref/<unknown-ref>`.
**Expected Results**: HTTP 404 reports that the booking reference was not found.
**Business Rule**: Reference lookup returns only an existing unique reference.
**Suggested Layer**: API

### TC-311: Do not cancel when the confirmation is declined
**Category**: Negative
**Priority**: P1
**Preconditions**: Authenticated user owns a booking.
**Steps**: 1. Select Cancel Booking. 2. Decline the confirmation dialog.
**Expected Results**: The booking remains in the list and no cancellation success message is shown.
**Business Rule**: Cancellation requires explicit user confirmation.
**Suggested Layer**: E2E

### TC-312: Reject malformed authorization
**Category**: Negative
**Priority**: P0
**Preconditions**: Booking endpoint is available.
**Steps**: 1. Send requests with an invalid, expired, or malformed bearer token.
**Expected Results**: Each request returns HTTP 401 and does not expose booking data.
**Business Rule**: Booking routes are protected by JWT authentication.
**Suggested Layer**: API

### TC-400: Book exactly one ticket
**Category**: Edge Case
**Priority**: P1
**Preconditions**: Event has at least one available seat.
**Steps**: 1. Submit a booking with quantity 1. 2. Verify the booking and event availability.
**Expected Results**: The booking succeeds, has a one-ticket total, and reduces availability by one.
**Business Rule**: Quantity 1 is the lower valid boundary and is refund eligible.
**Suggested Layer**: E2E

### TC-401: Book exactly ten tickets
**Category**: Edge Case
**Priority**: P1
**Preconditions**: Event has at least ten available seats.
**Steps**: 1. Set quantity to 10. 2. Complete the booking.
**Expected Results**: The booking succeeds and total price and availability reflect ten tickets.
**Business Rule**: Quantity 10 is the upper valid boundary.
**Suggested Layer**: E2E

### TC-402: Book when only the requested number of seats remains
**Category**: Edge Case
**Priority**: P0
**Preconditions**: User-specific availability equals N, where N is between 1 and 10.
**Steps**: 1. Book exactly N tickets. 2. Reload the event.
**Expected Results**: The booking succeeds and the event is shown as sold out for that user.
**Business Rule**: A booking is accepted when requested quantity exactly equals availability.
**Suggested Layer**: API

### TC-403: Prevent booking one ticket beyond availability
**Category**: Edge Case
**Priority**: P0
**Preconditions**: User-specific availability is N.
**Steps**: 1. Request N+1 tickets, within the form’s maximum where possible. 2. Inspect the response.
**Expected Results**: The request is rejected for insufficient seats and existing bookings remain unchanged.
**Business Rule**: Seat availability is an enforced upper bound.
**Suggested Layer**: API

### TC-404: Handle an empty bookings list
**Category**: Edge Case
**Priority**: P1
**Preconditions**: Authenticated user has no bookings.
**Steps**: 1. Open My Bookings. 2. Inspect available actions.
**Expected Results**: “No bookings yet” and Browse Events are visible; booking-only actions are not shown.
**Business Rule**: Users may have zero bookings after cancellation or clear-all.
**Suggested Layer**: E2E

### TC-405: Clear an already empty bookings list
**Category**: Edge Case
**Priority**: P2
**Preconditions**: Authenticated user has no bookings.
**Steps**: 1. Open My Bookings. 2. Verify the clear action is hidden or has no destructive effect. 3. Refresh.
**Expected Results**: The empty state remains stable and no error or phantom booking appears.
**Business Rule**: Clear-all is safe when the user has no records.
**Suggested Layer**: E2E

### TC-406: Keep booking references unique across repeated bookings
**Category**: Edge Case
**Priority**: P1
**Preconditions**: Event has enough seats for multiple bookings.
**Steps**: 1. Create multiple bookings. 2. Collect all references.
**Expected Results**: Every booking reference is unique and conforms to the event-title prefix rule.
**Business Rule**: Reference generation retries collisions and guarantees uniqueness.
**Suggested Layer**: Unit

### TC-407: Handle a booking at the nine-record boundary
**Category**: Edge Case
**Priority**: P0
**Preconditions**: User has exactly eight bookings.
**Steps**: 1. Create the ninth booking. 2. List bookings. 3. Create the tenth booking.
**Expected Results**: The ninth booking is retained with nine total records; the tenth replaces only the oldest record and the total remains nine.
**Business Rule**: The limit applies at nine bookings, with FIFO pruning only on overflow.
**Suggested Layer**: API

### TC-408: Preserve customer data after reload
**Category**: Edge Case
**Priority**: P2
**Preconditions**: A booking was created with a name, normalized email, and formatted phone.
**Steps**: 1. Navigate away from the detail page. 2. Reload the bookings list and detail page.
**Expected Results**: Persisted customer name, email, phone, quantity, and total remain accurate.
**Business Rule**: Booking customer fields are stored with the booking.
**Suggested Layer**: E2E

### TC-500: Show loading state while bookings are fetched
**Category**: UI State
**Priority**: P2
**Preconditions**: Network response for the bookings request is delayed.
**Steps**: 1. Open My Bookings. 2. Observe the page before the response resolves. 3. Allow the request to complete.
**Expected Results**: A loading indicator is shown without an incorrect empty state, then replaced by booking cards or the true empty state.
**Business Rule**: The bookings UI must distinguish loading from no data.
**Suggested Layer**: Component

### TC-501: Show booking error state when loading fails
**Category**: UI State
**Priority**: P1
**Preconditions**: The bookings API returns an error.
**Steps**: 1. Open My Bookings. 2. Inspect the rendered state.
**Expected Results**: A clear error state or toast is shown, no stale or unauthorized booking data is displayed, and retry behavior is available where supported.
**Business Rule**: API failures must not be presented as a successful empty result.
**Suggested Layer**: Component

### TC-502: Show the booking-limit warning at the correct threshold
**Category**: UI State
**Priority**: P2
**Preconditions**: User booking count can be controlled.
**Steps**: 1. View My Bookings with a low count. 2. Add bookings until the documented close-to-limit threshold. 3. Reopen the page.
**Expected Results**: The sandbox warning is hidden at low counts and visible near the booking limit with the documented limit message.
**Business Rule**: Booking-limit banners are conditional and should not appear when counts are low.
**Suggested Layer**: Component

### TC-503: Disable or prevent booking while an event is sold out
**Category**: UI State
**Priority**: P0
**Preconditions**: The selected event has zero seats available for the user.
**Steps**: 1. Open the events list and event detail. 2. Inspect booking controls.
**Expected Results**: The UI indicates sold out and does not allow a successful booking submission.
**Business Rule**: Bookings cannot exceed available seats.
**Suggested Layer**: E2E

### TC-504: Update quantity controls and displayed total
**Category**: UI State
**Priority**: P1
**Preconditions**: Event detail booking form is open.
**Steps**: 1. Use the increment and decrement controls. 2. Observe ticket count, total price, and control boundaries.
**Expected Results**: Ticket count and total update together; decrement cannot go below 1 and increment cannot exceed 10 or available seats.
**Business Rule**: The UI enforces the valid quantity range and price calculation before submission.
**Suggested Layer**: Component

### TC-505: Show form validation errors without submitting
**Category**: UI State
**Priority**: P1
**Preconditions**: Event detail booking form is open.
**Steps**: 1. Leave required fields blank or enter invalid values. 2. Select Confirm Booking.
**Expected Results**: Field-level validation feedback is shown, the confirmation request is not treated as successful, and entered valid values remain available for correction.
**Business Rule**: Invalid customer data must be rejected before a confirmed booking is shown.
**Suggested Layer**: Component

### TC-506: Display cancellation confirmation dialog
**Category**: UI State
**Priority**: P1
**Preconditions**: User is viewing an owned booking detail page.
**Steps**: 1. Select Cancel Booking. 2. Inspect the dialog. 3. Close or decline it.
**Expected Results**: The dialog clearly asks for confirmation and provides a non-destructive cancel path; the booking remains present when cancellation is declined.
**Business Rule**: Cancellation is destructive and requires explicit confirmation.
**Suggested Layer**: E2E

### TC-507: Show refund spinner for the documented delay
**Category**: UI State
**Priority**: P2
**Preconditions**: User is viewing an owned booking detail page.
**Steps**: 1. Select Check Refund Eligibility. 2. Observe immediately and after the result delay.
**Expected Results**: The refund spinner is visible while the client-side check runs for approximately four seconds, then is replaced by exactly one eligibility result.
**Business Rule**: Refund eligibility includes a four-second client-side loading animation.
**Suggested Layer**: Component

### TC-508: Remove cancelled booking from the current list
**Category**: UI State
**Priority**: P1
**Preconditions**: User has multiple bookings and is viewing the bookings list.
**Steps**: 1. Open one booking and cancel it. 2. Return to or observe the bookings list.
**Expected Results**: The cancelled card disappears without removing the other cards; an empty state appears only when the last booking is cancelled.
**Business Rule**: Individual cancellation affects only the selected booking.
**Suggested Layer**: E2E

### TC-509: Keep the list stable during clear-all mutation
**Category**: UI State
**Priority**: P1
**Preconditions**: User has multiple bookings and the clear-all request is delayed.
**Steps**: 1. Select Clear All Bookings and confirm. 2. Observe the mutation state. 3. Allow the request to finish.
**Expected Results**: The action cannot create duplicate requests, then the list transitions to the empty state only after the deletion succeeds; an error is shown if it fails.
**Business Rule**: Clear-all is an atomic user-scoped deletion.
**Suggested Layer**: Component
