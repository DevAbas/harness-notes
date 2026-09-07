# Agent harness measurements

A measurement experiment on one repository. I write the task, an agent builds
it, and I measure what it got wrong and which mechanism should have caught it.
Fourteen measurements, three scaffolds and nine pieces of harness work, all on a
support-ticket admin panel with its own design system.

The question: as a codebase grows, which mechanisms actually constrain a coding
agent, and which only appear to.

I expected the design system rules to break. Almost none of them did — six were
broken across the first six measurements and two scaffolds. What broke instead
was everything nobody had written down: five hand-built confirm dialogs, five
hand-written selected states, six hand-assembled page titles, the same gap
filled again and filled differently. The harness closed those kinds, and what
replaced them is prose — a comment, a docblock or a rule in AGENTS.md describing
a property the code beside it does not have. That census stands at twenty-four
instances, and no layer here has closed it.

         what the task was      prompt style       layers         findings

    m01  CSV export, tickets    one line           —              4
    m02  Saved views            short paragraph    —              10
    m03  HTTP API layer         detailed spec      —              9
    m04  Assign a ticket        one line           —              5
    m05  Status/priority admin  one line + 2 Qs    —              5
    m06  Customer segments      one line           —              7
    m07  CSV export, customers  one line           1 2 3          4
    m08  Bulk actions           one line + 1 Q     1 2 3          5
    m09  New visual identity    tokens + 4 rounds  1 1b 1c 2 3    5
    m10  Global search          detailed brief     1 1b 1c 2 3    4
    m11  Rename the npm scope   mechanical         1 1b 1c 2 3    1
    m12  Status as a workflow   detailed brief     1 1b 1c 2 3    5
    m13  Unify saved views      detailed brief     1 1b 1c 2 3    4
    m14  Plan catalogue admin   one line           1 1b 1c 2 3 4  7

The counts are not comparable down the column. The protocol changed while the
experiment ran — the blind audit arrived around m08, the packaged skill after
it, and at m14 the sensors began running inside the agent's own loop — so a
count measures the run and the method together. Three rows do not match a plain
read of their files. m08 was audited a second time, blind, and that pass found
eight against my five. m14's seventh is the only finding in this record that a
layer produced rather than a person. And m03 reads nine where its file numbers
ten: m03-10 is a synthesis of the nine above it rather than a tenth finding, and
it carries a Layer: line because the file numbers every section that way.

    METHOD.md        how a measurement is run, and what each header carries
    README.md        what each one showed — this file, from here down
    observations.md  the patterns that recur across measurements
    measurements/    the detail behind any single row

The subject repository is separate. Every measurement is a branch that was never
merged, so each defect stays where it was found, and the tags mark the harness
state each run happened in.

---

# Failures log

Repo: a support tickets admin panel. React, TypeScript, Tailwind, and its own
design system with a README that spells out the rules.

At the start nothing enforced those rules. There was no AGENTS.md, no CLAUDE.md
and no lint rule for the design system. The rules were written down and that was
all.

Question being measured: does the agent follow rules that are written but not
enforced?

That was the starting state, and the first six measurements answer it. The repo
has since grown a reports section, a customers directory, authentication, a
global search and a plan catalogue. It has also grown a harness in four layers —
primitives, with a typography and token layer extending them; lint rules; an
AGENTS.md; and sensors — and every measurement after m06 tests what a layer
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

All 222 tests pass at m06. Typecheck and lint are clean. Every finding above
passes all three.

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

## m12 — ticket status as a workflow
A detailed brief, measured. Baseline harness-06, layers 1, 1b, 1c, 2 and 3 in
place, 22 files modified, 14 new and one deleted, 534 tests up from 493. The
second measurement run with a detailed brief, and the first to change behaviour
that already worked rather than add a surface beside it: status went from a
value any screen could set to a position with named moves out of it. Audited
blind through the packaged skill after the commit, which is m11's protocol gap
closed, and before the diff was read.

