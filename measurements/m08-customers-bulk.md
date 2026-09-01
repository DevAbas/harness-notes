# Measurement 8 — "Add bulk actions to the customers list."

Task: see m08-prompt.txt
Branch: measure/08-customers-bulk
Harness state: layers 1, 2 and 3 in place
Prompt style: one line, plus one clarifying answer
Baseline: harness-03
Codebase: ~6 screens, 166 source files, 335 tests
Files touched: 23 modified, 3 new
Lint: 19 warnings, unchanged from the pre-change baseline

The agent stopped and asked which bulk actions the list should offer, noting
that the selection mechanism was the same either way. I answered "change plan
and delete, both admin-only". It did not ask about the selection mechanism, the
placement, or the API shape.

This is the second measurement with a harness in place, and the first that
required the agent to extend the design system rather than only consume it.

## m08-1 Alert is switched off twice, two different ways, in one feature
CustomersToolbar renders an inline error with

    <Alert tone="danger" className="border-0 bg-transparent p-0">

CustomersPage renders a band with

    <Alert tone="danger" className="items-center rounded-none border-x-0 border-t-0">

Both are reasonable, both are commented, and both are the primitive imported
for its tone and role with its appearance overridden. Second and third
instances of the shape first recorded as m07-2.

What is missing is a variant. An inline error in a toolbar and a band across a
card are two shapes Alert should know, and while it knows neither, className is
the only way to ask for them — which means the next one is switched off a third
way.

Correction, from the second audit below: the `border-0 bg-transparent p-0`
override is in TicketsToolbar.tsx, not CustomersToolbar.tsx — which contains no
Alert at all. So this is two features rather than one, and one of the two is
pre-existing ticket code this branch never touched. The diagnosis stands; the
heading and the attribution above it do not. The full census is three overrides
in two shapes, recorded under "The inventories were incomplete in opposite
directions".
Caught by: nothing
Layer: guide missing (no inline or band variant on Alert)

## m08-2 There is no Checkbox primitive, and the agent said so
ListRow now renders a raw <input type="checkbox"> with a hand-written
`size-4 accent-primary`. The docblock states it plainly: "It is a raw input
rather than a primitive because there is no checkbox primitive yet; when there
is, this is one of the three places to change."

Three instances now: the ticket table, MultiSelect's option list, and this. That
is the count at which Icon, Heading and StateMessage were written.

Recorded as a finding, but note what is different about it: the gap was named
by the agent, in the file, with the migration sites counted. In m01 through m06
a gap like this was filled silently.
Caught by: nothing
Layer: guide missing (no Checkbox primitive)

## m08-3 AGENTS.md is written at screen scale; a strip is not a screen
CustomersBulkActionsBar carries `bg-primary-subtle px-4 py-3` by hand, and gives
Apply no variant, so it renders primary beside a danger Delete in the same
strip.

Every one of those follows the letter of AGENTS.md and none of them is answered
by it. "primary is the one action a screen exists for" is a rule about screens;
this is a panel with two actions in it. "CardBody owns card padding" is a rule
about cards; this is a strip. The ticket BulkActionsBar carries the same class
string, so the two now agree by copying rather than by rule.
Caught by: nothing — no primitive owns this shape and no rule describes it
Layer: guide missing (no rule at panel scale)

## m08-4 Two screens, two select-all mechanisms, no rule either way
The ticket table puts select-all in a header checkbox. The customers list puts
it on the actions bar as "Select all 20", with a "Clear selection" beside it,
and hides it once everything loaded is ticked.

Both are argued, and the argument is sound: a table has a header row and a
bounded page; a list has neither, and "all" grows every time load-more is
pressed.

The finding is not that either is wrong. It is that a user meets two different
interfaces for the same operation on two screens of one product, and nothing in
the repo says which shape belongs where. AGENTS.md answers Table versus List;
it does not answer what selection looks like inside either.
Caught by: nothing
Layer: guide missing

## m08-5 The new screen fixes two defects the old one still has
On the ticket list, applying a bulk status clears the entire selection, and the
status Select stays on the value just applied rather than resetting.

The customers bar does neither. onSuccess drops only the ids the request acted
on — "anything ticked in the meantime was not part of what was just done" — and
the plan Select returns to its placeholder.

So the agent read the ticket bar, took its shape, and left its behaviour behind.
That is the inverse of m06, where the pattern was copied defects and all.

