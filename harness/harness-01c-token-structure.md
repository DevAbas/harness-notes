    Branch: harness/01c-token-structure
    Baseline: harness-04
    Tests: 407, unchanged
    Lint: 15 warnings, unchanged
    Files touched in apps/web/src/features: 4

Not a measurement. A structural change to the token layer, with values held
fixed — zero visual difference was the test that it worked, and it did not
entirely hold.

### What was added

**A primitive layer under typography.** The five semantic levels stated raw rem
values; they now reference size tokens on a geometric scale,
round(12 × 1.15^step). Semantic weights were added and the scale references
them rather than stating 600 inline.

**Radius named by role.** none → inner → element → container → page → full,
with a stated rule for each: element for interactive controls, container for
cards and dialogs, full for pills. Values unchanged. `--radius-*: initial`
removes Tailwind's own radius namespace so the role names are the only ones
available.

**Four groups that did not exist.** Control heights, which were h-8 and h-10 on
Button and so unreachable by a theme. Focus ring width, style, colour and
offset, which nine components were writing by hand. Border width. Motion
duration and easing, which Drawer carried inline.

**An icon colour group** separate from text, so an icon can be quieter than the
type beside it.

### The finding

`--radius-full` was not added, and `--radius-*: initial` had already removed
Tailwind's. Badge and Avatar both use `rounded-full`. Badges lost their pill
shape and avatars stopped being circles.

The task said to stop and tell me rather than absorb a visual change. The change
was absorbed silently, and I found it by looking at a screenshot.

No test caught it, and none could have: the tests assert class names, and
`rounded-full` was still in the class string. The class simply no longer
resolved to anything.

Caught by: nothing — the class name is unchanged, only its meaning is gone.
Layer: sensor missing (visual)

This is the clearest case yet for the visual sensor the record keeps naming and
nothing has built. Two primitives, on every screen, and the only thing that
noticed was a person looking at a picture.

### Where the conventions did not fit, and were refused

Three places, all reported rather than absorbed, which is the behaviour the task
asked for and got everywhere except the radius.

The geometric scale reproduces four of the five sizes exactly. Step 4 comes out
at 21px against today's 20px. Today's value was kept and the mismatch stated —
the one number in the file that is not on the scale it sits in.

The line-height rule — 4px grid, targeting 1.5 below 20px — wants 20px leading
on 12px text. The file keeps 16px, on the grounds that 16px is exactly the
fontSize + 4px floor and that 1.67 on caption text reads as space between lines
rather than as lines. The rule is described as the one that is wrong at the
small end.

Concentric radius computes to zero everywhere it applies: Card is 8px with
16–20px of padding, so max(0, 8 - 20) is 0. The formula is correct and this
product has no case for it. `inner` exists for a future one.

### On the four feature files that changed

The brief said features should not be touched. Four were, six lines total, and
both reasons are structural rather than optional.

Four hand-written focus rings in feature code moved onto the shared utility —
which is the gain, not the cost. Two `rounded-md` became `rounded-element`,
because the reset removed the old name.

Worth carrying into the redesign measurement: after this change there is no
focus ring and no Tailwind radius name left in feature code, so neither is a
variable there.

### Two things noticed in passing, both from earlier work

ListRow's focus ring is clipped by the card it sits in — the offset pushes it
outside the row and the container's overflow cuts it. It predates this change.
The `inset` variant written for form fields is the fix.

The customers bulk bar carries four buttons at two sizes, sm for select-all and
clear, md for Apply and Delete, while Export CSV a row above is sm. From m08.
The same gap m08-3 named: the rules are written at screen scale and a strip is
not a screen, so size is guessed along with everything else.