Five findings. Both guides claim a fifth status needs no screen change, which is
what the brief asked for and what TicketStatusBadge's Record keyed by the status
union makes a compile error (m12-1). The moves card's empty state says there is
nowhere to take the ticket, and the one case it ever renders in is an agent
looking at a closed ticket, where the truth is that they may not reopen it
(m12-2). A bulk move report stays pinned above a table it no longer describes
(m12-3), a refused move leaves the ticket detail cache stale with no refetch
coming to correct it (m12-4), and a docblock cites a hook this change deleted
(m12-5).

m12-3 is a shape the record has not had before. Not a gap: new work diverging
from existing work that had already answered the same question, in writing, in a
docblock on the screen it was written for, in a file this change never opened.

All five closed on fix/m12-findings, with two tests added and verified to fail
against the unfixed code.

## m13 — one saved-view mechanism
A detailed brief, measured. Baseline fix-m12, layers 1, 1b, 1c, 2 and 3 in
place, 5 files modified, 3 new and 3 deleted, 561 tests up from 535. Audited
blind through the packaged skill after the commit and before the diff was read.

The brief's premise was false. It described two saved-view mechanisms where main
has one: m06's customer segments were never merged, and I read that note as if
it described main. The agent found this before starting, said so, and
reformulated the task correctly. What it did not do is record that anywhere in
the artifact, so the work ships as a unification with comments describing a
history the repository does not have (m13-1).

Four findings, and that is the first. The other three: a comparison that
disagrees with its own storage round trip about an absent key, unreachable from
today's scopes and guaranteed to fire on the first scope with an optional filter
(m13-2); the one test for the customer scope's distinctive behaviour, which
cannot fail (m13-3); and a hand-rolled guard where the shared contract exports
the schema (m13-4).

The browser pass produced no finding for the third measurement in a row, which
is not the same as producing nothing. The migration is the one guarantee no test
covers, and the browser is the only place it can be checked.

All four closed on fix/m13-findings, with a new test for m13-2 verified to fail
against the unfixed code and m13-3's reseeded to fail when the reorder is
removed.

## m14 — an admin screen for the plan catalogue
A one-line task, measured. Baseline harness-08, layers 1, 1b, 1c, 2, 3 and 4 in
place, 14 files modified and 16 new, 814 tests up from 757 and one of them
failing at the commit. A deliberate re-run
of m05 on a different union, and the first measurement with the sensors running
inside the agent's own loop. Audited blind through the packaged skill after the
commit and before the diff was read.

Six findings, all six from the audit, all six verified against the code before
being written down, and none of them wrong — a first in this record. Half of
every ladder refusal points the admin at a plan the dialog is not showing, and
the rule's docblock argues for exactly the property the sentence does not have
(m14-1). Two comments say the dialog tracks the live row, above four useState
initialisers that capture it at mount (m14-2). A store method with no caller and
a schema exported to nobody (m14-3). Plans is the second admin-only screen and
RequireRole.test.tsx is not in the diff (m14-4). A bounds docblock cites a `max`
on a field that is a text input (m14-5). And the bulk customer operations leave
the catalogue's count stale for the app's stale window, which no test can open
because the test client sets staleTime: 0 (m14-6).

A seventh came from the harness rather than from either pass. planKeys.ts is
sessionKeys.ts at 100% share and nothing recorded the pair, so lint:duplication
was red at the commit and the check's own test failed with it (m14-7). It is the
first finding in this record that a layer produced, and the first measurement
committed red — the mechanism is in observations.md, and the protocol rule it
produced is in METHOD.md.

None of the six was caught by any layer, and the re-run is what makes that
legible. Two of m05's five were layer-1 gaps and are impossible now — Icon and
Heading exist and the lint rules are at error. Two were visual, and the shape
they lived in was not built, because the editor is a modal rather than an
editable row. The union deletion m05 recorded did not recur: the plan's name is
not editable, because it is the identity a customer row stores. So the harness
closed the kinds m05 found, and what is left sits outside all four layers. Four
of the six are prose describing a property the code beside it does not have, one
is an assertion nobody wrote, and one is a defect the test client's own
staleTime puts out of reach — the m13 fixture shape at global scale.

