# Measurement 6 — "Add saved segments to the customers directory."

Task: see m06-prompt.txt
Branch: measure/customer-segments
Harness state: none (README only)
Prompt style: one line, nothing specified
Baseline: baseline-customers
Codebase: ~6 screens, ~110 source files, 222 tests
Tests: 222, up from 143

This measurement was set up to test ambiguity. The repo already answered
"how do you save a filter combination" once, in m02, on a screen with
different conventions and a different filter shape. Nothing says whether to
follow that precedent or fit the new screen. The question was whether the
agent would notice there was a choice.

## m06-1 Glyph icons, fifth instance — and the same two codepoints as m02
CustomerSegmentBar renders &#9998; and &#215; on the rename and delete controls.
The same two characters, in the same order, on the same two buttons, as
SavedViewsSidebar in m02. Both carry aria-labels naming the segment, which is
correct.

Fifth instance after m02-1, m05-1, StatCard's arrows and the Drawer close
button. The design system now has around fifteen primitives and no Icon.
Caught by: nothing
Layer: guide missing (one lint rule banning non-ASCII glyphs in JSX text closes
all five)

## m06-2 Everything stays inline, exactly as in m02-4
The 30-line chip block inside segments.map is not extracted. "All customers"
repeats nearly the same markup a second time. chipClasses and activeChipClasses
sit as module constants away from where they are used. The chip cannot be
tested on its own.

This is m02-4 reproduced in shape and in scale, four measurements later.
Caught by: nothing — no complexity, nesting or file-length rule
Layer: guide missing + sensor missing

## m06-3 A discriminated union rendered as three ternaries
The Dialog type is correct — save, rename and delete each carrying what they
need. The render uses three independent `dialog?.kind === '...' ? ... : null`
blocks, so TypeScript cannot check exhaustiveness and a fourth kind would
silently render nothing.

Identical to m02.
Caught by: nothing — the type is sound, only the render loses the guarantee
Layer: guide missing

## m06-4 No ConfirmDialog, fifth hand-built instance
The delete confirmation is assembled again: Modal, secondary Cancel, danger
confirm. After TicketListPage's bulk delete, SavedViewsSidebar (m02-5), the
taxonomy removal dialog (m05), and Drawer's footer pattern.

The same internal inconsistency as m02 recurs: the name dialog was extracted
into a component, the delete dialog was left inline, in one file.
Caught by: nothing
Layer: guide missing

## m06-5 A fifth hand-written selected state, and this one is identical

    'bg-primary-subtle text-primary-subtle-fg hover:bg-primary-subtle
     hover:text-primary-subtle-fg'

Character for character the string SavedViewsSidebar uses. The earlier
instances — DateRangeField, Tab, ListRow — were near-identical, differing by a
class or two. This one is a straight copy.

Button still has no selected variant.
Caught by: nothing
Layer: guide missing

## m06-6 CustomerSegmentNameDialog is SavedViewNameDialog with three strings changed
Same props interface, same useId form trick with the same comment transcribed
verbatim, same trim-then-check-duplicates logic, same case-insensitive
comparison, same error-clears-on-typing, same Modal footer, same Input prop set.
The differences: "view" becomes "segment" in two error strings, the label, the
placeholder, and an added docstring paragraph.

The verbatim comment proves it was read and transcribed rather than
reconstructed. The deprecated `type FormEvent` import came with it, which is
m04-1 propagating to a fourth file.

Worth stating plainly: this is a faithful copy of good code. Input's error prop
is used, Modal is imported rather than rebuilt, and the docstring reasons
correctly about why this is a Modal and not a Drawer. The result is two correct
implementations that now have to be fixed twice, and one generalised NameDialog
taking a noun would have covered both.
Caught by: nothing — no duplication detection, and the files are far apart
Layer: guide missing

## m06-7 A fourth empty-state treatment
A bare <p className="text-sm text-fg-muted"> inline in the strip. After
TableBody's, BarChart's and List's — the last two being identical string
literals in separate files.
Caught by: nothing
Layer: guide missing

## The result this measurement was for
The agent noticed the choice and argued one side. It rejected the sidebar
deliberately, and wrote down why: a dense list is scanned down for a person so
it wants its width, the filters below are one line for the same reason, and a
column of names beside sixty rows would take from the rows to say something a
row of chips says in the space already there. It also named the cost of its own
decision — rename and delete appear only on the selected chip, so deleting one
means selecting it first.

In five previous measurements it never reasoned about a design question that
way unprompted. The only comparable moment was m05, where it stopped and asked.

But the reasoning went only as far as the layout. Everything beneath it was
copied from the file it had just explained it was not following — the glyphs,
the inline block, the ternaries, the missing ConfirmDialog, the selected state
string, and the naming dialog wholesale.

This is the clearest evidence in the record for the claim that an agent
replicates the patterns already in a repository, including the flawed ones,
because here it explicitly said it was not doing that, and then did, in every
respect except the one it was thinking about.

## What was done well
customerSegments.ts validates what it reads from storage with type guards that
drop entries no longer parsing, including a plan that has left the domain.
Stored entries are normalised on both directions — plans in domain order,
search trimmed — so the same set of plans is one cache key rather than two.

areFiltersEqual compares plans as a set, so ticking order cannot make a segment
look modified, and compares search trimmed because listCustomers trims it too.

Changing a filter marks the active segment Modified rather than unselecting it,
which is the m02 behaviour carried over correctly.

The list semantics are explicit — role="list" with a comment explaining that
laying items out with flex removes the bullets, and in some browsers that takes
the list semantics with them.
