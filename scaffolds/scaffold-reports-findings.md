# Scaffold — reports section

Task: no prompt file survives
Branch: scaffold/reports
Harness state: none (README only)
Prompt style: detailed spec — scaffold work, not a measurement
Baseline: baseline-m04
Codebase: ~5 screens, ~90 source files, 143 tests
Files touched: 12 modified, ~30 new
Tests: 143, up from 85

Scaffold work. Recorded because the findings are useful, not because the
conditions were being measured.

## rep-1 CardFooter was rebuilt by hand
ReportToolbar writes:

    "flex flex-wrap items-center justify-between gap-4 border-t border-border
     bg-surface-muted px-5 py-3"

That is CardFooter's class string character for character, including
bg-surface-muted, which is used nowhere else. The primitive exists and was
reassembled instead of imported. Second instance of the shape: TicketsToolbar
did the same in m01, and m05 recorded CardBody being rebuilt the same way.
Caught by: nothing
Layer: README rule broken

## rep-2 The page title is hand-assembled for the fifth time
<h1 className="text-2xl font-semibold text-fg"> plus
<p className="text-sm text-fg-muted">. Fifth screen, fifth hand-built page
header. This change added five new primitives and Heading was not one of them.
Caught by: nothing
Layer: guide missing

## rep-3 Eight distinct padding values, none of them a rule violation
Added in this change: Alert px-4 py-3, StatCard px-5 py-4 (applied to Card
directly rather than using CardBody, which provides exactly that), ReportToolbar
px-5 py-3, BarChart's message state px-4 py-12. On top of the four m05 counted:
p-2, px-3 py-2, px-4 py-3, px-5 py-4.

Every value is on the spacing scale, so no lint rule fires and none ever will.
The clearest case in the record of "on the scale" not meaning "consistent", and
the strongest argument for a layout primitive over a lint rule.
Caught by: nothing
Layer: guide missing

## rep-4 A third hand-written selected state
DateRangeField styles its active preset with

    border-primary-border bg-primary-subtle text-primary-subtle-fg
    hover:bg-primary-subtle

SavedViewsSidebar, from m02, uses

    bg-primary-subtle text-primary-subtle-fg hover:bg-primary-subtle
    hover:text-primary-subtle-fg

Near-identical, differing by two classes. Button has four variants and none of
them is "selected".
Caught by: nothing
Layer: guide missing

## rep-5 BarChart reinvents the loading and empty state it imports
messageClasses is 'px-4 py-12 text-center text-sm text-fg-muted', hand-written.
TableBody already provides isLoading and isEmpty with a role="status"
announcement, and the README says loading and empty states belong there "rather
than in whichever ones remembered to do it". BarChart imports TableBody two
hundred lines further down the same file.
Caught by: nothing
Layer: README rule broken

## rep-6 Arrows are glyphs again, but handled correctly
StatCard renders ↑ ↓ → as text. Unlike m02-1 and m05-1 this is done properly:
the glyph is aria-hidden, a word is provided sr-only, and the docstring names
the reason — direction conveyed by shape alone. Not an accessibility failure.
What remains is that a glyph is font-dependent, sized by text-xs rather than an
icon scale, and this is the third file inventing its own arrow.
Caught by: nothing
Layer: guide missing

## rep-7 The design system README states something that is not true
Its opening section says the Tailwind palette has been removed, that tokens.css
clears the colour namespace with --color-*: initial, and that bg-blue-500
"produces no CSS at all". That reset was removed early on and those classes work
today. Four lines explain a safety property that no longer exists.

What is new: the agent read this file, added five sections to it, and did not
notice the opening claim was false.
Caught by: nothing
Layer: sensor missing (doc freshness)

## rep-8 The README describes an approach three components do not share
"Loading and empty states belong to TableBody via isLoading and isEmpty...
StatCard and BarChart own theirs the same way, so a screen full of figures waits
as one thing rather than as five."

TableBody has one implementation; BarChart hand-writes its own with different
padding (rep-5); StatCard writes a third. The sentence describes three
components sharing an approach when three implemented it privately.

Worse than the other comment findings because it is in the document that governs
everything else: a guide describing a more coherent system than the one that
exists will make the next agent confident about something untrue.
Caught by: nothing
Layer: sensor missing (doc freshness)

## rep-9 Dead export
toDimension in reportViews.ts is exported and never used; ReportsPage computes
the dimension inline with a ternary. Lint has a rule for unused locals, none for
unused exports.
Caught by: nothing
Layer: guide missing

## rep-10 Tabs selects by dispatching a synthetic click
Arrow-key navigation calls tabs[target].focus() then .click(). Any consumer
passing its own onClick to Tab has it fired by an arrow key. Calling
onValueChange directly would be cleaner. The docstring does state that moving
focus also selects, as a deliberate choice.
Caught by: nothing
Layer: not harness — minor

## What was done well
The feature code uses every primitive that exists — Table, Card, CardBody,
CardHeader, Button, Tabs, Alert, StatCard, BarChart. No raw button, no raw
table, no hand-assembled card chrome anywhere in features/reports.

Alert is a good primitive and closes two earlier findings as a class: shrink-0
on the action slot fixes the collapsing "Try again" button from m05, min-w-0
handles long unbroken strings, and role is derived from tone rather than left to
the caller.

BarChart genuinely contains the library to one file. Every fill reads
var(--color-*) from tokens.css — bars, grid, axis, tooltip, labels — so the
chart belongs to the design system rather than to recharts. The drawing is
aria-hidden with an sr-only Table carrying the same figures, which is the
correct pattern for a chart and is usually skipped.

Tabs is a correct ARIA implementation: roving tabindex, arrow keys with
wraparound, Home and End, id pairing, aria-controls only when the panel exists.

toStatChange takes a polarity per metric — more-is-better, less-is-better,
neutral — so the colour of a change is correct by design rather than by
coincidence. The README section generalising the Badge rule to AlertTone,
ChartTone and StatChangeIntent is the strongest piece of writing in that
document.

## State this leaves the repo in, deliberately
Alert now exists and is correct, and the two hand-rolled banners in
TicketListPage and TicketsToolbar are still in place, unmarked. Nothing
indicates which is preferred. That is the property later measurements are meant
to test, so it stays.