What was not asked for is the other half of the measurement. The prompt said see
and edit; what arrived was 2098 lines over 30 files, a customer count per plan
with a new cross-feature join, and a monotonic price and seat ladder that
permanently forbids a promotional inversion through the product. Same category
as m10's reports gate, and wider: there a screen was closed, here a rule was
invented.

Three of the seven closed on fix/m14-findings, with the ledger entry m14-7 asked
for. Removing the unused schema falsified the docblock above the shape it was
spread from, which is the census arriving by a new route: introduced by the fix
for another finding, and caught in the same change. Left open: m14-2, m14-4 and
m14-6.

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
to breadth of work, and m11 moves it to the number of open decisions. Corrected
twice more in observations.md: m12 and m13 both left five decisions open by name
and cost $16.32 against $6.34, so the variable is what had to be invented rather
than what was left open.

And one about the method rather than the work. The blind auditor reads a diff,
so it cannot see a file that correctly did not change: apps/api/dist/ still
carries the old scope precisely because the rename was right to leave it alone,
and the finding that mattered was outside the auditor's frame by construction.

The intersection between the two audits went to zero for the first time. m08 and
m10 each converged on four findings; m12 converged on none. Every finding was in
code, and the browser pass — which produced findings no diff reader could reach
on m05 and m09 — produced nothing here, and the one thing it did produce was
wrong. The intersection is evidence when it happens and is not evidence of
anything when it does not. m13 took the browser pass's silence to a third
measurement running; on m14 it confirmed a finding the audit had already made
rather than producing one of its own.

And the census keeps running. m12-1 is AGENTS.md again, in a change whose brief
made reading it mandatory, and the claim was one the brief itself had asked for.
A prompt that names a success criterion gets it restated as achieved.

m13 prices the brief's premise. It described two saved-view mechanisms where
main has one; the agent found that before starting and reformulated the task
correctly, and the correction exists only in a chat log. The work ships as a
unification whose comments explain a history the repository does not have
(m13-1). A brief that is wrong about the repo gets corrected in conversation and
committed as though it never was.

