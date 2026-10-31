# mock-07-event-signup-desk solution (Event Sign-up Desk, 6 tests, 9 defects)

| # | File, function | Wrong | Fix | Test |
|---|---|---|---|---|
| 1 | src/validators/eventValidator.js, DEFAULT_CAPACITY | the default capacity is 25; the README says 20 | `const DEFAULT_CAPACITY = 20;` | create event |
| 2 | src/services/queries.js, buildEventFilter | when `status` is given the filter is rebuilt as `{ status }`, which drops the `organizer` condition, so other people's events with that status appear | `filter = { ...filter, status: params.status };` | list events |
| 3 | src/validators/listParams.js, parseListParams | `pageSize` is read from `query.limit`; the frontend and README use `pageSize`, so the value is ignored and the default 10 is used | read `query.pageSize` | list events |
| 4 | src/validators/eventValidator.js, normalizeUpdate | `location` is always set (to `""` when absent), so a partial update wipes it | only set `changes.location` when `input.location !== undefined` | update event |
| 5 | src/controllers/eventController.js, update | the "capacity was raised" test runs after `event.set(changes)`, so it compares the new capacity with itself and promotion never runs | compute `grew` before `event.set(changes)` (layered: after this, #6 shows up) | update event |
| 6 | src/services/capacity.js, promoteWaitlist | `registration.seats >= left` stops the loop when a waitlisted registration fits exactly | `registration.seats > left` | update event, cancel registration |
| 7 | src/controllers/registrationController.js, create | the duplicate check has no status condition, so a cancelled registration blocks signing up again | add `status: { $in: ["confirmed", "waitlisted"] }` | register for event |
| 8 | src/controllers/registrationController.js, cancel | the registration is looked up with `findById(regId)` and never checked against the event in the URL, so another event's registration can be cancelled through this event | `findOne({ _id: regId, event: event._id })` | cancel registration |
| 9 | src/controllers/summaryController.js, show | `fillRate` divides `confirmed.count` (registrations) by capacity; the README uses confirmed seats | use `confirmed.seats` | event summary |

Layers: #5 then #6 (update event). #6 also breaks cancel registration on its own. #1 to #4 and #7 to #9 are independent.