The result is still an inconsistency: the same operation now behaves two ways in
one product, and the correct one is the new one. Nothing migrates the old
screen.
Caught by: nothing
Layer: not harness — two pre-existing defects, plus a divergence nobody will
notice until they use both screens

## What was done well
The checkbox problem was diagnosed rather than worked around. A ListRow with
onSelect is one wide Button, so a checkbox in `leading` would be nested inside a
button — the click never lands and the markup is invalid. Rather than build a
parallel row in the feature layer, the primitive was extended: selection is a
grouped prop rendered as a sibling of the row control, with rowClasses moved
from w-full to flex-1 so a full-width child cannot push the box off the end.
Both changes carry the reason.

CustomerRow was extracted from the .map() with the reason written down — "which
is where it was heading". That is m02-4 not happening.

ErrorBand was extracted inside CustomersPage, with the reason: "two copies of
the same six classes in one file is where a drift starts". The agent applied the
experiment's own central finding to its own output, before the second instance
existed.

A stale-closure defect was found and fixed unprompted: Dialog closes on Escape
even while busy, so a confirmation can be dismissed mid-request, a drawer
opened, and the delete land underneath it. setOpenCustomerId reads through the
state updater rather than off the closure, and there is a test that drives that
exact sequence with a held response.

MAX_CUSTOMER_BULK_IDS is 500 rather than MAX_CUSTOMER_PAGE_SIZE, with the
reason: a ticket selection is bounded by the page showing it, while this
selection accumulates pages and can outgrow any single response.

The plan Select opens on a placeholder rather than a default, because a default
here is one click from moving forty accounts onto Free — where the ticket bar
can open on `resolved` because that is what bulk-editing a queue is nearly
always for.

planOptions is one list serving both the filter and the bulk bar, because "a
second copy would be the one that is out of date the day a tier is added".

bulkDelete uses removeQueries rather than invalidateQueries for the detail keys
— there is nothing at the other end of those keys, and invalidating would ask
the server to confirm it with sixty 404s. Ticket keys are deliberately
untouched, because deleting a customer takes the link and not the history.

canManageCustomers was added beside canManageTickets rather than reused, on the
grounds that a customer screen gating on canManageTickets is a line that reads
wrong.

Zero new lint violations, and the agent verified that itself by stashing.

## What this measurement shows
Every finding is a missing primitive or a missing rule. None is a broken rule,
and none is a defect in what was built.

The three that matter — no Alert variant, no Checkbox, no rule at panel scale —
are all the same shape as the findings layer 1 was built to close, one level
further out. The design system now covers what a screen is made of; it does not
yet cover what a strip above a list is made of.

The prediction recorded in ../harness/harness-03-agents-md.md was that the Conventions half
would hold and the Product decisions half would get guessed. Both held again,
but m08-3 shows the boundary: the product decisions are written at screen scale,
and this task built something that is not a screen. The rule did not get broken;
it did not reach.

## A second audit, blind, run later

Run after the branch had already been merged, using the packaged blind-audit
skill. The auditor was a fresh general-purpose agent, given only the branch, the
base, and the task prompt including the clarifying exchange.

I deliberately withheld one thing: my note that the agent never asked about the
selection mechanism, the placement, or the API shape. That was the hypothesis
under test.

It found eight. Four map onto findings recorded above; five are new; one of
mine is new on my side and half of it was disputed.

### Where the two audits met

    m08-1  Alert switched off with className    →  their 8, same diagnosis
    m08-2  no Checkbox primitive                →  their 5, and the count is wrong
    m08-4  two select-all mechanisms            →  their 3, plus the defect inside it
    m08-5  the new screen fixes two defects     →  in their "what was done well"

Four independent convergences, none of them a coincidence of phrasing. That is
stronger evidence than either pass alone, which is the whole reason step 4 of
the skill exists.

In every one of the four, theirs went further. The Checkbox census is four sites
and not three. The select-all divergence contains a focus defect. The Alert
variant has a third copy neither of my notes accounted for.

### What only the blind audit found

**A failed bulk delete reports nothing inside the dialog that is covering it.**
confirmDelete closes the ConfirmDialog on success only, and the sole error
surface is a band inside the Card — outside the dialog, which carries
aria-modal="true". Verified with a probe forcing the DELETE to fail: dialog
still open, message present in the document and not inside the dialog, confirm
button back to enabled. A screen-reader user gets nothing, because aria-modal is
a declaration that everything outside the dialog is inert, and that is exactly
where the message is. The obvious next click re-fires the delete.