m14 is the first measurement a layer found something in. lint:duplication named
planKeys.ts and sessionKeys.ts, both files and the shape they share (m14-7);
every finding in this record before it came from a person or from an audit. It
arrived after the commit, because it is the one check whose input is what git
tracks rather than the working tree, so the measurement went in red with neither
pass looking. The other six sit outside all four layers, and being a re-run of
m05 is what makes that legible: the kinds m05 found are closed, and what is left
is mostly prose describing a property the code beside it does not have — the
category no layer here has closed.

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
ReportToolbar (rep-1), and BarChart reinventing what TableBody provides and
imports two hundred lines lower (rep-5, above). rep-3 is the padding count:
eight distinct values at this scaffold, every one of them on the spacing scale,
so no lint rule fires and none ever will.

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
still carried it, because the rule ships at warn — the same shape as m07-3. They
were migrated in harness-04, which found the count was six rather than four,
because the config comment naming them was itself stale.

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
   The census of prose contradicting the code beside it now stands at
   twenty-four instances across the record, re-taken in observations.md after
   m14. Nothing closes it — every other census in this record ends in a
   primitive, and there is no primitive to write for a comment that does not
   check itself.
   m11 is the same gap from the other side. m09-5 was a rule whose scope had
   stopped covering the tree it governs; m11-1 is a boundary check that covers a
   tree it should not — depcruise walked four dist/ directories, so just under
   half of the modules every header from m09 on reported were build output,
   including apps/web's minified Vite bundle, cruised for boundary violations.
   Closed in harness-06, which found the extent as well; the numbers are in the
   m11 entry above. Same cause as m09-5, opposite direction: a scope chosen once
   against a structure that has since changed.
   Answered by harness layer 4 — see harness/harness-07-sensors.md. First entry
   in the record that answers this item's opening sentence rather than
   qualifying it. Five sensors: mutation testing over packages/shared and the
   two non-JSX files in packages/ui, 84.39% killed at harness-07 against a
   threshold of 82 taken from that run; two a11y lint rules plus an axe sweep
   over the DOM the tests already render; structural duplication scored by
   share-of-file, because absolute thresholds cannot separate a copy from a
   legitimate parallel here; two rules resolving the paths and symbols that
   documents cite; and a gate. What each one found on its first run, and what
   each one cannot see, is in that file — including the gate's own catch, that
   eslint.config.js had been promoting rules on CI=true for four commits against
   a CI that never existed, so the strict tier had never run.
   What it does not close is stated by mechanism rather than by intent. The
   visual half stays open exactly as written above: jsdom has no layout, so
   contrast, target size and focus visibility are unchecked here and nowhere
   else, and neither the harness-01c radius break nor the m09 tints would be
   caught by anything in layer 4. Fixtures stay open because mutation testing
   mutates source — a test whose data cannot distinguish two branches stays
   green, and DateRangeField.test.tsx declares two presets ending on the same
   date, so half of isSameRange can be deleted with 703 tests passing. And most
   of the prose category stays open, because the two doc rules resolve
   citations and the census, twenty-four after m14, is mostly claims about
   behaviour, which cite nothing to resolve against. The instance harness-07
   records is the shape of the rest: support-desk/README.md opens by saying the
   repo has no AGENTS.md and no lint rule, three times false, naming no path and
   no symbol, and so invisible to both rules. It has since been rewritten into the
   past tense on that branch, by hand — the sensor could not see it and a
   person fixed it in the same change.
   Layer 4 shipped five sensors and none of them reads the artefact that
   actually ships. Both vitest projects run with `css: false`, so no test in
   this repository has ever seen a stylesheet, and CI ran typecheck, lint,
   boundaries, duplication and tests without ever running the build. A sixth
   was needed, and the defect that proved it came from the commit that added
   the five: a bare `reports` in .gitignore, put there for Stryker's output,
   also matched apps/web/src/features/reports, and Tailwind's scanner honours
   .gitignore, so the whole Reports feature was invisible to it — seven class
   sites producing no CSS, and not one of the five sensors that arrived in that
   same commit could see what it broke. See
   harness/harness-08-class-resolution.md.
   What the sixth closes is the harness-01c shape, by mechanism rather than by
   a person looking at a screenshot: a class whose name is unchanged and whose
   meaning is gone emits no rule, and the built stylesheet says so.
   `--radius-element` removed from tokens.css was named at 14 sites across 12
   files, with typecheck clean, lint at 5 warnings and all 757 tests green —
   including the eight in Alert.test.tsx, which asserts that class three times.
   The paragraph above was written of the five, and its first half no longer
   holds: the radius break is the case the sixth reads for.
   What stays open is the half that resolves. `--radius-element` changed to
   `2rem` rather than removed emits a rule, ships, and is invisible to this,
   which is the m09 tints exactly — every class resolving, three badges
   indistinguishable. That is still the visual sensor the record keeps naming,
   and jsdom's missing layout still leaves contrast, target size and focus
   visibility unchecked here and nowhere else.

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
      m13-prompt.txt   m13-saved-views-unified.md
      m14-prompt.txt   m14-customer-plans-admin.md

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
      harness-03-agents-md.md           layer 3: the eight rules no tool can
                                        check, as written at harness-03
      harness-04-migrate-stragglers.md  not a layer: two migrations, one finding
      harness-01c-token-structure.md    extends layer 1: token structure, one
                                        silent visual break
      harness-05-toolbar-tints-scope.md not a layer: three m09 findings closed —
                                        Toolbar, the tints, the rule's scope
      harness-06-boundary-scope.md      not a layer: m11-1 closed — four dist/
                                        directories excluded, 585 modules to 308
      harness-07-sensors.md             layer 4: five sensors, and what each of
                                        them cannot see
      harness-08-class-resolution.md    layer 4: a sixth sensor, over the built
                                        stylesheet the other five never read

The reports scaffold has no prompt file. Every other entry pairs its findings
with the prompt that produced them. The auth scaffold carries two: the
debugging that found auth-1 was a separate run, and its shape is worth keeping
apart from the spec that produced the code.
