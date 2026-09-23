# Booking Management Test Strategy

## Scope

This strategy covers booking creation, retrieval, filtering, pagination, refund eligibility, cancellation, clear-all behavior, seat accounting, booking limits, validation, authentication, cross-user isolation, and booking-page UI states.

Source of truth: [test-scenarios.md](test-scenarios.md), scenarios TC-001 through TC-509.

## Quality Goals

- Protect P0 booking journeys: create, view, cancel, clear, seat enforcement, booking-limit enforcement, and user isolation.
- Validate booking calculations and authorization close to the backend service boundary.
- Keep UI tests focused on user-visible workflows and asynchronous states rather than implementation details.
- Ensure tests are self-contained, use dynamic data, and do not depend on execution order.

## Test Pyramid

| Layer | Purpose | Target coverage | Test location |
|---|---|---:|---|
| Unit | Deterministic business logic and boundary algorithms | 3 scenarios | `backend/test/unit/bookingService.spec.js` |
| API | Routes, validation, authorization, persistence contracts, and booking calculations | 31 scenarios | `backend/test/api/bookings.spec.js` |
| Component | Refund logic, form rules, loading/error states, and mutation states | 9 scenarios | `frontend/components/**/__tests__/booking*.test.*` |
| E2E | Critical journeys across frontend, backend, and database | 16 scenarios | `tests/booking-management.spec.js` |

The counts above are scenario assignments, not a requirement to create one automated test per scenario. Related boundary examples may be parameterized within one test while preserving individual assertions and traceability.

## Unit Layer

Use isolated service or helper tests with repository and Prisma dependencies mocked. These tests should be fast and deterministic.

| Scenarios | Coverage |
|---|---|
| TC-101 | Booking reference prefix, six-character suffix, uppercase format, and uniqueness retry behavior |
| TC-105 | FIFO pruning preference for an oldest booking from a different event |
| TC-406 | Repeated reference generation remains unique |

**Unit checks**

- Mock `findByRef` to exercise both collision-free and collision-retry paths.
- Verify that the generated fallback reference still uses the event-title prefix.
- Verify no repository deletion occurs when the current booking count is below nine.
- Verify the target event is excluded from FIFO pruning before same-event fallback is considered.

## API Layer

Use an isolated test database or transaction rollback per test. Obtain event and booking IDs from setup responses; never hardcode database IDs. Authenticate through the real login endpoint or a documented token fixture.

| Scenarios | Coverage |
|---|---|
| TC-005 to TC-008 | Reference lookup, pagination metadata/order, event filtering, and status filtering |
| TC-100, TC-103, TC-104, TC-106, TC-109, TC-110 | Total-price calculation, per-user dynamic seats, nine-booking FIFO, cancellation seat effects, confirmed status, and static-event behavior |
| TC-200 to TC-205 | Missing/malformed authentication, list isolation, cross-user detail/reference access, cross-user cancellation, clear-all isolation, and dynamic-event isolation |
| TC-300 to TC-310, TC-312 | Required fields, name/email/phone rules, quantity and event-ID boundaries, missing events, insufficient seats, unknown bookings/references, and JWT failures |
| TC-402, TC-403, TC-407 | Exact-availability success, one-seat-over-availability rejection, and exact nine-booking boundary behavior |

**API assertions**

- Assert status, response error text/details, and database state after every mutation.
- Assert that rejected booking requests do not consume seats or create partial records.
- Assert that User B cannot infer User A booking data from either ID or reference lookup.
- Assert clear-all deletes only the authenticated user’s rows.
- Parameterize invalid field values, but keep the expected field and validation message explicit.

## Component Layer

Mock API responses and use controlled timers only for the refund delay. Component tests should assert accessible user-facing states and controls, not CSS classes or React Query internals.

| Scenarios | Coverage |
|---|---|
| TC-107, TC-108, TC-507 | Refund spinner, approximately four-second delay, eligible single-ticket result, and non-refundable group result |
| TC-500 to TC-502 | Loading state, failed fetch state, empty-state separation, and booking-limit warning threshold |
| TC-504, TC-505 | Quantity control boundaries, displayed total, and client-side form validation |
| TC-509 | Clear-all pending, success-to-empty transition, duplicate-action prevention, and failed mutation state |