The asymmetry is inside the change: the plan-change path has no modal, its
failure lands in the same band and is readable — and that is the path the agent
tested. The delete path has a modal in front of the band and no failure test at
all.

**Select-all replaces the selection rather than unioning it.** The change states
its selection model three times — derived, so a row the filters no longer match
drops out and comes back still ticked. toggleSelected and forgetSelected honour
it; onSelectAll does not. `setSelection(rows.map(...))` destroys every
remembered-but-hidden tick, and the damage is invisible when it happens because
the hidden ids were not being counted anyway.

The two behaviours are tested apart and the bug lives in their intersection:
select-all is exercised with no filter, where replace and union are
indistinguishable; the filter is exercised with no select-all.

**The select-all control unmounts itself on activation.** It renders only while
selectedCount < loadedCount, so pressing it makes its own condition false.
After the click, document.activeElement is body. A keyboard user is returned to
the top of the document. The ticket table's equivalent is a checkbox that stays
mounted and flips; the docblock argues carefully for moving the control to the
bar and does not carry over the one behavioural property the header cell had.

And the test enshrines it: `expect(queryByRole('button', { name: /^Select all/
})).not.toBeInTheDocument()` — a green assertion that the focused control was
removed.

**MAX_CUSTOMER_BULK_IDS makes two false claims and enforces nothing.** Its
docblock says the value is deliberately not MAX_CUSTOMER_PAGE_SIZE, "which is
how the ticket bulk bodies are bounded". The ticket bulk bodies are bounded by
MAX_PAGE_SIZE, which is 500 — the same number. So the prose describes a
considered departure from a value it is identical to, and cites a constant that
plays no part in the decision.

The comment then names the real risk — a selection accumulates pages — and the
code leaves it open. No client-side cap anywhere; the constant is referenced
outside its declaration exactly once, in a comment. When it trips, the outbound
.parse() throws a raw ZodError and toErrorMessage renders the JSON blob across
the card. This is the first place in the repo where that parse is reachable.

**The bulk error band has no way to go away.** bulkError reads from mutation
state, and the only reset() calls are inside the two handlers, each resetting
the other mutation. Nothing resets on Clear selection, on a filter change, or on
unmount — verified. The load-error band directly beneath it is tied to query
state and carries Try again. Two bands, same component, adjacent in the file:
one clears itself, one is permanent.

**leading's doc still recommends what selection was built to forbid.**
`/** An avatar, an icon, a checkbox: whatever the row leads with. */` sits
eleven lines above the new prop whose docblock explains that a checkbox cannot
go in leading. Three new statements say it must not; the one-line doc on leading
itself still lists it. leading's tooltip is what an editor surfaces at the call
site.

### What I got wrong

m08-1 misfiled a path. `border-0 bg-transparent p-0` is in TicketsToolbar.tsx,
not CustomersToolbar.tsx — which contains no Alert at all, and whose only change
on this branch was the planOptions rename. So "twice in one feature" is wrong
twice: two different features, and one of them is pre-existing ticket code this
branch never touched.

The diagnosis held and only the evidence was sloppy, which is exactly what makes
it dangerous: a misfiled path is easy to wave away wholesale.

### The disagreement

m08-3 said the strip carries `bg-primary-subtle px-4 py-3` by hand and gives
Apply no variant.

The auditor agrees on Apply — primary beside a danger Delete in one strip, two
competing emphases where the destructive one should be the only one shouting —
and says it missed that.

It disagrees on the padding. `px-4 py-3` matches the existing ticket bar exactly
and is the toolbar scale, not the card scale AGENTS.md forbids. Its reading is
that the framing is right — the code follows the letter of every rule and none
of them answers it — but that makes it a guide gap and not a code defect.

Recorded rather than averaged. m09 later measured the same gap at thirteen lines
of feature code, and harness-05 answered it with a Toolbar primitive.

### The inventories were incomplete in opposite directions

The full picture at this commit is three Alert overrides in two shapes:

    band  CustomersPage.tsx:259 (new) and TicketListPage.tsx:217 (pre-existing)
    bare  TicketsToolbar.tsx:26

The blind audit found both bands and missed the bare one. I found the bare one
and missed the second band.

Alert's variant prop, added later in harness-01b, opens its docblock with "Three
screens had…" — the union of what the two audits saw separately, and what
neither saw alone.

### Status on main

All eight are now closed. Six were fixed directly, on branch
`fix/blind-audit-findings`; two had already been closed by later harness work.

