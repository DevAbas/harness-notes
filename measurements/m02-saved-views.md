# Measurement 2 — "Add saved views. Save the current filter combination with a
name, sidebar, switch, rename, delete, localStorage, show active and modified."

Task: Add saved views. Save the current filter combination with a name,
      sidebar, switch, rename, delete, localStorage, show active and modified.
Branch: measure/saved-views
Harness state: none (README only)
Baseline: baseline-small
Codebase: ~4 screens, ~30 source files, 43 tests

7 files.

## m02-1 HTML entities instead of icons
Pencil and multiply signs written as &#9998; and &#215;. No Icon primitive
exists, so text characters were used. Font-dependent, inconsistent across
platforms, and × is a multiplication sign, not a close icon.
Caught by: nothing
Layer: guide missing (no Icon primitive)

## m02-2 Button weight is guesswork
"Save current filters" is primary, same visual weight as "New ticket". In
measurement 1 the same agent chose secondary for "Export CSV". Tokens exist,
hierarchy rules do not.
Caught by: nothing
Layer: guide missing (no rule for when to use which variant)

## m02-3 Row actions are keyboard-heavy
Three ghost buttons per row, all in the tab order. With many views this is
tedious. Not written down anywhere, so not the agent's fault.
Layer: guide missing

## m02-4 Everything stays inline
SavedViewsSidebar keeps two blocks inline that should be their own components:
the 25-line row inside views.map, and three ternaries rendering a discriminated
union. "All tickets" repeats nearly the same row markup a second time. Row
classes live in module constants, away from where they are used.

The row cannot be tested on its own. The union loses exhaustiveness checking,
so a fourth dialog kind would silently render nothing.

Pattern, not a one-off: the same file avoids extraction twice.
Caught by: nothing — no complexity, nesting, or file-length rule in eslint
Layer: guide missing + sensor missing

## m02-5 No ConfirmDialog primitive
Delete confirmation is hand-assembled again: Modal, secondary Cancel, danger
confirm. The same structure already exists in TicketListPage for bulk delete.
Third time this shape is written by hand.

Note the inconsistency inside one file: the name dialog was extracted into
SavedViewNameDialog, the delete dialog was not.
Caught by: nothing
Layer: guide missing (no ConfirmDialog in the design system)

## m02-6 useCallback with a changing dependency
All three callbacks in useSavedViews depend on `views`, which changes on every
mutation, so the memoisation gives nothing back. Worse, each closes over a
snapshot of `views`, so two calls in the same render cycle would lose the first
result. A functional setState would fix both.
Caught by: nothing — eslint exhaustive-deps is satisfied, the deps are correct
Layer: guide missing (no rule on functional updates)

## m02-7 "Modified" badge on a view that does not exist
isViewModified compares against DEFAULT_FILTERS when no view is selected, so
"All tickets" shows a Modified badge as soon as any filter is set. Nothing has
been modified — no view is selected.
Caught by: nothing — 11 tests pass, none covers this state
Layer: not harness — logic bug from an underspecified requirement

## m02-8 Typography assembled by hand on every page
<h1 className="text-2xl font-semibold text-fg"> is built inline. No Heading or
PageTitle primitive exists, so every page picks size, weight and color
separately. Colors have semantic tokens; typography does not.
Caught by: nothing
Layer: guide missing

## m02-9 CardBody exists and was ignored
TicketListPage writes a raw div with `px-5 py-4` — exactly what CardBody
provides. The README says to use the primitive when one exists.

Padding inside cards is now four different values across the app: p-2,
px-3 py-2, px-4 py-3, px-5 py-4. Gaps between blocks are consistent
(gap-6 outer, gap-4 inner); padding inside a card is not, because nothing says
what it should be.
Caught by: nothing — every value is on the spacing scale, so no lint rule fires
Layer: README rule broken + guide missing (no card padding rule)

## m02-10 The design system itself is the gap
CardHeader hardcodes h2 and text-lg, so a sidebar card and a main card get
identical heading weight. Typography is raw Tailwind throughout, while colors
have a full semantic scale. CardFooter is the only place using
bg-surface-muted, with no rule saying why.

Root cause behind m01-2, m02-1, m02-5, m02-8, m02-9 and this: the system has
primitives but no composites and no typography layer. The agent fills each gap
by hand, and fills it differently each time.
Layer: the harness is missing pieces, not just rules
