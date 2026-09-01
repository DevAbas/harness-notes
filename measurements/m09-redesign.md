# Measurement 9 — "Apply a new visual identity."

Task: see m09-prompt.txt
Branch: measure/09-redesign
Harness state: layers 1, 1b, 1c, 2 and 3 in place; layer 4 not built
Prompt style: full token values supplied, plus four rounds of correction
Baseline: harness-01c
Codebase: ~7 screens, ~200 source files, 407 tests
Files touched under apps/web/src/features: 9, 13 lines
Lint: 8 warnings, down from 15

The measurement layer 1 was built for. Every token changes and no component
logic does, so the question is simply how much feature code a new visual
identity has to touch.

Answer: nine files, thirteen lines, all of it one thing.

Press states were added in the same session, after the redesign landed, and
the colour corrections that followed are recorded here because three of them
are findings in their own right.

## m09-1 Nine files, one cause: there is no primitive for a strip
Every one of the thirteen lines is padding on a strip that sits above or inside
a card and is not a card:

  TicketsToolbar, CustomersToolbar, BulkActionsBar, CustomersBulkActionsBar,
  Pagination, TicketsTable's footer, CustomerDrawer, CustomersPage,
  TicketDetailPage

All of them carried `px-4 py-3` by hand and had to become `px-3 py-2`.

CardBody owns card padding and AGENTS.md says so. Nothing owns strip padding,
nothing says anything about it, and so the density change could not reach it
through a token.

This is m08-3 measured. That finding said the rules are written at screen scale
and a strip is not a screen; this is the same gap costing thirteen lines the
first time the scale moved. It will cost the same again on every screen added
before a Toolbar primitive exists.
Caught by: nothing — every value is on the spacing scale
Layer: guide missing (no strip primitive)

## m09-2 The subtle tints collapse into each other
On the customers list, Free, Pro and Starter are three badges nobody can tell
apart. On the ticket list, Open, Resolved and Medium are the same problem.

The cause is in the palette I supplied rather than in the work. --color-info was
derived from --color-primary because the source palette had no blue, and
--color-success was added to it for the same reason. All three sit in the teal
family, and at subtle lightness they converge.

What makes it a finding rather than a bad colour choice: the saturated versions
of the same three are clearly distinct — the report chart renders four bars in
info, warning, success and muted and every one is legible. So the palette is
fine and the derivation of the subtle tints is not. Five statuses need five
distinguishable tints, and deriving them by lightening a family that has three
members does not produce five.
Caught by: nothing
Layer: not harness — a palette derivation problem, visible only on screen

## m09-3 A token edit reported three times and applied once
The tint tokens shipped with the value I gave them,
rgba(84, 11, 14, 0.05) and 0.10 — which is the foreground colour, so hover and
press read pink rather than as a neutral darkening.

I asked twice for plain black. Both times the change was reported. Both times
`grep` showed the file unchanged. It landed on the third ask.

Nothing in the transcript distinguishes a reported change that landed from one
that did not. A large change shows in the diff stat and on the screen; a
two-line token edit shows in neither, and the only thing that caught it was
reading the file.
Caught by: reading the file, three times
Layer: not harness — a property of how corrections are verified

## m09-4 An unmeasurable instruction gets a small move every time
--color-primary-subtle was #E4EDEF, a blue-grey against a cream body, and read
cold everywhere it landed: the active nav item, the Admin badge, the selected
saved view.

I asked for it to be "warmed". It went to #ddeee9. Still cold. I asked again
with an exact hex and it took it, and was still wrong — now green, which is
where success lives in this palette.

The answer was not a colour at all. The nav did not want a primary tint; it
wanted --color-surface-inset, the palette's own warm neutral, which collides
with no status colour and agrees with the neutral hover tint beside it.

Recorded because two rounds were spent moving a value when the finding was that
the value was the wrong mechanism. "Warmer" is not measurable and produced a
small move each time it was asked; the exact hex was measurable and produced
exactly the wrong thing.
Layer: not harness — badly specified target

## m09-5 The lint rule cannot see apps/web/src/app
AppLayout's nav link carried `text-sm` and no rule fired.
no-raw-type-classes scopes to apps/web/src/features, which was the whole of the
app when the rule was written. app/ has since grown to hold AppLayout,
AppRoutes and the auth routes.

Three violations remain in features and the rule reports them. Whatever sits in
app/ it does not see.
Caught by: nothing — the rule does not look there
Layer: rollout — a scope chosen against a structure that has since changed

## What the harness did

The whole point of the measurement, and it holds.

Every colour, every type size, every radius, every shadow and every control
height changed from tokens. The report chart re-coloured itself with no code
change at all, because BarChart reads var(--color-*) rather than a palette —
that is the cleanest single proof layer 1 works.

The focus ring, the control heights and the motion durations were all
tokenised in layer 1c specifically so this change would not have to reach them.
None of them appears in the diff.

Lint went from 15 warnings to 8. Seven no-raw-type-classes violations cleared as
a side effect of the type work — not asked for, and a consequence of touching
every type class in the design system.

And Alert was never switched off with className, Checkbox was never hand-built,
no glyph was reached for. The three things layer 1b closed stayed closed through
a change that rewrote every surface they sit on.

## What this measurement shows

A full visual identity change — palette, typeface, type scale, radius, shadow
and density — cost thirteen lines of feature code, and all thirteen were the
same missing primitive.

That is the number the layers were built to produce. It is not zero, and the
reason it is not zero is a gap that was already named two measurements earlier
and not yet closed.
