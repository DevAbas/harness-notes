# Scaffold — customers directory

Task: see scaffold-customers-prompt.txt
Branch: scaffold/customers
Harness state: none (README only)
Prompt style: detailed spec — scaffold work, not a measurement
Baseline: baseline-reports
Codebase: ~6 screens, ~100 source files, 143 tests

Scaffold work. The goal was a second list pattern and a second detail pattern,
so later measurements have two legitimate answers to choose between rather than
one. Recorded because the findings are useful, not because the conditions were
being measured.

## cust-1 The multi-select was not built
The prompt asked for "a multi-select filter for plan, not single-value selects".
The Plan filter is a plain Select showing "All plans".

Second instance of an explicit instruction not being followed, after the union
deletion recorded in observations.md. That makes it a pattern rather than a
one-off — and it removed the one control that would have forced a genuinely new
selected-state pattern into the design system.
Caught by: nothing
Layer: not harness — an instruction overridden

## cust-2 Drawer duplicates Modal almost entirely
Same useId pair, same ref, same effect body, same Escape handler, same focus
restore, same portal, same bg-black/40 overlay, same onMouseDown backdrop
dismissal, same role, aria-modal, aria-labelledby, aria-describedby and
tabIndex={-1}. The header and footer blocks are identical class strings. The
close button is the same &#215; in both.

The genuine differences are roughly fifteen lines: positioning, the animation,
min-w-0 and truncate on the title, a side prop, and the scroll container.

The docstring argues that a modal interrupts and a drawer accompanies, which is
correct — but that argument is about purpose, not implementation. A shared
Dialog base with two presentations would have kept the argument and removed the
copy. As built, every dialog bug now has to be fixed twice.
Caught by: nothing — no duplication rule, and the two files are far apart
Layer: guide missing

## cust-3 A pre-existing defect was reproduced faithfully
Neither Modal nor Drawer traps focus. Both set aria-modal="true", which tells
assistive technology the rest of the page is inert, while Tab moves out of the
dialog into the page behind it.

Drawer's docstring says it "keeps Modal's dialog contract" — and it does,
including the gap in it. The design system README lists that contract as role,
aria-modal, an accessible name, Escape, and focus moved in and restored, which
stops one item short of what aria-modal promises.

This is the clearest instance in the record of the agent replicating a pattern
already in the repository, including the flawed part, because the new component
was written by copying the old one.
Caught by: nothing — no test tabs through a dialog
Layer: sensor missing (no keyboard-navigation test), and the README documents
the incomplete contract as if it were complete

## cust-4 CardHeader and CardFooter rebuilt inline, third and fourth instance
Drawer's header is CardHeader's class string character for character, including
the inner flex min-w-0 flex-col gap-1 wrapper and the title/description
structure. Its footer is CardFooter's, including bg-surface-muted.

Modal does the same. TicketsToolbar did it in m01, ReportToolbar did it in the
reports scaffold. Four copies of the same two blocks now exist.
Caught by: nothing
Layer: README rule broken

## cust-5 A third copy of the same empty and loading state
List defines messageClasses = 'px-4 py-12 text-center text-sm text-fg-muted'.
BarChart, from the reports scaffold, defines the identical string. TableBody has
a third implementation.

List's docstring says loading and empty states belong to it "for the same reason
they belong to TableBody" — the right principle, stated while implementing it a
third separate time rather than sharing it.
Caught by: nothing — identical string literals in three files, no duplication
rule
Layer: guide missing

## cust-6 A fourth hand-written selected state
ListRow: isSelected && 'bg-primary-subtle hover:bg-primary-subtle'

After SavedViewsSidebar (m02), DateRangeField and Tab (reports). Each slightly
different. Button still has no selected variant, so every component needing one
writes it again.
Caught by: nothing
Layer: guide missing

## cust-7 Badge stretches to full width in the drawer
The same Badge renders as a pill in the list and as a full-width bar in the
drawer. ListRow wraps trailing content in flex shrink-0 items-center; the
drawer's layout has no equivalent, so the badge inherits stretch from its flex
parent.

The component is correct and the usage is wrong. Nothing catches it: no test
renders a badge inside a stretching parent, and there is no visual sensor.
Caught by: nothing
Layer: sensor missing (visual)

## cust-8 StatCard exists and was not used
The drawer hand-assembles two mini-stats — "Signed up / 26 May 2026" and
"Tickets raised / 8" — which is exactly what StatCard does. Same shape as the
CardFooter duplication: the primitive exists, its layout was rewritten.
Caught by: nothing
Layer: README rule broken

## cust-9 The close control is a glyph, fourth instance
&#215; in both Drawer and Modal. It carries an aria-label, so this is not an
accessibility failure — but the multiplication sign is still standing in for an
icon in a design system that now has around fifteen primitives and no Icon.
After m02-1, m05-1 and StatCard's arrows.
Caught by: nothing
Layer: guide missing (no Icon primitive)

## What was done well
The second list pattern is genuine, not a table with the borders removed. List's
docstring makes the distinction properly: a table is a grid where every row
answers the same questions in the same order; a list has a shape and is scanned
down rather than read across.

ListRow uses aria-current rather than aria-selected, with the reasoning written
down — nothing here is a listbox, the row is not being chosen, it is the one
whose detail is open beside it. Most implementations get that wrong.

Avatar is correct. Initials derived from the name, a tone hashed from the name
so the same person is the same colour everywhere, and the docstring explains why
the tone is derived rather than passed: a caller choosing one would be a caller
saying something by it. The image fallback tracks which src failed rather than a
boolean, so a new picture is retried without an effect resetting a flag.

Seed avatars are locally generated data URIs, not external URLs — no network
calls, works offline and in CI. The mix of pictures and initials exercises both
paths of the component.

Drawer restores focus to the row that opened it, and honours prefers-reduced-
motion by dropping the movement rather than the drawer.

## State this leaves the repo in, deliberately
Two list patterns, two detail patterns, two error-display approaches, three
empty-state implementations, four selected-state implementations, and no
document saying which applies when. That ambiguity is what the next measurement
is for.
