    Branch: harness/02-guides
    Baseline: harness-01
    Tests: 312, up from 251 — 61 new, all of them plugin tests
    Lint: 19 warnings, 0 errors

### What was added
An ESLint plugin at internal/eslint-plugin-harness, plain ESM so ESLint loads
it with no build step.

Two tiers. `recommended` is all warnings, for humans mid-edit. `strict` is
errors for the rules the codebase already passes, warnings for the rest, and is
selected from CI=true or HARNESS_STRICT_LINT=1. The reasoning is taken from
Meta's Astryx plugin: an agent has no excuse for violating a stated rule, a
human mid-edit does.

Three custom rules:

**no-glyph-icons** — non-ASCII characters in JSX text. Written for the five icon
glyphs across four measurements, which Icon had already closed by the time the
rule shipped.

**no-raw-type-classes** — text-xs through text-2xl and font-semibold in feature
code, against the semantic scale added in layer 1.

**no-primitive-class-copying** — a raw element carrying a class string that
belongs to CardHeader, CardFooter, CardBody or StateMessage. Four components had
copied CardHeader and CardFooter by hand.

Configured rather than written: @typescript-eslint/no-deprecated,
max-lines-per-function, complexity, max-depth, max-params, and
dependency-cruiser for the package boundaries.

### Violation counts on arrival

    no-glyph-icons                3    (all false positives, see below)
    no-raw-type-classes          10
    no-primitive-class-copying    0
    thresholds                    0
    no-deprecated                 6
    dependency-cruiser            0

### What the rules found that nobody knew about
no-deprecated found six, not the four FormEvent sites the notes predicted. The
two new ones: queryClient.fetchQuery in useTicketsExport, and recharts' Cell in
BarChart. Both carry a concrete migration path in their own deprecation message.
Neither had ever been noticed — deprecations are a hint to tsc, not an error, so
typecheck passed the whole time.

That is the clearest argument in the record for a deterministic sensor: two
defects, invisible for months, found the moment a rule looked for them.

### What did not work
All three no-glyph-icons hits are false positives. They are typographic
punctuation in JSX text — "Loading customer…", "Loading ticket…", and a middot
on the ticket detail screen. The rule cannot tell punctuation from an icon
glyph, and the message compounds it by suggesting `<Icon name="close" />`
regardless of which character was found, which is nonsense for an ellipsis.

Options: allowlist … and ·, drop the fixed example from the message, or leave it
at warn. Unresolved at time of writing.

### The messages had to be rewritten
They shipped as four-line paragraphs carrying the full reasoning. In a lint run
the same essay repeated on every instance and the output was unreadable — ten
violations of one rule printed the same paragraph ten times.

Shortened to one sentence plus a README pointer. The reasoning moved to the rule
docblocks and the plugin README, where it is read once rather than per
violation.

Worth recording as a limit on the Astryx message style: a message should tell
the reader what to do, and the reasoning belongs somewhere it is read once.

### Thresholds

    max-lines-per-function   185   worst today 181 (TicketListPage.tsx)
    complexity                20   worst today 20 (TicketListPage.tsx)
    max-depth                  3   worst today 2
    max-params                 5   worst today 5 (fail() in apps/api/src/app.ts)

Two of the four sit exactly at the worst rather than above it, so there is no
headroom and the next line added to those functions fails the rule. That is
arguably the right behaviour, but it was not a deliberate choice.

185 is meant to look large. The honest reading is that this repo contains a
181-line function.

Off in test files: a describe block is a container, not a function anyone should
extract.

### Implementation notes worth keeping
ESLint core rules cannot carry a custom message, so thresholdRules.js wraps each
core rule and replaces only meta.messages. The counting logic is ESLint's,
untouched; the thresholds stay in eslint.config.js, which the messages point at.
That was the only way to satisfy both "configure these rather than write custom
rules" and "the message must say raising the threshold is an acceptable answer".

Exemptions live in the rule source with a test each, not in eslint.config.js, so
a config edit cannot widen them silently. Two exist: Avatar's size classes,
which size initials rather than type, and Table's font-semibold, which is a
weight on text-caption. Both tests also cover the narrowness — text-2xl in
Avatar still fails, font-semibold outside Table still fails.

dependency-cruiser needed preserveSymlinks: false to follow workspace symlinks
and tsPreCompilationDeps: true so type-only imports count. Both boundary rules
were verified by planting probes — packages/ui importing a feature file, and
packages/shared importing packages/ui — and both were caught.

### The baseline was already describing a system that did not exist
rep-7 and rep-8 are carried here rather than left in the reports scaffold,
because they qualify the whole log and this is the layer that had to decide what
to do about them.

Every measurement is scored against the design system README as a fixed
baseline. That README's opening section claimed the Tailwind palette had been
removed and that bg-blue-500 produced no CSS, months after that reset was taken
out (rep-7), and its loading-state paragraph described TableBody, StatCard and
BarChart sharing an approach that all three had implemented privately (rep-8).

So "the stated rules held" means they held against a document that was partly
fiction. It does not undo the findings — nothing false was said about colour
tokens, Badge unions or cn(), which is where most of the record sits. What it
qualifies is the framing: the yardstick was not neutral, and a guide describing
a more coherent system than the one that exists will make the next agent
confident about something untrue.

Layer 2 does not close it. Every sensor added here reads syntax. Whether a
paragraph still describes the code it governs is a question about meaning, and
no rule in this plugin can ask it.

### What this layer does not close
Card padding. Every value is on the spacing scale, so no rule can catch it.
Eight distinct values across the app and no lint rule will ever fire on one of
them. This needs a layout primitive or a written rule. Survived layer 1 for the
same reason; nothing in layer 2 changes it.

Which button variant an action gets. Not mechanically checkable — a machine does
not know which action is secondary.

When to extract a component. The threshold rules approximate it, but the number
is a judgement somebody had to pick.

Comments that contradict the code beside them, and documentation that outlives
it. Both need semantic reading. See the section above, and observations.md.

Visual consistency — row heights, alignment, the shape of an empty state. The
category m05 introduced, and the one nothing in this layer touches.
