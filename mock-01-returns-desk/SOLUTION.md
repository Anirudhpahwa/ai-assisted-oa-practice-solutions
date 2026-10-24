# mock-01-returns-desk solution (Returns Desk, 6 tests, 9 defects)

| # | File, function | Wrong | Fix | Test |
|---|---|---|---|---|
| 1 | rma/models.py, ReturnRequest.reason | default "not_needed" instead of "other" | default="other" | test_create_return |
| 2 | rma/validators.py, _required_text | checks `not value` before strip, so "   " passes | check `not value.strip()` | test_create_return (layered behind #1) |
| 3 | rma/workflow.py, ALLOWED_TRANSITIONS | approved missing "cancelled" | add it | test_status_workflow |
| 4 | rma/refunds.py, restock_fee_cents | floor division, not half up | (subtotal*10 + 50)//100 | test_items_and_refund_total |
| 5 | rma/queries.py, filter_returns | `&` instead of `\|` between search fields | use `\|` | test_filters_and_search |
| 6 | rma/queries.py, filter_returns | cancelled always excluded, so status=cancelled returns nothing | exclude cancelled only when no status filter (layered behind #5) | test_filters_and_search |
| 7 | rma/validators.py, parse_list_params | reads "skip" instead of "offset" (lost value) | params.get("offset") | test_sorting_and_paging |
| 8 | rma/queries.py, ORDERINGS | refund_desc ascending | "-refund_total_cents" (layered behind #7) | test_sorting_and_paging |
| 9 | rma/reports.py, build_summary | Count instead of Sum | Sum("refund_total_cents") | test_summary |