**Component assertions**

- Use role, label, placeholder, and `data-testid` locators in that order of stability.
- For the timed refund state, assert spinner visibility first and then wait on result visibility; do not use arbitrary sleeps.
- Mock exact booking counts around the warning threshold so hidden and visible states are both covered.
- Verify that a failed mutation does not optimistically erase booking cards unless the product explicitly specifies optimistic behavior.

## E2E Layer

Run Chromium-only Playwright tests against the configured frontend and backend. Each test logs in, creates or discovers its own data, asserts the outcome, and cleans up where possible. Use the existing test account for ordinary flows and the second account for cross-user setup.

| Scenarios | Test file / coverage |
|---|---|
| TC-001 to TC-004 | `tests/booking-management.spec.js`: book an event, verify list/detail content, and follow confirmation navigation |
| TC-009, TC-010 | `tests/booking-management.spec.js`: confirm individual cancellation and clear-all empty state |
| TC-102 | `tests/booking-management.spec.js`: verify visible availability changes after booking |
| TC-311 | `tests/booking-management.spec.js`: decline cancellation and verify the booking remains |
| TC-400, TC-401 | `tests/booking-management.spec.js`: quantity lower and upper boundary through the real form |
| TC-404, TC-405 | `tests/booking-management.spec.js`: empty state and safe empty clear behavior |
| TC-408 | `tests/booking-management.spec.js`: persisted customer details after navigation/reload |
| TC-503, TC-506, TC-508 | `tests/booking-management.spec.js`: sold-out controls, cancellation dialog, and list update after cancellation |

**Playwright rules**

- Prefer `getByTestId`, semantic roles, labels, and placeholders; use CSS classes only where no stable hook exists.
- Avoid `page.waitForTimeout()`. Use Playwright auto-waiting and explicit visibility/state assertions.
- Use filtered booking/event cards instead of positional selectors.
- Capture generated booking references and event titles at runtime.
- Keep each test independent; clear or cancel test-created bookings in setup/teardown.

## Priority Execution Order

### P0 smoke gate

Run on every pull request:

`TC-001, TC-002, TC-009, TC-010, TC-101, TC-102, TC-104, TC-200, TC-201, TC-202, TC-203, TC-204, TC-308, TC-312, TC-402, TC-403, TC-503`

These cover the primary booking lifecycle, seat correctness, booking limit, authentication, user isolation, and the highest-risk rejection paths.

### P1 regression suite

Run on pull requests touching booking, event, auth, API client, or shared UI code:

`TC-003 to TC-008, TC-103, TC-105 to TC-109, TC-205, TC-300 to TC-307, TC-309, TC-311, TC-400, TC-401, TC-404, TC-407, TC-506, TC-508, TC-509`

### P2 extended suite

Run nightly or before release:

`TC-005, TC-007, TC-008, TC-110, TC-310, TC-405, TC-406, TC-408, TC-500 to TC-502, TC-504, TC-505, TC-507`

## Test Data and Isolation

- Primary account: `rahulshetty1@gmail.com` / `Magiclife1!`.
- Secondary account: `rahulshetty1@yahoo.com` / `Magiclife1!`.
- Use seeded static events for stable smoke coverage and dynamically created user events for seat and sandbox isolation coverage.
- Use unique customer names and event data where mutations could persist between runs.
- Prefer API setup for reaching nine-booking, exact-seat, and pagination states; reserve UI setup for journeys whose UI behavior is under test.
- Do not assume static event IDs or booking IDs. Capture them from API responses or page content.

## Exit Criteria

- All P0 tests pass with no retries hiding failures.
- API validation and authorization scenarios pass with database-state assertions.
- No open P0/P1 defect remains for booking creation, seat accounting, cancellation, clear-all, or cross-user access.
- Component tests cover both success and failure states for refund and booking mutations.
- Playwright tests run without arbitrary waits, hardcoded booking IDs, or shared order-dependent state.
