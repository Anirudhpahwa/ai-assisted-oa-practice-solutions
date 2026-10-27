# mock-05-equipment-loan-desk solution (Equipment Loan Desk, 6 tests, 9 defects)

| # | File, function | Wrong | Fix | Test |
|---|---|---|---|---|
| 1 | loandesk/schedule.py, loan_days | uses `max()` of the category periods; the README says the shortest period applies | `min(...)` | test_create_loan |
| 2 | loandesk/loans.py, create_loan | the loan row is created before the availability check, and the check runs while lines are being saved, so a 409 leaves a loan and some lines behind | check every entry first (or wrap the work in `transaction.atomic()`) | test_create_loan |
| 3 | loandesk/validators.py, _items | `isinstance(quantity, int)` accepts `True`, because `bool` is a subclass of `int` | use `_is_int(quantity)` | test_create_loan |
| 4 | loandesk/loans.py, available_units | counts every line except those on returned loans, so cancelled loans still hold units | `filter(loan__status="active")` | test_extend_and_cancel |
| 5 | loandesk/loans.py, return_loan | passes `loan.checked_out_on` to `fees.late_fee` where the due date is expected (wrong argument) | pass `loan.due_on` | test_return_loan |
| 6 | loandesk/fees.py, chargeable_days | returns the full days late once past the grace period, instead of subtracting the grace day (layered behind #5) | `return late - GRACE_DAYS` | test_return_loan |
| 7 | loandesk/queries.py, filter_loans | the `equipment` filter restarts from `Loan.objects` and drops the earlier filters | `loans = loans.filter(lines__equipment_id=...)` | test_list_filters |
| 8 | loandesk/queries.py, list_page | `pages = total // per_page` rounds down | ceiling division: `(total + per_page - 1) // per_page` | test_sorting_and_paging |
| 9 | loandesk/reports.py, borrower_summary | `late_returns` counts loans returned after `due_on`; the README counts loans with a fee above 0, so loans inside the grace day are counted | `loan.fee_cents > 0` | test_borrower_summary |

Layers: #5 then #6 (return); #1, #2 and #3 are independent but all sit in test_create_loan.
