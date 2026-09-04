# Measurement 13 — "There are two saved-view mechanisms in this app and there should be one."

Task: see m13-prompt.txt
Branch: measure/13-saved-views-unified
Harness state: layers 1, 1b, 1c, 2 and 3 in place; layer 4 not built
Prompt style: detailed brief, measured
Baseline: fix-m12
Codebase: ~7 screens, 332 modules, 535 tests before / 561 after
Files touched: 5 modified, 3 deleted, 3 new plus one new directory
Lint: 5 warnings, unchanged
Duration: 12 minutes API, 36m wall, $6.34

The brief was wrong. It said there were two saved-view mechanisms to unify;
on main there is one. The customer segments that m06 built were never merged
— measure/customer-segments sits unmerged, and I read the note as if it
described main.

The agent found this before starting and said so, and reformulated the task
as one mechanism with the customer list getting its saved views through it.
That is the right reading and every other constraint held.

So this measures something the brief did not set out to measure: what an
agent does with a premise that is false.

Audited blind through the packaged skill after the commit and before I read
the diff.

## m13-1 The work is presented as a unification, in comments describing a past that is not in the history

Written from the audit. The reformulation happened in chat and left no trace in
the artifact. The commit message is one line, "measure 13: one saved-view
mechanism". `useSavedViews.ts` says all three concerns "used to be split ...
which meant the second screen to want saved views had to work them out again,
the same way, from scratch" — there was no second screen.
`features/customers/savedViews.ts` defends itself against a charge that could not
apply: "Nothing below is a copy of the ticket screen's scope with the strings
changed", about a file with no predecessor.

The consequence is not the refactor, which is sound. It is that saved views on
the customer list are new user-facing functionality — a sidebar, a naming modal,
a delete confirmation, a new storage key, and a page relayout from one Card to a
two-column flex row — delivered under a commit that says unification, and that
nobody asked for.

And the brief's guarantees split in two without anyone noticing. "The two screens
keep the behaviour they have today" and "a person who saved views on either one
still has them" are load-bearing for tickets and vacuous for customers, so the
one thing worth verifying got the same weight as a screen with nothing to
preserve.

Caught by: nothing. Tests, typecheck, lint and boundaries all pass, and a
reviewer reading the diff against the brief would agree the end state matches
the ask — noticing the premise was false means going and looking at main.
Layer: not harness — no sensor can check a brief's claims against the base

## m13-2 sameFilters and the JSON round trip disagree about an absent key

`isSameValue` compares key counts. `JSON.stringify` drops a key whose value is
`undefined`. So a filter object that normalises to three keys is stored as two
and read back as two, while the live screen still produces three, and the
comparison says they differ.

The saved view is marked Modified the instant it is saved, on every reload
afterwards, with no filter change and no way to clear it — which is the exact
failure `createSavedView`'s own comment says storing canonically prevents.

Both of today's scopes are safe: neither `TicketFilters` nor `CustomerFilters`
has an optional field. The hole is an optional filter, which compiles fine, and
it is the shape the shared test file itself chose to model — `onlyMine?: boolean`,
commented "Added after the fact, the way a screen grows a filter." The test
exercises it through `sameFilters` and separately round-trips a view through
storage, and never both, which is the only combination that fails. The round-trip
test passes because `toEqual` treats a missing key and an undefined key as equal
— the very equivalence `isSameValue` refuses.

The whole point of the folder is that a third screen is one file. This fires on
the first scope with an optional filter.

Caught by: nothing. Not reachable from today's scopes, and the docblock above it
argues it cannot happen.
Layer: rule broken — the code contradicts the comment directly above it

## m13-3 The one test for the customer scope's distinctive behaviour cannot fail

`CustomerSavedViews.test.tsx` names the set-ordering case as the thing the
customer list brings that the ticket queue cannot. The code under test is the
reorder in `normaliseFilters`, and `MultiSelect.toggle` already emits in options
order — "Emitted in `options` order, so choosing the same set twice is one value."
So `filters.plans` is never out of domain order whatever sequence the boxes are
clicked in, and the reorder is an identity function on every value the test can
produce. Delete it and the test still passes.

The case the reorder exists for is named in the scope's own comment — a view
stored by some other build — and is never seeded, though the file seeds storage
directly in two other tests.

Caught by: nothing, including coverage, which reports the line as executed. It
runs; it cannot be wrong.
Layer: sensor missing

## m13-4 A hand-rolled guard where the shared contract exports the schema

`isCustomerPlan` in the new customer scope, where `customerPlanSchema` is
exported from `@support-desk/shared` and the ordering transform beside it is the
same expression already written there. Every other untrusted-input parse in
apps/web goes through a shared schema; this is the one that does not.

The transform is not exported, so using it needed a change in packages/shared —
which is the case the brief asked to be raised rather than worked around.

Weak: nothing is wrong today, and all three copies of the ordering rule derive
from `CUSTOMER_PLANS`, so none can go stale independently. Recorded because the
code is new and the primitive is one import away.

Caught by: nothing. No rule covers "a zod schema exists for this union".
Layer: guide missing

## What I checked myself

The migration, which is the brief's riskiest guarantee and has no sensor. Saved a
view on main, switched to the branch, reloaded: the view is there, named
correctly, and applies its filters. Both screens walked end to end — save,
select, modify, rename, delete — and the set-ordering case by hand. No
difference between the two sidebars.

Nothing found. Third measurement in a row where the browser pass produced no
finding, after m11 and m12.

But it is not the same as producing nothing. The migration is the one guarantee
no test covers, and the browser is the only place it can be checked. The audit
reached the same conclusion independently and by a better route — the
pre-existing ticket test that seeds legacy storage directly still passes
byte-identical, which is proof rather than an assertion.

## What was done well

From the audit: the generic split is honest — `SavedView<TFilters>` never reads a
field of `TFilters`, the six scope answers are the minimum, and the two filter
shapes stay different as the brief required. The storage key kept its old value
with the reason written down. `sameFilters` is a better idea than what it
replaced: `areFiltersEqual` named each field it compared, so a filter a screen
grew was silently uncompared, and the test documenting that defect is well aimed.
The `.map()` block was extracted per AGENTS.md rather than copied down.
`readSavedViews` uses flatMap so surviving entries are typed by construction.

## What this measurement shows

Cost tracks new concepts, not open decisions. m12 and m13 both left five
decisions open by name; m12 cost $16.32 over 31 minutes and m13 $6.34 over 12.
m12 built a domain from nothing — a transition graph, guards, a role gate, a
history, a bulk report. m13 generalised code that already existed: 174 insertions
against 522 deletions. That is the third correction to this note, after
harness-05's batching and m11's open-decision count.

The zero intersection held for a second measurement. m12's findings were all in
code and the browser found none; m13's were too. The browser pass has produced
findings on m05, m09 and harness-01c, all of them visual — three structural
measurements in a row have produced none.

And m13-1 is a new shape for the census. The record has seventeen instances of
prose describing intent the code beside it does not carry out. This is prose
describing a past the repository does not have — written to explain a change
that was reformulated, in an artifact where the reformulation left no other
trace. The agent was right about the premise and said so; the code says
otherwise.
