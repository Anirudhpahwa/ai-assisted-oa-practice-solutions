# mock-06-shift-roster-desk solution (Shift Roster Desk, C++17, 6 tests, 7 defects)

| # | File, function | Wrong | Fix | Test |
|---|---|---|---|---|
| 1 | src/text.cpp, trim | only strips leading whitespace, so trailing spaces and the `\r` of CRLF files stay on every field | `text.substr(first, last - first + 1)` using `find_last_not_of` | test_parse_inputs |
| 2 | src/engine.cpp, rejection_reason | `filled.insert(...)` runs as the SHIFT_TAKEN check, so a request that is rejected later (overlap, rest, weekly limit) still reserves the shift | take `filled` as const, check with `count`, and insert the shift only when the request is approved | test_basic_decisions |
| 3 | src/engine.cpp, rest_between | uses minutes-of-day only and ignores which day each shift is on, so shifts on different days get a negative gap and are rejected | use absolute times: `absolute_start(b) - absolute_end(a)` and the mirror | test_overlap_and_rest, test_weekly_limit, test_pay |
| 4 | src/timeutil.cpp, week_index | `(days + 4) / 7` makes weeks start on Sunday (day 0 is a Thursday); the README says Monday to Sunday (layered behind #3 in test_weekly_limit) | `(days + 3) / 7` | test_weekly_limit |
| 5 | src/engine.cpp, decide_all | passes the week total including the new shift as `week_minutes_before` to `shift_pay`, so the shift's own minutes are counted twice toward overtime (layered behind #3 in test_pay) | pass only the minutes before this shift: drop `+ shift->minutes()` | test_pay |
| 6 | src/pay.cpp, shift_pay | integer division truncates; the README rounds half up | add half the divisor before dividing: `(weighted * rate + kMinutesTimesPercent / 2) / kMinutesTimesPercent` | test_pay |
| 7 | src/report.cpp, summarize | the tie-break compares `staff_id` with `>`, so equal pay is ordered Z to A | `a.staff_id < b.staff_id` | test_summary_and_output |

Layers: #3 first hides #4 (test_weekly_limit) and #5 (test_pay); #6 shows first in test_pay because the direct pay checks run before the engine checks.
