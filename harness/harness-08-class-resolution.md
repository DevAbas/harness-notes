    Baseline: harness-07
    Tag: harness-08
    Layer: 4 — sensors, sixth
    Tests: 703 before / 757 after
    Lint: 5 warnings, unchanged
    Boundaries: 336 modules, clean
    Duplication: 12 pairs, 0 new

Layer 4 shipped five sensors and none of them reads the artefact that actually
ships. Both vitest projects run with `css: false`, so no test in this repository
has ever seen a stylesheet, and CI ran typecheck, lint, boundaries, duplication
and tests without ever running the build.

### Problem / Mechanism / Tool

    Problem:   A design token is deleted or renamed. Every class built from it
               keeps its name, keeps compiling, keeps passing its tests, and
               silently produces no CSS.
    Mechanism: Tailwind emits a rule only for a class it can build, so the built
               stylesheet already holds the answer. Read the artefact, not the
               source.
    Tool:      internal/class-resolution, `npm run lint:classes`. About 500 lines
               over the typescript compiler API and a brace scanner for the CSS.
               No new dependency.

### It found a live defect while it was being designed

`.gitignore:17` carried a bare `reports`, added for Stryker's output. An
unanchored pattern matches a directory of that name at any depth, so it also
matched `apps/web/src/features/reports`, and Tailwind's scanner honours
`.gitignore`. The whole Reports feature was invisible to it: `text-right` at four
sites, `tabular-nums` at two, `xl:grid-cols-4` at one, all producing no CSS.

It arrived in the commit that added the sensor layer, and none of the five
sensors in it could see what it broke. Fixed on this branch, all three patterns
anchored, with a comment saying why.

This is also why the file list comes from `git ls-files` rather than a filesystem
walk: a walk honouring `.gitignore` would go blind exactly where Tailwind is
blind.

### Both halves were seen to fire

`--radius-element` removed from tokens.css, rebuilt: the check named 14 sites
across 12 files, columns included, against a prediction written before the run.
With the token gone, typecheck was clean, lint was 5 warnings, and all 757 tests
passed — including all eight in Alert.test.tsx, which asserts that class three
times. Only the new check fired.

Token restored, rebuilt: nothing reported.

The second shape was proved live too. `--focus-ring-color` deleted:
`.focus-ring:focus-visible` stays in the stylesheet, so the no-rule check stays
silent, and the dangling-property check names both call sites. `var(--x, 0)` is
never reported — a fallback is how a stylesheet says a missing property is
intended.

### The measurement that decided the design

Extraction is by AST value position: a class attribute's initializer, an argument
to cn/clsx/twMerge/cx/toHaveClass, or a `*Classes` variable's initializer.

The word *value* is the whole difficulty and it was measured rather than argued.
Every literal filtered by namespace gives 45 distinct false positives and still
misses `tabular-nums`. The flag propagated to every descendant gives 31.
Propagated to values only gives 0. Four descents make that difference, each a
real false positive before it was closed: not into a call's callee, not into an
element-access index, not into a condition, not into a non-class call's
arguments.

### What it cannot see, with the sizes

Prose: 4 lines, measured. `@source not` already decided prose is not usage.

The `*Classes` convention: it is how 30 constants are found. Rename one and
coverage drops, which is what the printed *outside* count exists to announce.

A class built from an expression: counted, not dropped. Zero today — there is not
one interpolated class name in the repository, and the four concatenated base
strings never split a token.

A class that resolves to the wrong value. `--radius-element` changed to `2rem` is
invisible to this. That is the visual half, and it stays open.

### Three numbers in the sensor's own docs were wrong

Written from exploration rather than measured: "17 variant maps", "six
concatenations", "20 sites of noise in internal/". The real numbers are 30, 4 and
0. Caught and corrected by the agent doing the work.

That is the census this record keeps, appearing inside the sensor built to check
a different half of it.

### Where it runs

The gate only, 0.3s over 276 files, with a build ahead of it at about a second
warm. It refuses a stylesheet older than the newest file feeding it and names
both files rather than answering confidently about a checkout that no longer
exists.

CI gained a Build step to make this possible, which closes the separate gap that
nothing automated here had ever compiled the stylesheet.

### Rejected

A lint rule: ESLint sees one file and no CSS, so it would need a second copy of
tokens.css, and the drift is the defect. Tailwind's own compiler: undeclared
transitive dependency, internal API, and it answers "would this compile" rather
than "did this ship" — all three first-run defects were files Tailwind never
scanned. eslint-plugin-tailwindcss: wants a JS config v4 does not have. A CSS
snapshot: fails on every legitimate change and names no class. postcss and jsdom
computed styles, for reasons already recorded.
