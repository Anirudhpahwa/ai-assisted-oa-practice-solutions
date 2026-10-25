# mock-03-stock-count-desk solution (Stock Count Desk, 6 tests, 9 defects)

| # | File, function | Wrong | Fix | Test |
|---|---|---|---|---|
| 1 | stocktake/validators.py, clean_new_session | `note` is validated but left out of the returned dict, so it is never saved (lost value) | add `"note": note` to the returned dict | test_create_session |
| 2 | stocktake/conflicts.py, find_conflict | only `open` sessions conflict; a `submitted` session at the location should too (layered behind #1) | `status__in=LIVE_STATUSES` | test_create_session |
| 3 | stocktake/validators.py, parse_count | no check that `counted` is 0 or more | raise ValidationFailed when `counted < 0` | test_record_counts |
| 4 | stocktake/counts.py, record_count | a recount updates `counted` but never recomputes `variance` (layered behind #5) | `line.variance = counted - line.expected` | test_record_counts |
| 5 | stocktake/workflow.py, is_assigned | compares the actor to `created_by` instead of `assigned_to`; breaks recording, submit and the approve guard | compare with `session.assigned_to` | test_record_counts, test_submit_approve_and_void |
| 6 | stocktake/approval.py, approve | adds the variance to `on_hand` instead of replacing `on_hand` with `counted` (layered behind #5) | `product.on_hand = line.counted` | test_submit_approve_and_void |
| 7 | stocktake/queries.py, filter_sessions | `scheduled_to` uses `__lt`, so the end date is excluded | `scheduled_for__lte` | test_list_filters |
| 8 | stocktake/queries.py, SORTS | `sort=location` orders by the raw column, so it is case sensitive | order by `Lower("location")` (import `Lower`) | test_sorting_and_paging |
| 9 | stocktake/reports.py, location_summary | shortage value sums `variance * unit_cost`, which is negative; it should be the shortfall (`-variance`) times the cost | `Sum(-F("variance") * F("product__unit_cost_cents"), ...)` | test_location_summary |

Layers: #1 then #2 (create), #5 then #4 then #3 (record counts: with #5 fixed the recount assertion fails first, then the negative-count check), #5 then #6 (approve).
