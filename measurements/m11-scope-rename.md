# Measurement 11 — "Rename the npm scope from @harness-sample to @support-desk."

Task: see m11-prompt.txt
Branch: measure/11-scope-rename
Harness state: layers 1, 1b, 1c, 2 and 3 in place; layer 4 not built
Prompt style: mechanical migration, measured
Baseline: fix-m10
Codebase: ~7 screens, 585 modules, 493 tests before and after
Files touched: 145 modified, none new
Lint: 5 warnings, unchanged
Duration: 4 minutes API, 8m 34s wall, $1.38

The first measurement with no design decision in it. Rename the npm scope
from @harness-sample to @support-desk, after the repo itself was renamed.
195 occurrences across 144 files, plus the root package name. Nothing about
what to build, only what to replace.

Audited blind through the packaged skill before I read the diff.

## The audit found nothing

Eleven measurements, and this is the first with zero findings. 227 insertions
against 227 deletions — every line replaced, none added or removed, which is the
signature a mechanical migration should leave. Typecheck, lint at 5 warnings,
493 tests and 585 modules all match the baseline exactly.

The reason is the task. There was no gap to fill differently, because there was
no gap: every occurrence had one correct replacement and the compiler could see
all of them.

## m11-1 The boundary check counts build output
`.dependency-cruiser.cjs` has `doNotFollow: { path: 'node_modules' }` and nothing
else. `apps/api/dist/` sits under the `apps` path that `depcruise apps packages`
walks, so 157 of the 585 modules it reports are stale build artefacts.

The module count is therefore a function of when a build last ran, not of the
source tree. It appears in the header of every measurement from m09 on, and the
same number in two files does not mean the same thing.

The `dist/` files still carry the old scope, because the rename correctly left
build output alone. So the check reported 585 modules both before and after a
rename that did not reach a quarter of them.

Same family as m09-5, from the other side: there a rule's scope had stopped
covering the tree it governs, here it covers a tree it should not. Both are a
scope chosen once against a structure that has since changed.
Closed in harness-06, where the extent turned out to be wrong: there are four
`dist/` directories, not one, and excluding all of them took the count from 585
modules to 308 — so the artefact share was 277, not the 157 estimated here.
Caught by: nothing — reported by the agent doing the rename, as a side effect
Layer: rollout — one line, `exclude: { path: '(^|/)dist/' }`, deliberately not
fixed on this branch

## What was done well

The migration was complete and the report was specific: 195 occurrences, 144
files, named by category — package names, dependency entries, imports,
`eslint.config.js`, the dependency-cruiser comments, the CSS `@import`, the four
lint rule messages that name the scope in their reported text, and the tests
asserting that text.

Three things were deliberately left: the `eslint-plugin-harness` directory and
package basename, the `harness/` rule prefix, and `HARNESS_STRICT_LINT`. The
reason given is that these are the plugin's identity rather than the npm scope,
and it is right — the plugin is the harness, and the harness did not get renamed.

The lockfile was the one place judgement was needed and it was measured rather
than guessed. Deleting `package-lock.json` and `node_modules` and running a
clean install passed everything but moved the boundary count to 586: npm placed
`@vitejs/plugin-react` nested under both `apps/web` and `packages/ui` rather than
hoisted once. This was confirmed by cruising a clean worktree of HEAD and diffing
the module lists, not inferred. The lockfile was then restored and `npm install`
run against it, which rewrites the workspace entries without touching the
resolved tree — a 32-line diff, no version churn, count back to 585.

That is the standard the auth debugging set: observer counts measured rather than
inferred. Second instance.

## What this measurement shows

Cost tracks decisions, not size. m10 was 30 files and cost $17.56 over 30 minutes
of API time; m11 was 145 files and cost $1.38 over 4. Five times the files,
thirteen times cheaper.

harness-05 recorded that combining three pieces of work cost time and inferred
batching was the cause. m10 corrected that to breadth of work. m11 corrects it
again, and this is the sharper statement: the variable is the number of open
decisions. m10 left seven design questions to the agent. m11 left none.

And a finding about the method rather than the work. The blind auditor sees the
diff. `dist/` is invisible to it, because those files did not change — the whole
point is that they should have been considered and correctly were not. The most
valuable thing this measurement produced came from the agent doing the work,
noticing a number move, and chasing it.

An audit against a diff cannot find what is missing from the diff. That is a
different blind spot from the one recorded in observations.md, where the auditor
was right about a mechanism and wrong about its extent.

## A protocol gap

The prompt says do not commit, so `git diff main...measure/11-scope-rename` is
empty and the whole change is only visible as working-tree modifications. The
auditor tried the three-dot form first and found nothing before falling back.
Either the audit runs after the commit, or the audit prompt says to read the
working tree. Worth fixing before m12.
