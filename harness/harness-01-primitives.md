    Branch: harness/01-primitives
    Baseline: baseline-ui-package
    Two commits: the new pieces, then the migration

### What was added
Icon, wrapping lucide-react so feature code never imports the library, with
sizes from the token scale rather than surrounding text size.

A semantic typography layer in tokens.css: --text-title, --text-section,
--text-subsection, --text-body, --text-caption, each with its own line height
and, where it matters, font weight.

Heading, taking a semantic level with an optional element override so visual
weight and document outline are chosen separately. CardHeader gained the same
flexibility, defaulting to its previous behaviour.

Text, for body and muted helper text, carrying wrap-anywhere so a long unbroken
string cannot push a card off the page.

ConfirmDialog, extracting the Modal + secondary Cancel + danger confirm shape.

A selected variant on Button.

StateMessage, one implementation of the loading and empty state.

BaseDialog, shared under Modal and Drawer, and a focus trap added to it —
neither trapped focus before, while both set aria-modal="true".

### Findings this closed
Glyph icons: m02-1, m05-1, the StatCard arrows, the Modal and Drawer close
buttons, m06-1. Five instances, verified gone by grep.

Hand-assembled page titles: six screens.

Hand-built confirm dialogs: five instances.

Hand-written selected states: five instances, two of them character for
character identical.

Duplicate empty and loading states: four implementations, three of them the
same string literal.

CardHeader and CardFooter class strings copied by hand: four instances.

The two error banners never migrated onto Alert, which had existed since the
reports scaffold.

### What it did not close
The design system's own files kept their raw type classes until a follow-up
pass. See observations.md.

Card padding is still decided per call site. Every value is on the spacing
scale, so no rule can catch it. This needs a layout primitive or a written rule,
not a lint rule — it is the clearest case in the record of "on the scale" not
meaning "consistent".

Which button variant a given action gets is still unstated. The variant now
exists for selection; the rest is a decision nobody has written down.

### Layer 1b — Checkbox and Alert variants

    Branch: harness/01b-primitives
    Merged with: remeasure/07-customers-export, measure/08-customers-bulk
    Tag: harness-01b
    Tests: 357

Two gaps reached three instances each and were closed the same way the first
five were.

**Checkbox.** A raw `<input type="checkbox">` with a hand-written
`size-4 accent-primary` existed in the ticket table's row and header cells, in
MultiSelect's option list, and in ListRow's selection slot. The primitive it
gained on the way in: an accessible name it cannot render without, an
indeterminate state — the ticket table's header cell needed one and had no way
to say "some of these" — and the disabled treatment the other primitives carry.

**Alert variants.** Alert was being imported for its tone and role and then
switched off with className, three times and three different ways. It now
carries the two shapes that were being asked for: an inline error that sits in a
row of controls with no chrome, and a band that spans a card edge to edge. The
ErrorBand wrapper in CustomersPage became redundant and was removed.

### A note on ordering

harness/01b was branched from main while m07 and m08 were still unmerged, so the
third checkbox call site — ListRow's selection slot, added in m08 — did not
exist in the branch. It was written for two sites and migrated a third after the
merge.

Worth recording as a property of the method rather than a mistake: measurements
are held off main deliberately, and harness work branched from main is therefore
working against a codebase that is one or two measurements behind. Either merge
the measurements first, once their findings are written, or expect to finish the
migration after the merge.