Fixed by later work, before the audit was run:

    leading's doc               harness-01b
    the ErrorBand duplication   harness-01b, via Alert's variant prop
    the Checkbox code half      harness-01b

Fixed directly, in order of severity:

**The dialog error.** ConfirmDialog gained an `error` prop rather than the page
stuffing an Alert into `children`. The argument for putting it in the primitive
is the one its own docblock already makes for owning `isBusy` — every
confirmation in the app that fires a request has the same hole, and four call
sites each choosing a tone and a position is how drift starts. `children` is the
question; `error` is the answer to the last time it was answered.

CustomersPage split its one `bulkError` into `planError`, which stays on the
card where nothing covers it, and `deleteError`, which goes in the dialog. The
delete mutation resets when the dialog opens rather than when it closes:
`mutate`'s callbacks belong to the observer being reset, and dismissing the
dialog mid-flight is a real sequence here — resetting on close would kill the
`onSuccess` that shuts the drawer over a just-deleted customer.

**Select-all replacing rather than unioning.** `selectAllLoaded()` now unions the
on-screen ids into the stored selection. The test sits in the intersection the
old pair of tests left uncovered: tick a row, filter it out of view, tick
another, press select-all, clear the filter, assert the first is still ticked.

**The self-unmounting control.** It is a Checkbox now, mounted always, checked
when everything loaded is ticked and indeterminate otherwise — which is its
state for every selection that put the bar on screen. A new `onDeselectAll`
prop was needed rather than reusing `onClearSelection`, because unticking
"everything loaded" is not "everything": routing it through clear would have
recreated the second defect on the opposite transition.

One consequence stayed and is documented rather than hidden: unticking with no
filter empties the selection, the bar unmounts, and focus does drop — identical
to pressing Clear selection, which has always done that. It is the bar's shape,
not the control's.

**The permanent error band.** `forgetBulkFailure()` runs on a filter change and
on Clear selection, guarded on `isError`. The guard is load-bearing: filters are
not disabled during a bulk action, and `reset()` on a running mutation takes its
`onSuccess` with it, so typing in the search box mid-request would have silently
orphaned `forgetSelected`.

Unmount needed nothing, and that was checked rather than assumed — a throwaway
test failed a plan change, unmounted, and remounted against the same
QueryClient; the band did not return, because mutation state lives on the
observer. An unmount cleanup would have been dead code with a false comment on
it. Recorded in the docblock, probe deleted.

**MAX_CUSTOMER_BULK_IDS.** The rationale now says what is true: 500 is the same
number as MAX_PAGE_SIZE, not a departure from it, and MAX_CUSTOMER_PAGE_SIZE was
never what bounds the ticket bodies. It stays a constant of its own because the
two answer different questions that happen to share an answer — a ticket
selection is made inside the page showing it, so one bound does both jobs there;
a customer selection accumulates pages, so the page size would cap a growing
selection at one page of it.

The bound is now enforced at the two places a selection can exceed it.
`selectAllLoaded()` caps the union. The bar derives
`selectableCount = min(loadedCount, maxBulkIds)` so the control says what it will
actually do — 600 loaded reads "Select all 500" and checks at 500, rather than
saying 600 and looking broken on the second press. Hand-ticking past the cap
disables Apply and Delete with an inline warning, which is the path that used to
reach the outbound `.parse()` and render a ZodError blob across the card.

**Apply's variant.** Now `secondary`, and the ticket bar was fixed in the same
change — it carried the identical defect and was outside the brief I gave. That
is the divergence pattern in this record caught before it settled: a newer
screen gets the correct behaviour and the older one keeps the defect, because
the brief named one of them.

**The stale doc line.** packages/ui/README.md now states the rule and records the
miscount rather than repeating it. The "Never hand-build a checkbox" section
above it still counts a different three; that census was taken on the harness
branch before ListRow had a `selection` at all, so it is right for its moment and
was left alone.

429 tests, up from 417. Twelve new, and every one of them was verified to fail
against the code it was written for.

### The hypothesis

I withheld the observation that the agent never asked about the selection
mechanism, the placement, or the API shape. All three came back anyway:

    selection mechanism  →  the replace-not-union bug, the focus loss
    placement            →  the error outside the dialog, ErrorBand
    API shape            →  MAX_CUSTOMER_BULK_IDS

Three unasked questions, three clusters of defects, found by an auditor that did
not know the questions existed. The agent asked the one question whose answer it
could not guess — which actions — and silently guessed on three it could.
