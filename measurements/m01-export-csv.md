# Measurement 1 — "Add an export to CSV button above the ticket list."

Task: Add an export to CSV button above the ticket list.
Branch: measure/export-csv
Harness state: none (README only)
Baseline: baseline-small
Codebase: ~4 screens, ~25 source files, 43 tests

6 files.

## m01-1 No BOM in the CSV
Excel on Windows breaks non-ASCII text without a BOM. All test data is plain
English ("Dana Cole"), so the tests never hit the problem.
Caught by: nothing
Layer: sensor blind spot — test data is not like real data

## m01-2 Two error banners, built by hand
TicketsToolbar has role="status". TicketListPage does not, and its error matters
more. There is no Alert component, so the same thing was built twice.
Caught by: nothing
Layer: guide missing (no Alert) + sensor missing (no a11y check)

## m01-3 cn() skipped
TicketsToolbar writes class names as plain strings. The README says to use cn()
in every new component. In measurement 2 the same agent used it correctly.
Caught by: nothing
Layer: README rule broken

## m01-4 exportError is never cleared
The old error stays on screen after the filters change.
Caught by: nothing
Layer: not harness — normal state bug
