    Baseline: fix-m13
    Tag: harness-07
    Layer: 4 — sensors
    Tests: 562 before / 703 after
    Lint: 5 warnings, unchanged
    Boundaries: 332 modules, clean

The first three layers constrain what gets written. None of them asks whether
the result is correct, accessible, original or honestly described. This is the
fourth, and it is the one the record has been asking for since m05.

### The premise corrections

Four of the instances named in the brief were already fixed — the aria-modal
gap, the Modal/Drawer duplication, the naming dialog copied onto a second
screen, and the list role, which packages/ui does set. The classes were all
real, so the live instances were targeted instead.

The brief was written from these notes, and the notes describe the state of
measurement branches rather than main. Same shape as m13's false premise, and
the second time it has happened.

### The five sensors

Each one is stated the same way before anything else about it:

    Problem:
    Mechanism:
    Tool:

**Mutation testing.**

    Problem:   A test can execute a line without asserting anything about it.
    Mechanism: Change the source and see whether a test dies.
    Tool:      Stryker 10, Vitest runner.

487 mutants over packages/shared plus the two non-JSX files in packages/ui.
84.39% killed. The threshold is 82, set from that first run rather than chosen.

Both named instances confirmed, and both deliberately left unfixed.
reports.ts:49 survived: `<=` on a range the file's own docstring calls
inclusive, so a 367-day range passes a validator whose message says 366.
useFocusTrap.ts:58-63 came back as no coverage, which is the stronger result —
and the test named for it is called "when there is nothing in it to focus",
while Dialog always renders a close button.

Two tiers, because 81 minutes is not a loop. The fast tier drops the web
project: 5 minutes, 77.21%, seven points down.

The cost was documented per file, and the assumed cause was wrong. The guess was
that the web tests were redundant through MSW. types.ts fell from 100 to 33 and
navigation.ts from 57 to 43, because those files are mostly labels — rendered,
never served.

Cannot see: it mutates source, not fixtures. DateRangeField.test.tsx declares
two presets ending on the same date, so half of isSameRange can be deleted with
every test green.

Rejected: a coverage threshold. A number that gets optimised.

**Accessibility.**

    Problem:   Nothing asks whether a screen works for anything but a mouse.
    Mechanism: Rules for what lint can see, axe over the DOM the tests render.
    Tool:      require-list-role, aria-modal-needs-focus-trap, axe-core.

It cost nothing to stand up: axe runs over the DOM the 703 tests already
render.

require-list-role found the four live instances, all of them flex containers.
axe does not catch them — its list rule checks for `<li>` children, and all four
had them. axe swept 27 components clean, and 8 screens, finding `region` on both
auth screens, which render outside AppLayout. Recorded, not fixed.

aria-modal-needs-focus-trap reports zero, and ships anyway. The gap it names is
one of the four the brief had already closed — BaseDialog gained the trap in
layer 1 — so it is a ratchet rather than a finding: nothing holds that fix in
place except the rule. The mutation run says the same thing from the other
side. useFocusTrap is the file whose own test turned out to assert nothing.

Cannot see: the entire visual half of WCAG. jsdom has no layout, so contrast,
target size and focus visibility are unchecked here and nowhere else.

Rejected: eslint-plugin-jsx-a11y. Peer-capped at ESLint 9, and on this codebase
it finds nothing anyway, because Input.types.ts removes `id` from the public
props.

**Duplication.**

    Problem:   A copy passes every check the original passes.
    Mechanism: Compare structure between files, score by share-of-file.
    Tool:      ~200 lines over the typescript-eslint parser, no new dependency.

jscpd was rejected on measurement rather than on principle. All three of its
modes filter tokens by type and hash token values, so it fires on
Input/Textarea and on LoginPage/RegisterPage — both legitimate — and is silent
on useBulkDelete*, which is 117 of 117 tokens structurally and 28 literally. It
inverts signal and noise.

Absolute thresholds cannot work here either: the largest structural clone in the
repo is a legitimate pair. The metric that separates the two kinds is
share-of-file, and no off-the-shelf tool offers it.

Threshold 0.85, 12 pairs recorded — 9 parallel, 3 real copies.

Cannot see: SavedViewNameDialog/TicketMoveReasonDialog, ruled out at 48%, and a
genuine finding. Lowering the threshold to catch it surfaces about 50 pairs.
Both numbers are pinned in tests.

**Document freshness.**

    Problem:   Prose outlives the code it describes and nothing reads it.
    Mechanism: Resolve every path and symbol a document cites against the tree.
    Tool:      Two rules over comments and Markdown.

doc-path-exists found 6, all of them residue of the monorepo move, including a
"read this first" link that was a 404. 45 tokens checked, 0 false positives.

doc-symbol-exists is restricted to SCREAMING_SNAKE_CASE, and the numbers say
why: 25 cited, 25 resolve. PascalCase gives 30 unresolved and camelCase 71, all
of them false.

Cannot see: support-desk/README.md opens by saying the repo has no AGENTS.md
and no lint rule — three times false, and citing neither a path nor a symbol, so
neither rule has anything to resolve against. That is most of the category.

That claim now reads in the past tense, rewritten on this branch by hand. The
sensor could not see it and a person fixed it in the same change, which is the
part worth keeping: what closed the instance was somebody reading the file, and
nothing built here makes that repeatable.

**The gate.**

    Problem:   A sensor nobody runs reports nothing, indefinitely.
    Mechanism: Fast checks on every write, everything slow on the branch.
    Tool:      A PostToolUse hook, and CI.

eslint.config.js has promoted rules on CI=true, for four commits, against a CI
that never existed. The strict tier had never run.

A commit hook was rejected, and the reason is the constraint itself: it lives in
.git/hooks, untracked — which is precisely a rule that does not show up in a
diff.

### What the work caught about itself

Two, and both belong in the note.

The first full mutation run scored 39%, with 186 mutants reported uncovered in
files known to be covered. Workspace symlinks escape Stryker's sandbox, so the
tests ran against the unmutated original. A sensor reporting a confidently wrong
number is the failure mode this layer exists to prevent, and the first instance
of it was the sensor's own.

Worth noting that layer 2 met the same feature of the tree from the other side —
dependency-cruiser needed `preserveSymlinks: false` before it would follow a
workspace import at all. Different mechanism, same cause, and in both cases the
tool returned a number rather than an error.

And the hook's first catch was the work that built it: a config file written and
left out of every tsconfig.

### What is still open

reports.ts:49 and the Dialog test were left unfixed deliberately. They are the
evidence that the sensors work, and a sensor that ships with nothing to catch
has never been seen to catch anything. Both go to a separate fix branch.

Recorded and not fixed here either: `region` on the two auth screens, and the
DateRangeField fixture that leaves half of isSameRange deletable. Both are
things a sensor found. support-desk/README.md's opening claim is the one that
got fixed, and it is the one no sensor found.
