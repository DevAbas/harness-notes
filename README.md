# Failures log

Repo: a support tickets admin panel, since extended with a reports section and
a customers directory. React, TypeScript, Tailwind, and its own design system
with a README that spells out the rules.

Nothing enforces those rules. No AGENTS.md, no CLAUDE.md, no lint rule for the
design system. The rules are written down and that is all.

Question being measured: does the agent follow rules that are written but not
enforced?

That is the starting state, and the first six measurements answer it. The repo
has since grown a harness in layers — primitives, a typography and token layer,
lint rules, an AGENTS.md — and every measurement after m06 tests what a layer
closes.

How a measurement is run, what each file's header carries and what the layer
labels mean is in METHOD.md. This file is the result.

---

# Measurements with no harness

Each ran with the harness in its default state — README only. Prompt style
varies and is named on each line, because it is part of what is being measured.

Two scaffolds — the reports section and the customers directory — were built in
this same regime, and a third, authentication, came later with layers 1, 1b, 2
and 3 in. All three are summarised under The three scaffolds below; their
findings are cited throughout as rep-N, cust-N and auth-N.

## m01 — export to CSV
One line, nothing specified. Baseline baseline-small, 6 files. Four findings: no
BOM in the CSV, a second error banner built by hand, cn() skipped, an export
error that is never cleared.

## m02 — saved views
A short paragraph naming the features. Baseline baseline-small, 7 files, 11
tests green. Ten findings, and the first statement of the root cause: the design
system has primitives but no composites and no typography layer, so the agent
hand-assembles what it cannot import, and assembles it differently each time.

## m03 — HTTP API and TanStack Query
A detailed spec; scaffold work rather than a feature. Baseline baseline-merged,
~40 files, 82 tests green. The design system held completely and everything the
spec named was built well. What broke sits between the named pieces: a CSV
export that silently caps at 500 rows, a zod parse that throws past the error
channel its own docstring promises, a per-field `details` array plumbed end to
end and read by nobody, three env variables validated to three different
standards. A detailed spec raises the floor on everything it names and does
nothing for the joins.

## m04 — assign a ticket from the detail page
One line, nothing specified. Baseline baseline-workspaces, 3 files. The smallest
change in the record on the largest codebase so far, because PATCH
/tickets/:id already accepted assignee from m03 — which disproves the size
hypothesis m02 raised. Findings: a deprecated FormEvent import copied from m02's
file, an error message instructing the user to type a magic string, no max
length on a field the contract caps, and button weight guessed the same way as
m02-2.

## m05 — status and priority admin
One line, plus two clarifying questions the agent asked and had answered.
Baseline baseline-m04, 26 files modified and 6 new. Set up to test design-system
consistency directly, and the pattern held: stated rules kept, existing
primitives used, and the design system extended correctly for the first time —
labelHidden added to Input and Select, with the README updated in the same
change and in the existing voice. What the system does not provide was
hand-assembled again. Two of the five findings need a visual regression sensor
the repo does not have, a category the earlier measurements never reached.

Separately, and recorded in observations.md because it is a property of the
method: the agent deleted the two unions it had just been told stay
compile-time constants.

## m06 — saved segments for the customers directory
One line, nothing specified. Baseline baseline-customers, 222 tests green, up
from 143. Set up to test ambiguity — the repo already answered "how do you save
a filter combination" once, in m02, on a screen with different conventions and a
different filter shape, and nothing says whether to follow that precedent.

The agent noticed the choice and argued one side in writing, naming the cost of
its own decision. That is new. Then it copied the file it had just explained it
was not following: the glyphs, the inline block, the three ternaries, the
missing ConfirmDialog, the selected-state string character for character, and
the naming dialog wholesale.

---

# What the measurements with no harness show

I expected the design system rules to break. Almost none of them did.

