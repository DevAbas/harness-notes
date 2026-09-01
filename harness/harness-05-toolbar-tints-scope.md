    Branch: harness/05-toolbar-tints-scope
    Baseline: baseline-redesign
    Tests: 417, up from 407
    Lint: 5 warnings, down from 8
    Duration: 24 minutes wall, $10.93
    Files touched: 19 modified, one new component directory

Not a measurement. Three pieces of harness work, all of them named by m09.

### The Toolbar primitive

m09-1 recorded that a density change cost thirteen lines of feature code, every
one of them padding on a strip that is not a card. CardBody owns card padding;
nothing owned strip padding, so a token could not reach it.

Toolbar now does. After the migration there is one `px-3 py-2` left in feature
code, and it is not a strip — it is StateMessage having its padding overridden
through className in SavedViewsSidebar.

That one is worth noting on its own: it is the same shape as the Alert findings
from m07 and m08. A primitive exists, the shape it needs does not, and className
is the only way to ask. Third instance of that pattern, first one involving
StateMessage.

### The subtle tints

Five statuses now have five distinguishable tints:

    primary  #bee7db    was #e4edef
    info     #c9f5f5    was #e7f0f2
    danger   #ffd9d7    was #f6e3e2
    success  #d7eecb    was #e6ede6
    warning  #fff3b0    unchanged

All five are more saturated than what they replaced. The old set was pale enough
that three of them converged; the new set is visible on a #FDFBF0 body without
being loud.

Note what was not the problem. The saturated versions of the same families were
always distinct — the report chart rendered four legible bars throughout. The
fault was entirely in how the subtle tints were derived: lightening a family
with three members does not produce five distinguishable results.

### The lint scope

no-raw-type-classes now covers apps/web/src rather than apps/web/src/features.
The old scope was the whole app when the rule was written; app/ has since grown
to hold AppLayout, AppRoutes and the auth routes, and the rule was blind to all
of it.

Widening it surfaced the violations and they were migrated in the same change,
along with the three that were already reported in features. The rule now
reports zero.

Five warnings remain: two no-deprecated — fetchQuery and recharts' Cell, both
still outstanding from harness-04 — and three no-glyph-icons false positives on
typographic punctuation, which have been known since layer 2.

### What the agent built to check its own work

The brief said five badges side by side are five colours, so check them side by
side rather than one at a time. To satisfy that, the agent generated an SVG
swatch: before and after, four groups — plans, statuses, priorities, raw tokens
— each rendered twice, once on surface and once on surface-muted.

Nothing asked for a visual comparison tool. Claude Code has always been able to
write a file and open it; across nine measurements it never did this. What was
different was the acceptance criterion: "five badges side by side are five
colours" cannot be checked by reading hex values, so a way of seeing them
together had to exist.

Worth separating two things. The capability came from the tooling. The behaviour
came from the shape of the task.

And it does not generalise. In harness-01c the radius reset silently broke Badge
and Avatar on every screen, and nothing was built to catch it, because nothing
in that task required looking. A sensor that appears when asked for is not a
sensor — defects arrive where nobody is looking, which is the whole argument
for the visual layer this record keeps naming and has not built.

### On cost

First entry with a figure: 24 minutes and $10.93, against roughly 200 source
files and 417 tests.

Some of that is the codebase. Some of it is the harness itself — AGENTS.md, the
plugin README and the design system README are all read before work starts, and
that is what layer 3 costs on every task.

And some of it was the brief. Three unrelated pieces of work in one prompt, one
of them opening with "look at the nine call sites before naming it", meant
twelve minutes of reading before a line was written. Three separate prompts
would likely have been cheaper; combining them did not save the overhead, it
multiplied the context each part had to be held in.
