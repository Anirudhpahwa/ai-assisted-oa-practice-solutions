# mock-04-help-desk solution (Help Desk, 6 tests, 9 defects)

| # | File, function | Wrong | Fix | Test |
|---|---|---|---|---|
| 1 | helpdesk/sla.py, SLA_HOURS | `high` is 12 hours, the README says 8 | `"high": 8` | test_create_ticket, test_update_ticket |
| 2 | helpdesk/views.py, ticket_collection | reads the header `X-Idempotency-Key`; clients send `Idempotency-Key`, so the key is never found (lost value; layered behind #1) | `request.headers.get("Idempotency-Key")` | test_create_ticket |
| 3 | helpdesk/validators.py, clean_ticket_update | `description` is always added to the changes with a default of `""`, so any PATCH without it wipes the description | add it only when `"description" in data` | test_update_ticket |
| 4 | helpdesk/tickets.py, update_ticket | a history event is recorded whenever `priority` or `assignee` is in the body, even if the value is unchanged (layered behind #3) | also require `changes[kind] != getattr(ticket, kind)` | test_update_ticket |
| 5 | helpdesk/tickets.py, change_status | `resolved_at` is never cleared when a resolved ticket is reopened | `elif target == "open": ticket.resolved_at = None` | test_status_and_comments |
| 6 | helpdesk/serializers.py, ticket_detail_to_dict | history ordered newest first (`-created_at, -id`); README says oldest first | `order_by("created_at", "id")` | test_status_and_comments |
| 7 | helpdesk/queries.py, filter_tickets | `assignee=none` filters on `assignee__isnull=True`, but unassigned tickets store `""` | `filter(assignee="")` | test_list_filters |
| 8 | helpdesk/dates.py, day_bounds | the exclusive upper bound is the start of the last day, so the last day is left out (helper used by the list and the report) | `day_start(last_day + timedelta(days=1))` | test_dates_sorting_and_paging, test_sla_report |
| 9 | helpdesk/reports.py, sla_report | `round()` rounds half to even, so 90.5 becomes 90 (layered behind #8) | `math.floor(mean + 0.5)` | test_sla_report |

Layers: #1 then #2 (create); #3 then #4, with #1 still showing in the due_at assertion (update); #8 then #9 (report). Tests 5 and 6 are independent of each other but both sit in test_status_and_comments.
