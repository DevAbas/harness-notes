# Measurement 5 — "Add a section to the settings page where an admin can see
and edit the ticket statuses and priorities."

Task: see m05-prompt.txt
Branch: measure/status-admin
Harness state: none (README only)
Prompt style: one line, plus two clarifying answers the agent asked for
Baseline: baseline-m04
Codebase: ~4 screens, ~70 source files, 105 tests
Files touched: 26 modified, 6 new

This measurement was set up to answer one question: does the agent implement
against the existing design system consistently, or diverge from it. Findings
are limited to that. The same branch also produced findings about architecture
and correctness — one of those is recorded in observations.md because it
describes a property of the method rather than of this measurement.

Before building, the agent stopped and asked two questions. I answered: editing
means label and badge appearance only, not adding or removing values, and the
unions stay compile-time constants; and persist on the server, consistent with
tickets.

## m05-1 Icons are still bare Unicode glyphs
The reorder controls render ↑ and ↓ as text inside a ghost Button:

    <button aria-label="Move pending down" class="...">↓</button>

Font-dependent, sized by text-sm rather than an icon scale, colour inherited
from text. There is still no Icon primitive, so an icon arrives as a character.
Second confirmed instance — m02-1 was ✎ and × in SavedViewsSidebar, three
measurements earlier, and nothing caught it in between.

The aria-label is present and names the row, which the README does require.
Caught by: nothing
Layer: guide missing (no Icon primitive; a lint rule banning non-ASCII glyphs
in JSX text closes both instances permanently)

## m05-2 A third heading treatment, hand-assembled
"Statuses" and "Priorities" are section headings inside a card body. The design
system has page titles, hand-built in each page, and CardHeader, and nothing
for a level between them, so one was invented here.

Same root cause as m02-8: colours have a semantic scale, typography does not.
Caught by: nothing
Layer: guide missing

## m05-3 Table rows change height when errors appear
A new row renders a validation message under both the value and the label
field, so the row grows and the appearance select, preview and actions sit
mis-centred against it. Nothing reserves space for the error state; row height
is whatever the tallest cell happens to be. The error text also does not align
between the two columns.

The design system has no row-with-a-field primitive, so the layout was
assembled by hand.
Caught by: nothing — no visual regression sensor exists
Layer: sensor missing (visual), guide missing (no layout primitive)

## m05-4 The preview cell collapses to a stub
An empty row renders TaxonomyBadge with an empty label, producing a small grey
pill rather than a badge or an empty state. It reads as broken rather than
pending.
Caught by: nothing — a component test asserting that a badge with no label
renders something meaningful would catch it cheaply
Layer: sensor missing

## m05-5 A header-only card for the locked state
For a non-admin, the section renders a Card containing only a CardHeader. The
header's border-b is left dangling with nothing beneath it, so the card looks
unfinished rather than locked. The role card above it has a header, a divider
and a body.

The comment above it reads "Rendering nothing rather than a disabled editor
keeps an agent from reading it as something to unlock" — but it does not render
nothing, and the description it renders says "Switch role above to edit them",
which is precisely presenting it as something to unlock.
Caught by: nothing
Layer: guide missing (no locked or empty card state)

## What was done well
The design system was extended correctly, for the first time in five
measurements. labelHidden was added to both Input and Select with identical
implementations, rendering the label sr-only while keeping it as the control's
accessible name — the docstring states plainly that it is never a way to have
no label. The README was updated in the same change, in the existing voice,
documenting both the prop and the requirement that the hidden name still
identify the row.

The Badge section of the README was rewritten under real pressure: ticket
statuses became configuration while BadgeStatus stayed a fixed five-value
union. The rule that the design system must not learn what a ticket is was
kept, the mapping moved to features/taxonomy/appearance.ts, and the reasoning
was written down — that the two unions sharing five words is a coincidence, and
the table is what keeps it one rather than a dependency.

Every colour is a semantic token. No arbitrary values. No hand-rolled buttons.
Primitives are used where they exist.

## What this measurement shows
The pattern is unchanged from m01 through m04, now on the measurement designed
to test it directly. Stated rules held. What the design system provides was used
correctly. What it does not provide — icons, section headings, a row containing
a field, an empty or locked card state — was hand-assembled, and assembled
differently from the last time the same gap came up.

The distribution of what could catch these differs from earlier measurements.
One is closable by a deterministic lint rule. Two need a written guide. Two need
a visual regression sensor the repo does not have and nothing in the earlier
measurements called for. Visual consistency is the first category where the
computational sensors that carried m01 through m04 mostly do not apply.