Every colour is a semantic token. No arbitrary values. No hand-rolled buttons.
Badge statuses stay inside the union. Accessible attributes are there. Out of
everything the README actually states, six things were broken across six
measurements and two scaffolds: cn() was skipped once (m01-3), CardBody was
ignored once (m02-9), CardFooter and then CardHeader were rebuilt inline rather
than imported (rep-1, cust-4), BarChart reinvented the empty and loading state
TableBody provides and that it imports two hundred lines lower (rep-5), and
StatCard was rewritten by hand rather than used (cust-8).

The data layer was repeatedly stronger than asked for. RFC 4180 CSV with formula
injection guarding, full runtime validation of everything read from
localStorage, and in m06 type guards that drop stored entries that no longer
parse. None of it was requested.

So the problem is not that written rules get ignored. The problem is uneven
output where nothing is written — the same gap, filled again, filled
differently.

## The same findings recur, with nothing added to the harness between them

- Glyph icons standing in for an Icon primitive: five instances (m02-1, m05-1,
  StatCard's arrows, the Modal and Drawer close buttons, m06-1). m06 used the
  same two codepoints as m02, on the same two buttons.
- Hand-built confirm dialogs: five, and twice inside a single file the name
  dialog was extracted while the delete dialog was left inline.
- Hand-written selected states: five, two of them character for character
  identical.
- Duplicate empty and loading states: four, three of them the same string
  literal.
- Hand-assembled page titles: six screens.
- Blocks left inline that should be components: m02-4 reproduced in shape and in
  scale as m06-2, four measurements later.

All 222 tests pass. Typecheck and lint are clean. Every finding above passes all
three.

## Written instructions are not reliably followed either

The "rules held" claim covers the design system README. It does not extend to
prompt instructions. Two were overridden outright: m05 was told the status and
priority unions stay compile-time constants and deleted them, and the customers
scaffold was told to build a multi-select plan filter and built a plain Select
(cust-1). Both registered as success — typecheck, lint and tests green, and in
m05 more easily green than before. Two instances is a pattern, not a one-off.

## The agent reasons about the thing it is looking at

m05 stopped and asked two questions before building. m06 identified a genuine
design fork and argued its side. Both are new, and neither propagated downward:
m05 built the version its own question had excluded, and m06 copied everything
beneath the layout decision from the file it had just declined to follow.

---

# Measurements with a harness in place

Same method, different baseline: each runs against a tagged harness state rather
than against README-only.

## m07 — export to CSV on the customers list
One line, nothing specified. Baseline harness-03, layers 1, 2 and 3 in place, 3
files modified and 3 new, 324 tests. A deliberate re-run of m01 on a different
screen, so the comparison is against a recorded run rather than a memory.

Four findings again, and of a different kind. Three of m01's four are now
impossible: Alert exists and is used, cn() is enforced, the CSV writer is
shared. What is left sits further out — a runaway guard that truncates in the
way its own docstring argues against, Alert imported for its tone and role and
then switched off with className, a third name for one constant, and an existing
lint warning left untouched in an edited file because the rule ships at warn.
That last one is the first direct evidence of what the warn tier costs.

## m08 — bulk actions on the customers list
One line, plus one clarifying answer. Baseline harness-03, layers 1, 2 and 3 in
place, 23 files modified and 3 new. The first task that had to extend the design
system rather than only consume it.

Every one of the five findings is a missing primitive or a missing rule. None is
a broken rule, and none is a defect in what was built. The Checkbox gap was
named by the agent, in the file, with its three migration sites counted — in m01
through m06 a gap like that was filled silently. m08-3 marks the boundary the
next measurement prices: AGENTS.md is written at screen scale, and a strip above
a list is not a screen.

Audited a second time later, blind, by a fresh agent through the packaged skill,
which found eight. Four converged with mine and in every one of the four the
blind pass went further; five were new, one of mine was wrong in its evidence
and one was disputed. Eight is the number m10 is compared against, and all eight
are closed on main. The two passes answer different questions — one names the
missing harness layer, the other names what shipped through the hole — and the
census neither saw alone is recorded in observations.md.

## m09 — a new visual identity
Full token values supplied, plus four rounds of correction. Baseline
harness-01c, layers 1, 1b, 1c, 2 and 3 in place, 407 tests. The measurement
layer 1 was built for — every token changes and no component logic does, so the
question is only how much feature code a new visual identity has to touch.

Palette, typeface, type scale, radius, shadow and density, for nine files and
thirteen lines under features, and all thirteen are the same missing primitive:
padding on a strip that is not a card. That is m08-3 priced. The report chart
re-coloured itself with no code change at all, because BarChart reads
var(--color-*) rather than a palette.

Three of the five findings are not about the harness — subtle tints derived by
lightening one colour family until they converge on screen, a two-line token
edit reported three times and applied once, and two rounds spent moving a value
when the finding was that the value was the wrong mechanism. The fifth is
rollout: no-raw-type-classes scopes to apps/web/src/features and cannot see
apps/web/src/app, which did not exist when the rule was written.

## m10 — a global search
A detailed brief, measured. Baseline fix-blind-audit, layers 1, 1b, 1c, 2 and 3
in place, 20 files modified and 10 new, 492 tests up from 429. The first
measurement run with a detailed brief: every earlier detailed prompt was treated
as scaffold, and this one states what is needed, what is already decided and
what is deliberately left open. Audited blind through the packaged skill before
the diff was read.

Four findings against m08's eight, so the brief halved the count and left the
kind alone. Three of the four are confident prose the code beside it does not
do: AGENTS.md gained a gating invariant that holds for one screen of four,
close() names the stale-results risk and closes half of it — the previous
search's results are selectable for 250ms after the next first keystroke — and
Input's new docblock describes a sibling mechanism it does not match. The
fourth is a test file asserting the new role rule and breaking it four lines
later, passing because the harness has two role sources that disagree.

The design system boundary held under pressure: told to say so rather than
assemble a missing primitive in the feature, the work built CommandPalette in
packages/ui, documented it, and escalated the one class it had to write itself.
And one product decision was invented rather than asked for — reports was made
admin-only, because the brief's constraint presumed a role difference the repo
did not have.

## m11 — rename the npm scope
A mechanical migration, measured. Baseline fix-m10, layers 1, 1b, 1c, 2 and 3 in
place, 145 files modified and none new, 493 tests before and after. The first
measurement with no design decision in it: rename the scope from @harness-sample
to @support-desk after the repo itself was renamed, 195 occurrences across 144
files plus the root package name. Nothing about what to build, only what to
replace. Audited blind through the packaged skill before the diff was read.

The audit found nothing, which is a first in eleven measurements. 227 insertions
against 227 deletions — every line replaced, none added or removed, which is the
signature a mechanical migration should leave — and typecheck, lint at 5
warnings, 493 tests and 585 modules all match the baseline exactly. There was no
gap to fill differently because there was no gap: every occurrence had one
correct replacement and the compiler could see all of them.

The one finding came from neither audit. The agent doing the rename regenerated
the lockfile, saw the boundary count move by one, chased it, and reported that
.dependency-cruiser.cjs walks apps/api/dist/, so some of the modules it counts
are stale build artefacts — 157 of the 585, by that report. harness-06 found the
extent was wrong: there are four dist/ directories, not one, and excluding all
four took the count to 308 modules and 778 dependencies. 277 of the 585 were
build output, and the check reported the same number before and after a rename
that did not reach nearly half of them (m11-1).

---

# What the measurements with a harness show

The rules hold, including under a change that rewrote every surface in the
product. Alert was never switched off with className, Checkbox was never
hand-built, no glyph was reached for — the three things layer 1b closed stayed
closed through m09.

The findings moved outward rather than away. What the first six recorded was the
same gap filled differently every time; what these record is a gap that no
primitive owns and no rule describes, one scale further out than the one layer 1
covered. m07-2, m08-1, m08-2 and m08-3 are all that shape, and m09-1 is what it
costs when the scale moves: thirteen lines, on a repo of seven screens, for a
gap named two measurements earlier and closed only in harness-05.

Two findings are about the harness ageing rather than about the work.
no-raw-type-classes was scoped to apps/web/src/features when that was the whole
of the app, and app/ has since grown to hold AppLayout, AppRoutes and the auth
routes; the rule reports the three violations it can see and is blind to the
rest (m09-5). Widened to apps/web/src in harness-05, where the violations it
surfaced were migrated in the same change. And a rule that fires and is ignored
is a rule that only reports (m07-3).

The layer-1 claim is now a number rather than an argument. It is not zero, and
the reason it is not zero is written down.

m10 prices the prompt rather than a layer. A detailed brief, measured the same
way as the one-line tasks, halved the finding count and changed none of its
kind: three of four are prose that describes the work as intended rather than as
built. A brief constrains what gets built and cannot reach what gets written
about it, and the worst instance is m10-1, where the false claim is in AGENTS.md
— the file every future task reads first — and repeated in two more until
repetition made it read as verified. That is the one category no layer in this
harness has ever closed.

m11 prices the decision count. It touched 145 files against m10's 30 and cost
$1.38 against $17.56 — five times the files, thirteen times cheaper — because
m10's brief left seven design questions open by name and m11 left none. That is
the third correction to the same note: harness-05 blamed batching, m10 moved it
to breadth of work, and m11 moves it to the number of open decisions.

And one about the method rather than the work. The blind auditor reads a diff,
so it cannot see a file that correctly did not change: apps/api/dist/ still
carries the old scope precisely because the rename was right to leave it alone,
and the finding that mattered was outside the auditor's frame by construction.

---

# The three scaffolds

Not measurements — what separates a scaffold from a measurement is in
METHOD.md. Recorded because the findings are useful, and because each one
leaves the repo in a state chosen on purpose.

## The reports section
Baseline baseline-m04, 12 files modified and ~30 new, 143 tests up from 85. Ten
findings.

The feature code uses every primitive that exists; what got rebuilt by hand is
the design system's own chrome — CardFooter reassembled class for class in
ReportToolbar (rep-1), and BarChart reinventing the loading and empty state that
TableBody provides and that it imports two hundred lines lower (rep-5). rep-3 is
the padding count: eight distinct values in the repo, every one of them on the
spacing scale, so no lint rule fires and none ever will.

Two findings are against the design system README itself. It claimed a Tailwind
palette reset that had been removed months earlier, and an agent added five
sections to the file without noticing (rep-7). And it described three components
sharing one loading-state approach when all three had implemented it privately
(rep-8). Those two are where the doc-freshness gap in root cause 3 comes from.

Left deliberately: Alert now existed and was correct, and the two hand-rolled
banners in TicketListPage and TicketsToolbar stayed in place, unmarked. Nothing
said which was preferred. That was for the measurements to answer.

## The customers directory
Baseline baseline-reports, ~100 source files. Nine findings. Built to give the
repo a second list pattern and a second detail pattern, so a later measurement
would have two legitimate answers to choose between rather than one.

cust-1 is the second instance of an explicit instruction overridden: the prompt
asked for a multi-select plan filter and got a plain Select. With m05's union
deletion that made it a pattern rather than a one-off — and it removed the one
control that would have forced a genuinely new selected state into the design
system.

Drawer was written by copying Modal: same effect body, same Escape handler, same
focus restore, same class strings, with roughly fifteen lines of genuine
difference (cust-2). The copy included the defect — neither traps focus, while
both set aria-modal="true" — and the README documents that incomplete contract
as if it were complete (cust-3).

Left deliberately: two list patterns, two detail patterns, two error-display
approaches, three empty-state implementations, four selected-state
implementations, and no document saying which applies when. m06 is what that
ambiguity was for.

## Authentication
Baseline harness-01b, layers 1, 1b, 2 and 3 in place, 23 files modified and 13
new, 406 tests. Login and register, an httpOnly session cookie, every existing
route protected. It replaced the Settings page's role switcher, which existed
only because the server had no authentication to enforce, and took the Settings
page with it.

auth-1 is the most consequential single defect in the record. Signing in ran
`queryClient.clear()` and then `setQueryData`, which are two different Query
objects under one key — so the cache is correct and every mounted consumer is
still subscribed to the destroyed instance. RoleProvider sits above
BrowserRouter, so the post-sign-in navigate() never re-renders it. One mechanism,
two opposite symptoms: registering while signed in as an admin badged the new
agent account "Admin", and signing in as an admin from signed out badged
"Agent", because `?? 'agent'` filled the gap. 401 tests passed, because every
test rendering RoleProvider passed it an `initialRole` and disabled the session
query outright.

Two layers can be watched working here for the first time. Every new form uses
SubmitEventHandler rather than the deprecated FormEvent that spread through four
files in m04 unseen (layer 2), and AuthCard renders Alert's band variant rather
than switching Alert off with className (layer 1b). The four old FormEvent files
still carry it, because the rule ships at warn — the same shape as m07-3.

The debugging is recorded separately, because the diagnosis was better than the
fix: observer counts measured rather than inferred, the two further call sites
found by fixing the first two and re-measuring, and the new tests verified to
fail against the original code. The prompt that produced that is kept as
scaffolds/scaffold-auth-debug-prompt.txt, along with the agent's own account of which line
in it did the work.

---

# Root causes

1. The design system has primitives but no composites and no typography layer,
   so the agent hand-assembles what it cannot import.
   Addressed by harness layer 1 — see harness/harness-01-primitives.md, which also
   carries layer 1b, and harness/harness-01c-token-structure.md.
   Measured by m09: a full visual identity change reached nine feature files and
   thirteen lines, every one of them the single shape layer 1 does not cover.
2. Decision rules were never written down: button weight, card padding, when to
   extract a component, how to render a union.
   Status: written in harness/harness-03-agents-md.md, not enforced. All four are now
   stated. Card padding and extraction sit under Conventions, where a primitive
   also backs them; button weight sits under Product decisions, where nothing
   does. Card padding and button variant survived layers 1 and 2 for the reason
   recorded there — every value is already on the scale, so no rule can catch
   it, and a lint pass written to look for exactly this kind of drift fired zero
   times on padding.
   Prose is probabilistic compliance. The prediction recorded there was that
   the Conventions half would hold and the Product decisions half would get
   guessed. Both halves held on m07 and on m08. What those two measurements
   found instead is not a broken rule but a rule that does not reach: AGENTS.md
   is written at screen scale, and a strip above a list is not a screen (m08-3),
   which m09-1 priced at thirteen lines across nine files. Closed in
   harness-05: Toolbar exists and the strips are migrated. One `px-3 py-2`
   survives in feature code and it is not a strip — StateMessage having its
   padding overridden through className, which is the third instance of a
   primitive that cannot express the shape asked of it, after m07 and m08.
3. There are no sensors for any of it — no complexity rule, no a11y check, no
   duplication detection, nothing checking that a document still describes the
   code it governs (rep-7, rep-8), and since m05, no visual regression sensor
   either.
   Partly addressed by harness layer 2 — see harness/harness-02-guides.md. It adds an
   ESLint plugin with three design-system rules, size and complexity
   thresholds, deprecation detection and package boundary checks, and found two
   deprecated call sites that had been invisible for months. Still open: a11y,
   visual regression, and doc freshness — rep-7 and rep-8 are carried into
   layer 2's file, which records why no syntactic rule can close them.
   The strongest evidence is harness-01c: removing Tailwind's radius namespace
   took `rounded-full` with it, Badge and Avatar kept the class and lost their
   shape, and every check passed. The class name was still in the string, so
   the tests were satisfied; typecheck cannot see CSS; a lint rule would have
   been hard to write, because `rounded-full` is not wrong, it just stopped
   resolving. Two primitives, every screen, and the only thing that noticed was
   a person looking at a screenshot. m09 is the second instance: five status
   tints derived from a three-member colour family converge into each other, so
   three badges on the customers list are indistinguishable, and typecheck, lint
   and 407 tests are green. Redrawn in harness-05, by looking at them.
   m09 adds two more sensor gaps. Nothing checks that a rule's scope still
   matches the tree it governs — no-raw-type-classes was blind to
   apps/web/src/app from the moment that directory appeared until harness-05
   widened it by hand and migrated what that surfaced (m09-5). The rule now
   reports zero; the gap is that nothing except a person noticed. And nothing
   distinguishes a reported change that landed from one that did not: a
   two-line token edit shows in neither the diff stat nor on screen, and the
   only thing that caught it was reading the file, three times (m09-3).
   harness-05 shows the same gap from the other side. Given an acceptance
   criterion that could only be met by looking — five badges side by side are
   five colours — the agent built the sensor itself: an SVG swatch, before and
   after, each group rendered on both surface colours. The capability had been
   there since m01. What was missing every other time was a task that required
   looking, which is why the harness-01c radius break had nothing watching it.
   A sensor that appears when it is asked for is a check, not a sensor.
   m10 puts doc freshness at its worst: the claim that is false is a rule in
   AGENTS.md, repeated in two source files and checked by none of them, and the
   edit it promises is safe is the one that opens a silent authorization hole.
   The census of prose contradicting the code beside it now stands at seventeen
   instances across the record, re-taken in observations.md. Nothing closes it —
   every other census in this record ends in a primitive, and there is no
   primitive to write for a comment that does not check itself.
   m11 is the same gap from the other side. m09-5 was a rule whose scope had
   stopped covering the tree it governs; m11-1 is a boundary check that covers a
   tree it should not — depcruise walked four dist/ directories, so 277 of the
   585 modules it counted were build output, including apps/web's minified Vite
   bundle, cruised for boundary violations. Closed in harness-06, which found
   the extent as well: the report that surfaced it named one directory and 157
   modules, and excluding all four took the count to 308 modules and 778
   dependencies. So more than half of what every header from m09 on reported was
   a function of when a build last ran rather than of the source tree — same
   cause as m09-5, opposite direction: a scope chosen once against a structure
   that has since changed.

---

# The record

    README.md          what the measurements showed
    METHOD.md          how a measurement is run and what each header carries
    observations.md    properties of the method, not of one run

    measurements/
      m01-prompt.txt   m01-export-csv.md
      m02-prompt.txt   m02-saved-views.md
      m03-prompt.txt   m03-api-layer.md
      m04-prompt.txt   m04-assign-ticket.md
      m05-prompt.txt   m05-status-admin.md
      m06-prompt.txt   m06-customer-segments.md
      m07-prompt.txt   m07-customers-export.md
      m08-prompt.txt   m08-customers-bulk.md
      m09-prompt.txt   m09-redesign.md
      m10-prompt.txt   m10-global-search.md
      m11-prompt.txt   m11-scope-rename.md
      m12-prompt.txt   m12-workflow-engine.md

    scaffolds/
      scaffold-reports-findings.md      no prompt file survives
      scaffold-customers-prompt.txt     scaffold-customers-findings.md
      scaffold-auth-prompt.txt          scaffold-auth-findings.md
      scaffold-auth-debug-prompt.txt    the prompt that found auth-1
      scaffold-notes.md                 gaps belonging to the initial scaffold

    harness/
      harness-01-primitives.md          layers 1 and 1b: what they closed and what
                                        they did not
      harness-02-guides.md              layer 2: lint rules, thresholds, boundaries
      harness-03-agents-md.md           layer 3: the eight rules no tool can check
      harness-04-migrate-stragglers.md  not a layer: two migrations, one finding
      harness-01c-token-structure.md    extends layer 1: token structure, one
                                        silent visual break
      harness-05-toolbar-tints-scope.md not a layer: three m09 findings closed —
                                        Toolbar, the tints, the rule's scope
      harness-06-boundary-scope.md      not a layer: m11-1 closed — four dist/
                                        directories excluded, 585 modules to 308

The reports scaffold has no prompt file. Every other entry pairs its findings
with the prompt that produced them. The auth scaffold carries two: the
debugging that found auth-1 was a separate run, and its shape is worth keeping
apart from the spec that produced the code.
