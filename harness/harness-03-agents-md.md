    Branch: harness/03-agents-md
    Baseline: harness-02
    54 lines, one file, no code changed

### What it carries
Eight decisions, and only decisions no tool can check. Everything a linter, the
type system or a test enforces is already enforced and is deliberately absent —
glyph icons, raw type classes, copied primitive class strings, deprecated types,
the thresholds, the package boundaries. The file points at
internal/eslint-plugin-harness/README.md and packages/ui/README.md rather than
restating either.

Also absent: tokens, cn(), and accessibility props, which the UI README covers
and the type system partly enforces.

### The split
The eight sit under two headings, added after the first draft.

**Conventions** follow from how the design system is built and do not change
with the product: CardBody owns card padding, an editable surface is a form with
type="submit", a discriminated union renders through a switch, Alert and
StateMessage are never hand-assembled, a repeated block inside a .map() gets
extracted.

**Product decisions** are what this product chose, and they change here first:
the five button variants, Modal versus Drawer, Table versus List.

The test for which side a rule belongs on: could a designer change it tomorrow?
CardBody's padding value can change, but the rule is "use the primitive", and
that does not. Whether a screen may carry two primary buttons genuinely can.

The reason for splitting: a redesign only has to read the second half. Without
the split, every rule reads as permanent and all eight get re-litigated.

### One thing the writing exposed
The first draft said danger appears "on the confirming button of a
ConfirmDialog". Two existing call sites contradicted it as written —
TicketDetailPage and BulkActionsBar both use danger on the trigger that opens a
ConfirmDialog rather than on the confirm inside it.

Reworded to "danger marks a destructive action, on the trigger and on the
confirming button alike. Always route a destructive action through
ConfirmDialog." Both call sites are then correct, and the rule says what was
actually meant.

Worth noting as a property of writing this file at all: stating a rule precisely
enough to be followed is also stating it precisely enough to be checked against
the code, and the code answered back.

### Escalation
The file closes with the rule that governs itself: a rule broken repeatedly
becomes a lint rule — a file under src/rules, a docblock, a test, and a ROLLOUT
row. Taken from OpenAI's practice of promoting a rule into code when
documentation falls short.

### What is expected to happen to it at scale
At four screens this file is read. At fifteen, the task context crowds it out —
that is OpenAI's finding about large instruction files, and 54 lines is small
enough to survive longer than most, but not indefinitely.

The Conventions half is the more durable one: those rules also have primitives
behind them, so an agent that ignores the text still hits the component. The
Product decisions half is prose only, and prose is probabilistic compliance.

Prediction to check on the next measurement: the conventions hold, the product
decisions are the ones that get guessed.
