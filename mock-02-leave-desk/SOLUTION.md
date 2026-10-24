# mock-02-leave-desk solution (Leave Desk, 6 tests, 9 defects)

| # | File, function | Wrong | Fix | Test |
|---|---|---|---|---|
| 1 | timeoff/dates.py, count_weekdays | loop stops before `end`, so the last day is not counted | `while day <= end` | test_create_request |
| 2 | timeoff/conflicts.py, find_conflict | no status filter, so cancelled and rejected requests block new ones (layered behind #1) | add `status__in=ACTIVE_STATUSES` | test_create_request |
| 3 | timeoff/decisions.py, decide | self-decision check compares the actor to `employee.name` instead of `employee.username` | compare with `username` | test_decision_and_balance |
| 4 | timeoff/decisions.py, adjust_balance | balance changed in memory but never saved (layered behind #3; also hides in cancel) | `employee.save(update_fields=["balance_days"])` | test_decision_and_balance, test_cancel_request |
| 5 | timeoff/decisions.py, cancel | balance restored for every annual request, including pending ones (layered behind #4) | restore only when `leave.status == "approved"` | test_cancel_request |
| 6 | timeoff/queries.py, filter_requests | department filter starts from a fresh queryset, dropping the earlier filters (layered behind #7) | `leaves = leaves.filter(employee__department=...)` | test_list_filters_and_paging |
| 7 | timeoff/validators.py, parse_list_params | reads `page_size`, the API and frontend send `per_page` (lost value) | `params.get("per_page", ...)` | test_list_filters_and_paging |
| 8 | timeoff/queries.py, department_summary | `Count("employee")` without `distinct=True` counts requests, not people | add `distinct=True` | test_summary_by_department |
| 9 | timeoff/queries.py, who_is_out | uses ACTIVE_STATUSES (pending and approved) instead of approved only | `status="approved"` | test_who_is_out |
