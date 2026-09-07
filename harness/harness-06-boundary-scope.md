    Baseline: measure-11
    Tag: harness-06
    Closes: m11-1
    Boundaries: 585 modules / 1170 dependencies before, 308 / 778 after

Not a layer. One line of configuration, and the note is worth writing because of
what the line uncovered rather than because of the change.

### What was wrong

`.dependency-cruiser.cjs` had `doNotFollow: { path: 'node_modules' }` and nothing
else, so `depcruise apps packages` walked every `dist/` directory under those two
trees. Added:

    exclude: { path: '(^|/)dist/' },

### The count was worse than reported

m11-1 named `apps/api/dist` and put the artefact share at 157 of 585 modules.
There are four `dist/` directories — `apps/api`, `apps/web`, `packages/ui` and
`packages/shared` — and excluding them all took the count from 585 to 308 and the
dependency count from 1170 to 778.

So 277 of the 585 were build output. `apps/web/dist` is Vite's bundle, which
means minified JavaScript was being counted as modules and cruised for boundary
violations.

**Corrected in the README pass after m14.** That sentence read "277 of the 585,
slightly more than half" from harness-06 until then, in the same paragraph as the
two numbers that disprove it: 277 of 585 is 47.4%, which is just under half. It
was restated in that form in observations.md and in README.md's third root cause,
and the one place that re-derived it rather than copying it — README.md's m11
entry, at "nearly half" — was right for three measurements while three other
places were wrong. Counted as an instance of the count-is-a-search entry in
observations.md.

The finding was right and its extent was wrong, in the same direction as the
audit-report entry in observations.md. This time it was the agent doing the work
rather than an auditor, and the correction came from `find . -type d -name dist`
rather than from reading a report more carefully.

### It was verified rather than assumed

A boundary check that reports zero violations reads the same whether the tree is
clean or the check stopped looking. `harness-01c` is the instance that made this
worth checking: removing Tailwind's radius namespace broke Badge and Avatar,
every check passed, and only a person looking at a screenshot noticed.

Two things were checked before the change was kept.

`packages/ui/package.json` resolves through `exports: { ".": "./src/index.ts" }`,
not through `dist/`, so excluding the directory cannot make a workspace import
stop resolving. The cruise output confirms it: `Alert.tsx →
packages/shared/src/index.ts` is still an edge, so the paths the three rules are
keyed on are still visible.

Then a deliberate violation — an export from `packages/ui/src/index.ts` reaching
into `apps/web/src/features/tickets/TicketListPage` — and the rule fired:
`ui-not-to-app-or-feature`, correct rule name, correct pair of files. Reverted
with `git checkout`.

### What this changes for the record

The module count in every header from m09 to m11 was a function of when a build
last ran, not of the source tree. 585 in one file and 585 in another do not mean
the same thing, and none of them meant what the header implied.

From m12 the numbers are 308 modules and 778 dependencies, and they measure the
source.
