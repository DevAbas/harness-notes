# Measurement 14 — "Add an admin screen where an admin can see and edit the
customer plans."

Task: see m14-prompt.txt
Branch: measure/14-customer-plans-admin
Harness state: layers 1, 1b, 1c, 2, 3 and 4 in place
Prompt style: one-line task, measured
Sensors: run by the agent in its own loop
Baseline: harness-08
Codebase: ~9 screens, 356 modules, 757 tests before / 814 after, 813 passing
Files touched: 14 modified, 16 new
Lint: 5 warnings, unchanged; lint:duplication red at the commit
Duration: 19m API, 40m wall, $18.62 — plus 15m and $8.30 for the audit
Header: taken before the commit, corrected after the fact — see m14-7

A deliberate re-run of m05 on a different union. m05 was the ticket status and
priority admin and it found five things. This is the plan catalogue, with all
four layers in place, and the sensors running inside the agent's own loop for the
first time in the record.

The tasks are not the same size, so the counts are not the comparison. The kinds
are.

## The seven findings

Six came from the blind audit, delegated to a fresh agent, and all six were
verified against the code before being written here. None was wrong — a first in
this record.

The seventh came from the harness, after the commit, and neither pass saw it.

### m14-1 Half of every ladder refusal points the admin at a plan the dialog is not showing

The rule builds its sentence from the pair of rungs and does not know which plan
is being edited. So "Lower Starter first, or price this one above it" is correct
when editing Pro and wrong when editing Starter: it tells the admin to undo the
field they just typed, and "this one" names a plan that is not on screen.

The docblock above it argues for exactly the property the sentence does not have.

Confirmed in the browser, by editing the lower rung upward and reading what came
back.

Caught by: nothing — the tests assert the sentence the rule produces, and the
rule produces it without ever being told which plan the dialog is editing
Layer: rule broken — the comment directly above states the property the code
does not have

### m14-2 Two comments say the dialog tracks the live row; it captures it at mount

Four `useState` initialisers and no `key`, so a refetch updates the title and the
customer count and nothing else.

The hazard the comment names — another tab re-pricing a plan, this dialog
re-submitting figures nobody can see — is exactly the three fields that do not
update.

Caught by: nothing
Layer: rule broken — the same shape as m14-1, twice in one file

### m14-3 A store method with no caller, and a schema exported to nobody

`PlanStore.get` has no caller. `customerPlanTermsSchema` is exported and consumed
nowhere.

`customerStore.ts`, three files away, states the rule: a store method with no
caller is a shape nobody has had to defend.

Caught by: nothing — an export counts as a use, so neither the lint tier nor any
of the six sensors reports it
Layer: sensor missing — no rule reports an exported symbol that nothing imports

### m14-4 Plans is the second admin-only screen and RequireRole.test.tsx is not in the diff

The file enumerates four assertions for Reports and none for Plans, while three
documents claim both screens are gated.

The wiring is correct today. Nothing holds it there.

Caught by: nothing — coverage reports the component as exercised, because Reports
exercises it, and mutation testing runs over packages/shared and two non-JSX
files in packages/ui
Layer: sensor missing — nothing can see an assertion that was not written

### m14-5 The bounds docblock cites a `max` that cannot exist

It says a bound tightened there tightens the control, citing a `maxLength` and a
`max`. The `maxLength` is real.

The `max` cannot be. Both number fields are text inputs with `inputMode`, argued
for in a comment twelve lines above, and `max` has no effect outside
`type="number"`.

Caught by: nothing — layer 4's two doc rules resolve paths and symbols, and an
attribute that is not there resolves nothing
Layer: not harness — a claim about what an attribute does cites nothing to check
it against

### m14-6 The bulk customer operations leave the catalogue's count stale, and no test can open the window

They invalidate only `customerKeys`, so the plan catalogue's customer count is
stale for the app's stale window after a bulk move. The server side is correct
and tested.

The house pattern is that no mutation crosses feature namespaces, so this does
not depart from it. What makes it worth recording is that the test client sets
`staleTime: 0`, so the window this defect needs never opens in any test.

Caught by: nothing
Layer: sensor missing — and the test client's own configuration is what puts it
out of reach, the m13 fixture shape one scale up

### m14-7 planKeys.ts is sessionKeys.ts, and the ledger did not know

Both files are a private `all` tuple under an object of thunks that spread it.
The detector scores the pair at 100% share, nothing in
`internal/duplication-check/src/index.js` records it, so `lint:duplication` exits
red and one test fails with it — `run.test.js` asserts `result.added` is empty,
which is the check itself rather than a test of it.

The measurement was committed red and I did not see it, because I took the
numbers before the commit. The detector's input is `git ls-files apps packages`:
195 files at main, 207 at the commit, and the twelve it gained include the file
that makes the pair. Every other sensor reads the working tree and answers the
same on either side of a commit. This one reads what is tracked, so there is a
state in between where it has nothing to say, and that is the state the header
was written from.

The audit could not see it either. The skill forbids the auditor from running
typecheck, lint, tests, boundaries, duplication or mutation, because "those
numbers are taken before the audit starts" — which is what keeps the two passes
independent, and is why a red check sat between them with nobody looking.

Caught by: lint:duplication, which named the pair, both files and the shape they
share — the only finding in this record that a layer produced
Layer: layer 4, working. What failed is the protocol around it

## What I checked myself

