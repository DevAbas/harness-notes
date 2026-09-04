# Measurement 12 — "Ticket status is a workflow, not a list."

Task: see m12-prompt.txt
Branch: measure/12-workflow-engine
Harness state: layers 1, 1b, 1c, 2 and 3 in place; layer 4 not built
Prompt style: detailed brief, measured
Baseline: harness-06
Codebase: ~7 screens, 321 modules, 493 tests before / 534 after
Files touched: 22 modified, 14 new, 1 deleted
Lint: 5 warnings, unchanged
Duration: 31 minutes API, $16.32

The module count is measured against harness-06, which excluded build output
from the boundary check: 308 modules at that tag, 321 here. Headers before m11
report a number that included `dist/` and is not comparable.

The second measurement run with a detailed brief, and the first to change
behaviour that already worked rather than add a surface beside it. Ticket
status went from a value any screen could set to a position with named moves
out of it.

Audited blind through the packaged skill after the commit and before I read
the diff.

## m12-1 Both guides claim a fifth status needs no screen change
AGENTS.md and README.md gained the same claim: adding a fifth status is an edit
to the workflow description, and no screen has to change.

TicketStatusBadge holds a Record keyed by the status union. A fifth status is a
compile error there — on a screen, in the design system, which is the one place
the claim is about.

Everywhere else it is true. The filters, the saved views, the reports and the
CSV export read TICKET_STATUSES for its members and would pick a fifth up
without an edit, which is the brief's constraint satisfied exactly as asked. The
claim is simply written wider than what was built.

The same paragraph miscounts its own tables and functions.
Caught by: nothing — the compiler does catch the fifth status, but only for
whoever tries the edit the guide calls safe
Layer: not harness — documentation, in the file every future agent reads first.
Same shape as m10-1, and the same file.

## m12-2 The moves card's empty state is only ever shown to someone who may not
The card that offers the moves out of a ticket has an empty state saying there
is nowhere to take the ticket.

Work out when the offers list is actually empty and it is one pair: an agent
looking at a closed ticket. Reopen is the only role-restricted move, so a role
is the only way for the list to come out empty — every other status offers
something to both roles.

So the sentence is only ever rendered in the one case where the truth is not
that there is nowhere to go, but that this person may not go there. The server
already answers with the right sentence — "Reopen is for administrators." The
card has it and shows the other one.

A test asserts the wrong sentence verbatim, which is what holds it in place.
Caught by: nothing — the branch is covered, and the coverage is what fixes the
wrong answer
Layer: guide missing — nothing says an empty state has to name the reason the
list is empty when a role is the reason

## m12-3 The bulk move report stays pinned above a table it no longer describes
BulkMoveReport is rendered unconditionally and reset nowhere. A report on one
selection survives filter changes, saved views, paging and an unrelated bulk
delete, sitting above a table whose rows it may no longer describe.

CustomersPage answered this already, and in writing: a docblock there explains
why the reset belongs on the way in rather than on the way out, because every
path that changes what the table shows is a path that would otherwise have to
remember to clear the band.

Worse than the case it diverged from. The customers band only persists on
failure. This one persists on success, which is the common path.
Caught by: nothing
Layer: rule missing — the answer exists, in a docblock on the screen it was
written for, in a file this change never opened

## m12-4 A refused move leaves the detail cache stale and nothing invalidates it
The error path does not invalidate the ticket detail query. The app sets
refetchOnWindowFocus: false, so there is no background refetch coming to correct
it — the ticket on screen keeps what it was showing when the move was refused,
including the offers that produced the refused move.

The person reads the sentence, clicks the same button, and gets the same
sentence.

The success path already does exactly the invalidation the error path needs.
Caught by: nothing
Layer: sensor missing

## m12-5 A docblock cites a hook this change deleted
useBulkUpdateCustomerPlan's docblock names useBulkUpdateStatus as the pattern it
follows. useBulkUpdateStatus is gone, deleted by this change, because a second
door to a status is precisely what the workflow closes.

The file is outside the diff, so no reviewer sees it. The next person to read it
goes looking for a hook that does not exist.
Caught by: nothing — a name in a comment is not a reference
Layer: rollout — the deletion was right and its citations were not followed.
Closable mechanically: a docblock naming a symbol that no longer resolves.

## What I checked myself
It found nothing the audit did not, and one thing that was wrong.

I worked from the browser and the console: refused moves, the agent and admin
views, the history card, reassignment. The refusal path is right — 409 with the
condition named, and the same sentence on both sides. Reopen asks for a reason
and forward moves do not, which is the rule the brief asked for.

I concluded there was no role difference at all. That came from comparing the
agent and admin views on an open and a pending ticket, and from a close that
returned 200. It was wrong. Reopen returns 403 with "Reopen is for
administrators." The gate exists, it sits on the one move I had not tried, and
the two statuses I compared are the two where no gate applies.

Three other things I noted do not follow from the brief and are recorded as
observations rather than findings:

- start does not take ownership and release does not give it up, though both
  are named for it.
- Making a ticket unowned still means typing "Unassigned" into the assignee
  field — m04-2 surfacing again, now that ownership gates a move.
- The history card covers status moves only, which matches what it says it
  covers.

## What was done well
From the audit's own section.

No second door to a status: the patch schema lost the field, the bulk status
route is gone, and a status in a patch is a 400 rather than being silently
stripped.

The role gate lives in the workflow rather than on the route, so the interface
and the server are reading the same description to decide what an agent may do.

`reads: 'ticket' | 'move'` on a guard is what makes two refusal surfaces
possible from one description: a ticket-condition can be evaluated up front and
shown disabled with its reason, a move-condition can only be asked for in a
dialog.

ticketStatusRank is derived from TICKET_STATUSES rather than written down again.

Promise.all returned from onSuccess, so the mutation stays pending through
invalidation rather than settling before the screen catches up.

Refusal sentences single-sourced between client and server — which is what makes
m12-2 a discarded sentence rather than two sentences that disagree.

## What this measurement shows
The intersection between the two audits was zero, for the first time.

m08 converged on four and m10 on four. Here nothing did, and the reason is where
each of us was looking. I was in the browser and the console; every finding was
in code.

That is not an argument against the browser pass. m05 and m09 produced findings
that only a person looking at a screen could produce, and no audit reading a
diff would have produced them. On this measurement it paid nothing, and the one
thing it produced was wrong.

Two of the five findings are prose contradicting the code beside it, which is
the census that has been running since m03. One of them is in AGENTS.md again —
the same file as m10-1, in a change whose brief made reading it mandatory.

And m12-3 is a shape the record has not had before. Not a gap: new work
diverging from existing work that had already answered the same question, in
writing, in a file the agent did not open.
