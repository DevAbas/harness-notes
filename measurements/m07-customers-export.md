# Measurement 7 — "Add an export to CSV button above the customers list."

Task: see m07-prompt.txt
Branch: remeasure/07-customers-export
Harness state: layers 1, 2 and 3 in place
Prompt style: one line, nothing specified
Baseline: harness-03
Codebase: ~6 screens, ~120 source files, 312 tests
Files touched: 3 modified, 3 new
Tests: 324, up from 312
Lint: 19 warnings, byte-identical to the pre-change baseline

The first measurement taken with a harness in place. The task deliberately
mirrors m01 — the same feature on a different screen — so the comparison is
against a recorded run rather than a memory.

## m07-1 The runaway guard truncates the way it was written to prevent
fetchEveryCustomer stops at MAX_EXPORT_PAGES and returns what it has. The
docstring argues for the cursor walk on the grounds that asking for one
oversized page "would mean an export that quietly stopped at the hundredth
customer — and stopped silently, because a truncated CSV looks exactly like a
complete one." The guard does the same thing at page fifty. Unreachable against
a sixty-customer seed, and a throw would be consistent with the argument.
Caught by: nothing
Layer: not harness — minor

## m07-2 Alert is imported and then switched off

    <Alert tone="danger" className="border-0 bg-transparent p-0">

The comment explains it: what is being reused is the tone and the role, not the
chrome. That is a fair argument, and it means Alert is used here for two
attributes with everything else overridden by className.

Same shape as the selected-state finding one level up: the primitive exists, the
form needed does not, so className fills the gap. The next inline error will
switch it off with a different set of classes.
Caught by: nothing
Layer: guide missing (no inline variant on Alert)

## m07-3 An existing lint warning in a touched file was left alone
CustomersToolbar.tsx carried a no-raw-type-classes warning before this change
and carries it after, on a moved line. The agent edited the file, the rule
fired, and nothing happened — because the rule ships at warn.

Correct behaviour for the task given. Recorded because it is the first direct
evidence of what the warn tier costs: a rule that fires and is ignored is a rule
that only reports.
Caught by: the rule fired and was ignored
Layer: rollout decision

## m07-4 A third name for one constant
CUSTOMERS_CSV_MIME_TYPE is CSV_MIME_TYPE re-exported. TICKETS_CSV_MIME_TYPE is
the same. The wrapper made sense in m01, when there was no shared lib/csv.ts;
there is one now and the wrapper was copied anyway.
Caught by: nothing
Layer: not harness — minor

## What was done well
The shared writer was found and used. lib/csv.ts was not re-implemented, so RFC
4180 quoting and the spreadsheet-formula guard are written once.

The ticket pattern was read and rejected with a reason. useTicketsExport asks
for a single page sized to total, which works because MAX_PAGE_SIZE is 500
against a forty-ticket queue. The customer endpoint caps a page at 100, so the
agent wrote a cursor walk and wrote down why copying would have been wrong.

That is the direct inverse of m06, where the agent explained why it was not
copying the ticket screen and then copied it in every other respect.

The walk ends on nextCursor === null rather than on a short page, with a comment
naming the reason — a filter can produce a short page at any point. That is a
subtle bug most implementations ship.

It went to listCustomers rather than through fetchQuery, because
customerKeys.list deliberately holds no cursor and caching the walk would mean a
second, contradictory key scheme.

Placement was argued from the existing docblock: CustomersToolbar keeps its
filters on one line because a dense list should not lose rows to chrome, so a
second bordered row for one button would cost two customers on screen.

The button is secondary, matching AGENTS.md and the ticket export. Errors go
through Alert. No hand-padded div at the card scale.

Zero new lint violations, verified by the agent itself with git stash.

## What this measurement shows
Against m01, which produced four findings on a repo with no harness, this
produced four — but of a different kind. m01's were a missing BOM, two
hand-built error banners, a skipped cn(), and a stale error state. Three of
those four are now impossible: Alert exists and is used, cn() is enforced, and
the CSV writer is shared.

What is left is smaller and further out: a guard that contradicts its own
argument, a primitive used for two of its attributes, a warning ignored because
it is a warning, and a wrapper copied out of habit.

The prediction recorded in ../harness/harness-03-agents-md.md was that the Conventions half
would hold and the Product decisions half would get guessed. On this measurement
both held. The repo is still six screens; the prediction is about fifteen.
