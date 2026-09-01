# Measurement 10 — "Add a global search."

Task: see m10-prompt.txt
Branch: measure/10-global-search
Harness state: layers 1, 1b, 1c, 2 and 3 in place; layer 4 not built
Prompt style: detailed brief, measured
Baseline: fix-blind-audit
Codebase: ~7 screens, 585 modules, 429 tests before / 492 after
Files touched: 20 modified, 10 new
Lint: 5 warnings, unchanged
Duration: 30 minutes API, 1h 10m wall, $17.56

The first measurement run with a detailed brief. Every earlier detailed prompt
was treated as scaffold; this one states what is needed, what is already
decided, and what is deliberately left open, and is measured like the one-line
tasks.

Audited blind through the packaged skill before I read the diff.

## m10-1 AGENTS.md states an invariant the code satisfies for one screen
The change added a rule to the file every future agent reads first:

    Say who may reach a screen with `roles` on that table, and ask with
    `canReachNavigationTarget`. Gating a screen there gates the nav item, the
    route and every search result that lives on it, in one edit.

The same claim is repeated in packages/shared/src/navigation.ts and in
RequireRole.tsx: "A screen gated in NAVIGATION_TARGETS is gated everywhere at
once."

Only one of the four routes is wrapped. Adding `roles: ['admin']` to `customers`
would remove the nav item, remove the customer group from that role's search,
and leave /customers rendering in full for any agent who types the address —
the exact hole RequireRole was written to close for reports. The nav and the
search go quiet, so nothing looks wrong.

Worse than an ordinary gap, because it is written down as done. The one edit the
guide promises is safe is the one that opens a silent authorization hole.

Three files repeat the claim and none of them checks it. Repetition made it read
as verified.
Caught by: nothing — tests exercise only `reports`
Layer: rule broken, and a candidate for the lint plugin: a target with narrowed
roles must have a RequireRole route. That is exactly what AGENTS.md's own
closing section says to do with a rule prose cannot carry.

## m10-2 The previous search's results come back, selectable, on the next first keystroke
useGlobalSearch is disabled while the debounced term is empty and keeps previous
data. GlobalSearch branches on the live term rather than the settled one, so
clearing the field is a reset to the component and not to the cache.

Search something, press Escape, reopen, type one letter, and for 250ms the
palette shows the previous results. hasResults is true, so isLoading is false
and no "Searching…" appears; destinations is built from the same stale groups,
so Enter inside that window navigates to a result of the search already
dismissed.

The auditor drove a QueryObserver against the repo's own query-core to confirm
the three transitions.

And the comment above close() names the exact risk and misses it: "A palette
opened again is a new question, so the term goes with it: the old results would
otherwise be the first thing on screen." The term goes. The answer stays, hidden
behind the empty-term branch until a keystroke uncovers it.
Caught by: nothing. The test file gets within one line — it searches, closes,
reopens, asserts the field is empty, and stops before typing again.
Layer: sensor missing

## m10-3 The reports test file asserts the new rule and breaks it four lines later
One render in ReportsPage.test.tsx gained `role: 'admin'` with a comment saying
only an admin reaches this screen. The other render in the same file was left at
the default, which is `agent` — so a live test depicts an agent sitting on a
screen this change made unreachable.

It passes because the harness has two role sources that disagree: setup.ts signs
every test in as the seeded admin on the server, while the client provider
defaults to agent. The screen renders, the now-adminOnly endpoints answer, and
the assertion holds.

The two new test files do set both ends and explain why, which makes the one
call site that does not stand out.
Caught by: nothing — it passes
Layer: sensor missing

## m10-4 Input.labelHidden documents itself as "the same as Checkbox's" while doing the opposite
Input's new docblock says it is the same prop Checkbox has, for the same reason,
and that an aria-label in its place would be a second way of naming a control.
The ui README repeats it.

Checkbox names its hidden-label control with exactly that aria-label, and
returns a bare input with no label element at all. Input renders an sr-only
label instead. Both are correct and they are not the same mechanism, and the new
prose argues against the one the sibling it cites actually uses.

No behavioural difference. A reader who trusts the README will believe Checkbox
keeps a hidden label element.
Caught by: nothing
Layer: not harness — documentation

## What was done well

The design system boundary held under pressure. The brief said to say so rather
than assembling a missing primitive in the feature; the work built CommandPalette
in packages/ui with its own test file, documented it in the README, and left
GlobalSearch holding only data and destinations.

The one place it writes a class the system owns — the active row's fill — is
disclosed in the component and in the README, and named as the second instance
of a hole: `selected` should be a utility beside `interactive` and `focus-ring`.
That is the escalation this repo asks for rather than a quiet exception.

Dialog gained a `header` seam that replaces the chrome and not the contract:
title still names the dialog, rendered sr-only, and aria-labelledby still points
at it. A dialog with its own header is named the same way one with a CardHeader
is, and no shape can be drawn without a name.

The combobox implementation is correct rather than plausible: focus never leaves
the field, arrows move aria-activedescendant, options are div role="option"
because a Button in a listbox is not an option, and the active option is derived
from a remembered id rather than an index so replaced results fall back to the
top.

Exhaustiveness is structural in three places — SCREEN_BY_RESULT_TYPE,
GROUP_LABELS and the builders record are all keyed by SearchResultType, so a
fourth result type is a compile error rather than an ungated group.

The ticket store's search was kept separate from list, with the reason: widening
list to read an assignee would silently change what the ticket screen's own
filter means. A test proves the divergence.

close() is stabilised with useCallback because Dialog moves focus into its panel
whenever onClose changes identity — a new closure each render would take focus
off the field each time the debounce settled, mid-word.

## What was not asked for

Reports was made admin-only. Three endpoints gained adminOnly, RequireRole was
written, the nav filters, and both READMEs were rewritten to justify it.

The brief's constraint — whatever an agent cannot reach through the nav should
not appear in their search — presumed a role difference that did not exist. On
main every screen was reachable by both roles. So a difference had to be
invented to satisfy the constraint and to write the agent-versus-admin test the
brief also required.

The reasoning given is coherent. It is still a product decision that revokes a
capability from every agent account, arrived at as a side effect of adding a
search box.

Also: the customer drawer's state moved from useState into the URL. Necessary,
since a search result has to land on a customer and a drawer is not a route, and
it preserves the original decision by replacing rather than pushing. It changes
an existing screen's behaviour — the drawer now closes when you click Customers
in the nav.

## What this measurement shows

Eight findings on m08's one-line task; four here on a detailed brief. So the
brief cut the count in half.

It did not cut the kind. Three of the four are the same shape: confident prose
that the code beside it does not do. AGENTS.md claims a gate that exists once.
close() names the stale-results risk and closes half of it. Input's docblock
describes a sibling it does not match.

That is the pattern the record has been accumulating since m03, and a better
brief does not touch it — because the brief constrains what gets built, and this
is about what gets written about what was built.

One correction to an earlier note. harness-05 recorded that combining three
unrelated pieces of work in one brief cost time, and inferred the batching was
the cause. This was one piece of work and cost longer — 30 minutes against 24,
and $17.56 against $10.93. The variable is the breadth of the work, not the
shape of the brief.
