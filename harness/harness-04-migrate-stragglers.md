    Branch: harness/04-migrate-stragglers
    Baseline: baseline-auth
    Tests: 407, up from 406
    Lint: 15 warnings, down from 19

Not a measurement. Two migrations bringing existing code onto patterns the repo
had already established elsewhere.

### The ticket bulk bar

Applying a bulk status now drops only the ids the request acted on, matching
what the customers bar does and for the reason its comment gives — anything
ticked while the request was in flight was not part of what was just done.

The Select returns to its default after an apply. Worth recording why this
needed real code rather than falling out of the first fix: on the customers side
the reset is unmount-driven, since the bar only renders while something is
selected, so emptying the selection destroys the plan state. The ticket bar was
the same, which is why its Select never actually stuck. The selection fix is
what creates the case where it would — a row ticked mid-flight keeps the bar
mounted through a successful apply, and without an explicit reset the bar sits
showing "Closed" over rows that are already closed.

So fixing the first defect created the second, and the agent found that rather
than shipping the first fix alone.

The default itself was left. A plan change is a billing change nobody asked for
and no plan is the obvious landing spot; a status is reversible from the same
bar and bulk-editing a queue is nearly always working it down to resolved.
'resolved' moved into a named DEFAULT_STATUS with that argument in its docblock.

Two things deliberately not changed, both flagged: confirmBulkDelete still
clears the whole selection, one line down from the fix; and the customers bar
now has the same sticky-Select in the mid-flight case, so it is the asymmetric
one.

### The deprecated form type

All four files migrated to SubmitEventHandler. No FormEvent left.

The rule still cannot move to error. It was six violations, not four — the
config comment was stale. queryClient.fetchQuery in useTicketsExport is a
one-line swap to queryClient.query. recharts' Cell in BarChart is a real rewrite
of that chart's per-bar colouring. Promotion is one line in eslint.config.js
once both land.

### The finding worth keeping

TicketListPage is now 183 lines against the 185 threshold, up from 181. It is
the function that set that number.

The agent wrote applyBulkStatus compactly to stay under it, and said so: a named
forgetSelected helper, which is what the customers page has, would have pushed
it to 186.

So the threshold changed the shape of the code, and made it worse. The same
operation is now written two ways in two files — one with a named helper, one
inline — because a counting rule was two lines away.

complexity is unchanged at 20 against a max of 20. Both counting rules are
errors in strict, so the next change to this file has to extract from it or
raise a threshold. The config calls raising it a fair answer, which it is, but
it is a decision rather than a default.