The browser pass. The role gate on all three surfaces — nav, route, server, both
GET and PATCH refused for an agent at 403. The ladder enforced server-side with a
409 naming both rungs. Schema bounds refusing a negative price. Price parsing
exact at 19.99.

The edit dialog is a modal rather than in-row editing, which is why m05's two
visual findings have no equivalent here.

One finding came out of it: m14-1, confirmed by editing the lower rung upward and
reading the sentence.

## What was done well

From the audit. The ladder is a table keyed by a union, adjacent pairs only with
the transitivity argument written down, and unlimited seats compared as the top
of the order rather than special-cased. The 400/404/409 split argued rather than
defaulted.

Reuse over rewriting: `CustomerPlanBadge` imported from the customers feature
rather than a fifth badge map, the respond helpers, `requireAdmin`, the key
factory shape, the layout from CustomersPage, the form pattern from
SavedViewNameDialog.

And the strongest idea in the diff: the plan's name is not editable, because it is
the identity a customer row stores, a saved view is written in, and an exported
CSV says.

## What was not asked for

The prompt said see and edit. What arrived was 2098 lines over 30 files, a shape
for the terms that the prompt never named, a customer count per plan with a new
cross-feature join, and a monotonic price and seat ladder that permanently forbids
a promotional inversion through the product.

That last one is a product decision made on my behalf, and it is the thing that
needs a person to sign it off separately from any defect.

Same category as m10's reports gate, and wider: there a screen was closed, here a
rule was invented.

## What this measurement shows

Six findings outside every layer, and one a sensor caught. That is the result,
and the second half of it is new.

The six are not a failure of the layers. Two of m05's five findings were layer-1
gaps — a glyph icon, a hand-assembled heading — and they are impossible now:
`Icon` and `Heading` exist and the lint rules are at error. Two more were visual,
and the dialog being a modal rather than in-row editing removed the shape they
lived in.
The union deletion m05 recorded did not recur; the plan's identity was recognised
and left alone.

So the harness prevented the kinds m05 found. What is left sits outside all four
layers.

Four of the six are prose describing a property the code beside it does not
have. The census has been running since m03 and no layer closes it. The two doc
rules resolve citations; these are claims about behaviour, which cite nothing.

m14-4 is an absent assertion, and nothing can see what was not written.

m14-6 is a defect the test client's own configuration makes unreachable, which is
the m13 fixture entry in observations.md at a different scale: there a fixture
ruled out the defect it guarded against, here a global test setting rules out a
whole class of them.

m14-7 is the other half, and the first finding in this record a layer produced.
It is also the first measurement committed red. Both of those are one sentence:
lint:duplication named the pair, both files and the shape they share, and the
report went unread because I took the numbers before the commit and the auditor
is forbidden from taking them again. The constraint that keeps the two passes
independent is what put this one outside both.

And the sensors ran inside the agent's loop for the first time, so the agent
corrected itself before finishing. What that removed does not appear anywhere in
this note, which is a limit on what the comparison to m05 can say. It is also
the one sensor that loop could not have satisfied: the duplication check reads
the tracked file list, which the commit changes after the loop has ended.

## What was fixed

Three findings closed on fix/m14-findings, off the measurement branch, plus the
ledger entry m14-7 asked for. Merged to main and tagged fix-m14. Three files.

m14-1: the deictic clause is gone rather than threaded through. The rule is asked
about a catalogue and not about an edit, so there is no edited plan at that layer
to hand it, and an optional parameter would have left the deixis wrong wherever
it was absent. Both remedies name both rungs — "Lower Starter's price, or raise
Pro's" — and the docblock that argued for naming the pair now carries the reason
the remedy names them too.

m14-3: `PlanStore.get` and `customerPlanTermsSchema` both removed, and the
store's docblock lost the paragraph arguing that a `get` which cannot miss is
worth having in the type.

Removing the schema falsified the docblock above `planTermsShape`. It argued the
shape was spread into three schemas rather than extended from one, so that
annotating the terms with their domain type did not cost the other two their
fields — and after the removal there are two, neither of them annotated. It was
rewritten to the reason that still holds: the edit body and the catalogue row
share three fields without being versions of each other, and extending would put
a field added to either onto the other. The alternative was a fresh false
docblock, introduced by the fix, in the file the fix was open in.

m14-5: the bounds docblock now describes the mechanism per field. Only the
description's bound is the control's own doing, a `maxLength` on the textarea.
Both figures are text inputs carrying an `inputMode`, argued for at the price
field, so their ceilings are branches in the dialog's `handleSubmit`, each naming
the bound in the message it shows.

m14-7: recorded as `parallel`, with the entry's argument checked rather than
asserted. The first draft said the other four factories carry enough keys of
their own to stay under the threshold, which is loose — `searchKeys` is two
thunks, the same as these. What separates them is that the other four take a
filter, a query or an id as a parameter, and these two are parameterless, so the
shared frame is the whole file.

814 tests, all passing. 5 warnings unchanged, boundaries clean at 356 modules,
class resolution clean, duplication at 13 over the line and 0 not recorded.

Three left open: the two comments claiming the dialog tracks the live row
(m14-2), the missing RequireRole assertions for /plans (m14-4), and the bulk
customer operations not invalidating planKeys (m14-6).

The fix pass wrote a census instance of its own and caught it in the same change.
Removing one schema left the docblock above the shape arguing for an arrangement
of three that had become an arrangement of two — the category this record has
counted since m03, arriving by a route no measurement has recorded before: not
written with the work, but introduced by the fix for a different finding, in a
file already open.
