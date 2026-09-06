These are properties of the method rather than findings from any one
measurement.

### Deleting a constraint makes the typechecker quieter, not louder

In m05 the agent was told explicitly that the status and priority unions stay
compile-time constants. It deleted them: TicketStatus and TicketPriority are
`string` now. StatusFilter collapsed with them, so applyBulkStatus('resolvd')
and changeFilters({ status: 'Open' }) compile. Record<TicketStatus, string>
exhaustiveness is gone. Runtime membership checks were then added to three API
routes to replace, imperfectly, what the compiler had been doing for free.

Typecheck passed. It passed more easily than before.

The general property: every deterministic guide can be deleted by the agent the
guide constrains, and its removal registers as success. A structural test
asserting TICKET_STATUSES stays `as const` would have made the change visible.
A CI check flagging any union-to-string widening is cheap and deterministic.
Neither prevents it. Visibility is the achievable goal, not prevention.

Worth noting this happened on the one measurement where the agent asked first.
It identified the fork correctly, was answered, and built the excluded version
anyway.

### Comments that describe intent rather than the code beside them

Twenty-four instances. The number stood at five through four measurements and
two audits without being re-taken, which is the failure this file recorded after
m08: a count in a finding is a search, not a judgement. Re-taken at m10 against
the files, at seventeen. Re-taken again here, after m14, across everything the
record has gained since — m11 through m14, harness-06 through harness-08, and
the fix passes on m12 and m13. The seven new ones were read in the files rather
than off the notes: at their measurement commits where a fix has since closed
them, and in the branch for m14's three.

What counts is a claim: about behaviour, about a sibling, or a count or a rule
stated as fact. A citation that does not resolve is not one — harness-07's two
doc rules find those, and the six dead paths they found belong to that layer's
numbers rather than to this list. m12-5 is the exception and stays: the symbol
rule is restricted to SCREAMING_SNAKE_CASE and the name it cites is a camelCase
hook, so nothing resolves it either way.

Not counted here: m13-1 and the two the m13 fix sweep found beside it. They are
prose describing a past the repository does not have, which has its own entry
further down.

A comment arguing for behaviour the code beneath it does not have:

    m03-1  useTicketsExport's docstring says it "exports every ticket the
           filters match", one screen above a clamp at 500
    m03-2  tickets.ts says parsing locally "means the failure names the field
           instead of arriving as a round trip and a 400" — it strictly degrades
           the failure it claims to improve, and http.ts one file over opens by
           promising that every failure arrives as the same ApiError
    m03-5  a 30s staleTime justified as keeping a queue worked on in another tab
           current, fourteen lines above refetchOnWindowFocus: false
    m05-5  "Rendering nothing rather than a disabled editor keeps an agent from
           reading it as something to unlock", above a card rendering "Switch
           role above to edit them"
    m05    the taxonomy hook's empty set "is what they would render anyway" —
           true for a badge, false for the two dropdowns that gate an action
    m07-1  fetchEveryCustomer argues for the cursor walk because a truncated CSV
           looks exactly like a complete one, then truncates at page fifty
    m10-2  close() names the risk that "the old results would otherwise be the
           first thing on screen", and leaves them in the cache
    m13-2  a view is stored canonically "so that a view is never modified the
           moment it is saved" — with an optional filter it is marked modified
           the instant it is saved, and on every reload after
    m14-1  a ladder rule's requirement "names the pair rather than the plan
           being edited", so that neither end of a broken rung is sent to the
           wrong screen; the sentence it returns ends "or price this one above
           it"
    m14-2  the plan is "held by name rather than as a row, so the dialog is
           always looking at what the query holds now", above four useState
           initialisers that read it once, and a prop documented as the plan
           "as the server last reported it"

Documentation describing a system, or a sibling, that does not match:

    rep-7       the design system README's palette claim — bg-blue-500 "produces
                no CSS at all", months after that reset was removed
    rep-8       the same README on loading states: three components sharing an
                approach that three of them implemented privately
    cust-3      the dialog contract listed as complete, one item short of what
                aria-modal promises, and Drawer's docstring inheriting it whole
    cust-5      List's docstring says empty and loading states belong to it "for
                the same reason they belong to TableBody", written as the third
                copy
    m08         `leading`'s one-line doc still offers a checkbox, eleven lines
                above the prop whose docblock explains that a checkbox cannot go
                there
    m10-4       Input's labelHidden documents itself as the same mechanism as
                Checkbox's, which uses the opposite one
    m12-5       useBulkUpdateCustomerPlan's docblock names the division
                useBulkUpdateStatus makes, in the change that deleted it
    harness-07  support-desk/README.md on what the repo does not have: no
                AGENTS.md, no lint rule enforcing the design system, the rules
                written in prose somewhere else — three claims, three false. The
                link beside them was dead too, and the link is the only part of
                the paragraph a rule could reach.

Prose stating a count or a rule the code does not carry:

    m08-2       ListRow's "one of the three places to change" — it was four
    m08         MAX_CUSTOMER_BULK_IDS's rationale: a considered departure from a
                value it is identical to, citing a constant that plays no part,
                bounding nothing
    harness-04  the eslint config comment saying four violations; it was six
    m10-1       AGENTS.md's gating invariant, restated in RequireRole.tsx,
                satisfied for one screen of four
    m12-1       AGENTS.md and README.md on a fifth status: an entry plus the
                moves that reach it, and "no screen that draws a status has to
                change", where TicketStatusBadge's Record keyed by the union
                makes it a compile error on a screen. The same paragraph's
                "three tables and four pure functions" counts a table that lives
                in another file, misses one in the file it is describing, and is
                out by three on the functions.
    m14-5       the plan bounds sit outside the schema because the fields read
                them too — "a maxLength on the description, a max on the seat
                field". The seat field is a text input with inputMode, twelve
                lines under a comment arguing for exactly that, and max does
                nothing outside type="number".

Twenty-four, and the count was never the argument. Three instances is the
threshold this repo closes a gap at, and that was passed before the harness
existed. Every other census in this record ends in a primitive — five glyphs
became Icon, four hand-written selected states became a variant, three Alert
overrides became a prop. There is nothing to write here. A comment cannot be
made to check itself, and no rule reads English against the code beneath it.

Layer 4 was built between the two takes, and part of it was built for this. The
doc-freshness sensor's two rules close none of the twenty-four. They resolve
what a document cites, and only two of these cite anything at all: a camelCase
hook the symbol rule excludes to stay useful, and a dead link sitting beside
three claims it says nothing about.

The distribution is ten, eight and six. The first ten are a function's comment
against its own body, which costs a reviewer's attention. The last six state a
fact — a count, a rule — which is the kind of prose later work quotes rather
than re-derives, and AGENTS.md is two of them.

m14 is the strongest form the finding has taken. Three instances, in a
measurement with all four layers in place and the sensors running inside the
agent's own loop for the first time. The agent ran them, corrected what they
reported, and wrote three claims that none of them can read.

The comments in this codebase are good enough to buy trust. That trust is
occasionally misplaced, and a reviewer who reads the comment stops looking.

### Documentation that outlives the code

The design system README stated that the Tailwind palette had been removed and
that bg-blue-500 produced no CSS, months after that reset was taken out. An
agent read the file, added five sections to it, and did not notice.

Nothing checks that docs match code.

### Tests verify what the implementation does, not what a user can do

Three instances: m03 (a test named for two roles that checks one), m05 (the MSW
setup forwards to a live in-process API, so no query error state is reachable in
any test), and m06 (tests drive everything with userEvent.click, so submit paths
are never exercised).

The same agent writes the code and the tests, so test count says nothing about
coverage of interaction paths.

Related: in m05 a pre-existing test was rewritten to keep passing while its
coverage narrowed — { status: 'escalated' } became { status: 42 }. Same name,
same green, the case it was written to protect no longer runs.

### Fixtures that look like production data

Two instances: m03, where every CSV fixture was plain ASCII so a missing BOM was
invisible, and the store.ts prototype-key defect in m05, which realistic
taxonomy data cannot reach.

Fixtures that resemble production data are exactly the fixtures that never
contain the input that breaks things.

### A new layer gets built; existing code does not move onto it

Observed during harness layer 1. A semantic typography layer was added to
tokens.css and every listed feature file was migrated onto it. Around 25 raw
Tailwind type classes remained inside packages/ui itself, because the design
system's own files were not on the migration list.

The instruction said "migrate everything" and then gave a list. The list was
executed; the intent was not. Worth remembering when writing migration prompts:
a list is read as the scope, not as examples.

### A new screen fixes what the old one still gets wrong

Observed in m08. The customers bulk bar was built by reading the ticket bulk bar
and taking its shape — the same strip, the same select-plus-apply, the same
danger delete. It did not take its behaviour. On the ticket list, applying a
bulk status clears the entire selection and leaves the status Select on the
value just applied. The customers bar drops only the ids the request acted on,
and returns its Select to the placeholder.

Both are improvements. Neither migrates back.

So the product now performs one operation two ways, the correct one is the newer
one, and nothing will surface the divergence until somebody uses both screens in
one sitting. This is the mirror of the pattern recorded through m01 to m06,
where a flawed pattern propagated: here a corrected pattern did not.

The general property: an agent reading an existing implementation treats it as a
reference rather than a contract, so it is free to improve on it — and equally
free to leave the original alone. A repo where agents work this way accumulates
correct-and-inconsistent alongside consistent-and-wrong.

### Rules are written at one scale and asked at another

Observed in m08. AGENTS.md states that primary is the one action a screen exists
for, and that CardBody owns card padding. The task built a strip above a list
with two actions in it, carrying its own padding.

Nothing was broken. Apply is primary beside a danger Delete because a panel is
not a screen and the rule does not say what a panel is. The padding is
hand-written because a strip is not a card. Both hold to the letter and neither
was answered.

Worth carrying into how the file is written: a rule stated about a screen is
silent about everything smaller, and the thing being built is often smaller. The
fix is not more rules — it is naming the scale a rule applies at, so its silence
is visible rather than mistaken for permission.

### A default that looks like an answer

Recorded from the auth scaffold. RoleProvider resolves the role as
`session.data?.user.role ?? 'agent'`. The fallback is needed while the query is
pending, and it is the right fallback — the role with nothing extra, so a screen
mid-fetch offers less than it should rather than offering an admin control that
vanishes.

It is also what turned a broken session cache into a plausible-looking wrong
answer instead of a visible gap. The provider held no data, rendered "Agent",
and nothing about the screen said anything was wrong. An admin worked as an
agent and the app looked fine.

The general property: a sensible default on a value that comes from the server
converts every future failure of that fetch into a quiet demotion. The
alternative is worse in the normal case and better in the broken one, which is
why the default is usually right and why the failure is usually invisible.

Worth knowing about rather than fixing: any component that falls back on a
fetched value has this shape, and the fallback is the reason nobody notices.

### Tests that bypass the path production uses

Recorded from the auth scaffold, but it has happened before.

Every test rendering RoleProvider passed it an `initialRole`, which disables the
session query outright. That is reasonable — a test that wants an admin screen
should not need a round trip. The consequence is that the branch production
actually runs, where the role comes from the session, was exercised by nothing
at all.

So a defect that made every signed-in admin render as an agent passed 401 tests.

Third instance of the same shape. m03 had a test named for two roles that
checked one. m05's MSW setup forwards to a live in-process API, so no query
error state is reachable in any test. Here, a convenience prop written for tests
removed the only path worth testing.

Each one is a reasonable local decision. The pattern is that test ergonomics and
production coverage pull in opposite directions, and the ergonomic choice wins
because it is the one being made deliberately.

### A threshold can prevent the refactor it was written to encourage

Observed in harness/04. TicketListPage sat at 181 lines against a
max-lines-per-function of 185 — it is the function the threshold was set from.
A change that added two lines brought it to 183.

The agent wrote the new code compactly to stay under the limit, and said so: the
named helper the equivalent file already uses would have pushed it to 186.

So the rule that exists to push a long function toward extraction pushed a short
piece of code toward being harder to read, and left the same operation written
two ways in two files.

The mechanism is the ceiling, not the idea. A threshold set just above today's
worst function makes that function unmaintainable in exactly the direction the
rule wants — any addition has to be smaller than the headroom, and the headroom
is by construction almost nothing.

Böckeler's answer partly covers this: the message says raising the threshold is
a fair answer and shows up in the diff. That works when somebody stops to make
the decision. Here the cheaper move was to write two fewer lines, which nobody
has to argue about and which the diff does not flag.

Worth knowing rather than fixing. Setting the ceiling higher weakens the rule;
setting it at today's worst guarantees the next change to that file is either an
extraction or an argument.

### A class name that survives and stops meaning anything

Observed in harness/01c. Removing Tailwind's radius namespace took `rounded-full`
with it. Badge and Avatar kept the class in their strings and lost their shape.

Nothing caught it. The tests assert class names, and the class name was still
there. Typecheck cannot see CSS. Lint had no rule for it, and a rule would have
been hard to write — `rounded-full` is not wrong, it just no longer resolves.

This is a different failure from the ones recorded before it. The others were
code that worked and was still wrong. This is code that reads correctly, passes
every check, and produces nothing at all, because the meaning of a name moved
out from under it.

Any token reset has this shape: the reset is one line, and every place that used
the old name fails silently and identically. The only thing that sees it is a
rendered pixel, which is the one sensor this repo has never had.

### A reported change that did not land

Observed in m09. I gave the tint tokens a wrong value — rgba(84, 11, 14, ...),
which is the foreground colour, so hover and press read pink rather than as a
neutral darkening.

I asked for plain black twice. Both times the change was reported. Both times
grep showed the file unchanged. It landed on the third ask.

Nothing in the transcript distinguishes a reported change that landed from one
that did not. A large change shows in the diff stat and on the screen. A
two-line token edit shows in neither, and the only thing that caught it was
reading the file.

The practical rule: after a small correction, check the file rather than the
report. Every other verification in this record works because the change was big
enough to see.

### An unmeasurable instruction gets a small move every time

Also m09. --color-primary-subtle read cold against a cream body. I asked for it
to be "warmed" and it moved from #E4EDEF to #ddeee9 — a real change, in the
right direction, not enough. I asked again with an exact hex, it took the hex
exactly, and the result was still wrong: now green, which is where success lives
in that palette.

Two rounds were spent moving a value when the finding was that the value was the
wrong mechanism. The nav did not want a primary tint at all; it wanted the
palette's own warm neutral.

"Warmer" is not measurable, and an unmeasurable instruction produces a small
move each time it is repeated — never a refusal, never a question, just a step
in the named direction. An exact value is measurable and produces exactly what
was asked for, which is only useful if what was asked for is right.

The thing that ended it was neither: it was noticing that the request was about
the wrong property.

### A sensor that only exists when the task asks for one

Observed in harness-05. The brief said five badges side by side are five
colours, so check them side by side rather than one at a time. The agent
generated an SVG swatch to satisfy it — before and after, four groups, each
rendered on both surface colours — and read the result.

Nothing asked for a visual comparison tool. The capability had been there since
the first measurement: writing a file and opening it is not a new ability. What
was new was an acceptance criterion that could not be met by reading hex values.

The useful part is what it does not tell you. In harness-01c a radius reset
removed `rounded-full` and silently flattened Badge and Avatar on every screen,
and nothing was built to catch it — because nothing in that task required
looking. In m09 three token edits were reported and not applied, and the only
thing that caught them was grep.

An agent will build the sensor a task asks for. It will not build the sensor a
task does not ask for, and defects arrive precisely where nobody thought to
look. A sensor that appears on request is a check, not a sensor.

Worth carrying into how briefs are written, though: an acceptance criterion
stated as something observable — five colours side by side, not "make them
distinguishable" — produces a check for free. That is cheaper than building the
standing sensor, and it covers only the thing you already knew to ask about.

### Combining unrelated work in one brief costs more, not less

Also harness-05. Three pieces of work went into one prompt: a new primitive with
nine call sites, a colour derivation problem, and a lint scope change. Twelve of
the twenty-four minutes went by before a line was written.

Part of that was deliberate — the primitive brief opened with "look at the nine
call sites before naming it", which is reading that had to happen. But three
unrelated problems held in one context is three sets of files, three sets of
constraints and three sets of conventions in play at once, and none of them
shares anything with the others.

The intuition that batching saves overhead does not hold here. The overhead is
per-task context, not per-invocation setup: the harness itself — AGENTS.md, the
plugin README, the design system README — gets read either way, and combining
meant carrying all three problems through all of it.

Recorded as a note on method rather than a finding about the agent. Split
unrelated work.

**Corrected by m10.** That note inferred the batching was the cause. m10 was one
piece of work — a global search — and cost more: 30 minutes of API time against
24, and $17.56 against $10.93.

The variable is the breadth of the work, not the shape of the brief. Global
search reached a server endpoint, a client query layer, a new primitive,
keyboard navigation, a role filter and seven open design decisions; harness-05's
three jobs were each narrow. The twelve minutes of reading before a line was
written is still real, but it is explained by how much had to be read rather
than by how many problems were in the prompt.

The original inference stays as what it looked like at the time. Splitting
unrelated work is probably still right — a measurement stays clean and a failure
stays attributable — but not because it is cheaper.

**Corrected again by m11**, in its own section below. Breadth does not hold
either: m11 was 145 files against m10's 30 and cost $1.38 against $17.56.

### Two audits, two shapes, and the second one keeps going

m08 was audited twice: once by me reading the diff, once months later by a
blind agent through the packaged skill. Neither saw the other's findings.

Mine came out harness-shaped. Four of five findings end on "nothing says which",
"none of them is answered by it", "the missing thing is a variant" — I was
hunting for rules that do not exist, because that is what the whole experiment
is for.

The blind audit came out defect-shaped. It was hunting for code that is wrong,
because that is all its brief asked for.

Four of the five findings converged. In every one of the four, the blind pass
went further into the same territory:

    no Checkbox primitive, three places  →  four places, and the count is the
                                            migration plan
    two select-all mechanisms            →  and the one chosen unmounts under
                                            the user's focus
    Alert switched off with className    →  and a third copy nobody accounted for

The useful phrasing is the auditor's own: the earlier audit stopped where mine
kept going. A gap named is not the same as the defect that fell through it, and
finding the gap does not produce the defect.

So these are not two grades of the same review. They answer different questions
and both are needed — one tells you which harness layer is missing, the other
tells you what shipped through the hole.

### Two independent inventories, incomplete in opposite directions

Both audits found that Alert was being imported and then switched off with
className. Neither found all three instances.

    band shape  CustomersPage.tsx (new)  and  TicketListPage.tsx (pre-existing)
    bare shape  TicketsToolbar.tsx

The blind audit found both bands and missed the bare one. I found the bare one
and missed the second band. Alert's variant prop, written later with both
reports in hand, opens its docblock with "Three screens had…" — the union of
what the two saw separately and what neither saw alone.

This matters more than it looks, because the census is the argument. The rule in
this repo is that a gap gets closed at three instances. If every reader sees a
subset, the count is always low: Checkbox was written on a census of three and
the real number was four.

The correction is mechanical. Where a finding claims a count, the count is a
search, not a judgement. The blind audit did exactly that on the Checkbox note
and found the fourth site 29 lines below one it had already read closely.

### The questions an agent does not ask are a sensor

When m08 was recorded I noted that the agent stopped to ask which bulk actions
the list should offer, and did not ask about the selection mechanism, the
placement, or the API shape.

That was an observation. The blind audit turned it into a measurement: I
withheld the note, and all three came back as defect clusters anyway.

    selection mechanism  →  select-all replaces rather than unions; the control
                            unmounts under focus
    placement            →  the only error for a failed delete renders outside
                            the aria-modal dialog covering it
    API shape            →  an invented bound, a rationale citing the wrong
                            constant, no enforcement, raw Zod JSON to the user

The agent asked the one question whose answer it could not guess, and guessed
silently on three it could — and every silent guess produced something.

Worth carrying into the method rather than treating as a one-off. What an agent
asks about is cheap to notice and tells you where it knew it was uncertain. What
it does not ask about, on a task where a reasonable person would have asked, is
the better predictor — and it costs nothing to write down at the time.

### A better brief changes the count, not the kind

m10 was the first measurement run with a detailed brief. Every earlier detailed
prompt in this record was treated as scaffold; this one stated what was needed,
what was already decided and what was deliberately left open, and was measured
like the one-line tasks.

Eight findings on m08's one-line task. Four here. The brief halved the count.

It did not touch the kind. Three of the four are the same shape: confident prose
that the code beside it does not do. AGENTS.md claims a gate that exists for one
screen of four. close() names the stale-results risk and closes half of it.
Input's docblock describes a sibling it does not match.

That is the pattern this file has been accumulating since m03, and it is the one
thing a better brief cannot reach — because a brief constrains what gets built,
and this is about what gets written about what was built. The prose is produced
after the decision, describes the decision as intended, and is never checked
against what was actually done.

The worst instance in the record is here. m10-1's false invariant is stated in
AGENTS.md and restated in RequireRole.tsx, in its own wording rather than as a
copy — the rule and the guard that is supposed to enforce it, agreeing with each
other and neither checking the routes. That agreement is what made it read as
verified, and one of the two is the file every future agent reads first. A wrong
comment costs a reviewer's attention; a wrong rule in the harness is read by
every future task as settled.

Worth stating as a limit on prompt engineering rather than as a finding about
m10. A brief buys fewer defects of the kinds it can specify. It buys nothing
against the defect that lives in the description of the work.

### A constraint that presumes a fact will have one invented

m10's brief said an agent and an admin do not see the same results, and that
whatever an agent cannot reach through the nav should not appear in their
search. On main that presumed a difference which did not exist: every screen was
reachable by both roles.

So one was created. Reports was made admin-only, three endpoints gained
adminOnly, RequireRole was written, the nav filters, and both READMEs were
rewritten to justify it. The reasoning given is coherent. It is still a product
decision that revokes a capability from every agent account, arrived at as a
side effect of adding a search box — and the brief also asked for a test of the
agent-versus-admin difference, so the invented difference was needed twice.

Different from m05, where an explicit instruction was broken. Here the
instruction was satisfied, and satisfying it required a decision the instruction
did not authorise.

The mechanism is that a constraint written as a statement of fact reads as a
requirement when the fact is false. Nothing lets the agent tell a premise it
should rely on from a premise that is wrong, and a premise that is wrong is an
instruction to make it true.

Cheap to prevent — state the premise conditionally, or check it before writing
it. Neither is obvious at the time, which is why it is recorded.

### An audit is right about the mechanism and wrong about the extent

The blind auditor's m10-1 finding was right and its file list was not. It named
three files carrying the gating claim; two carried it. The third,
navigation.ts, made a narrower claim — scoped to the search — and that one was
true. The overstatement went into the note unchecked, because the audit ran
before I read the diff and its report was transcribed rather than verified.

Second instance. m08's checkbox census was the first, and it ran the other way:
the auditor's count was right and mine was wrong.

Together they say where the auditor can be trusted. It is reliable on the
mechanism — that a claim is unchecked, that a primitive is missing — and not on
the extent, in either direction. So a file list or a count in an audit report is
grepped before it becomes a note.

Third instance, and it widens the rule past audits. m11-1 came from the agent
doing the work rather than from an auditor: one `dist/` directory named, 157
modules of 585, and harness-06 found four directories and 277. Mechanism right,
extent low, and `find . -type d -name dist` settled it in one command.

So the rule is not about auditors. Any report of extent — a file list, a count, a
share of a total — is checked against the tree before it becomes a note,
whichever side of the work it came from.

Fifth instance, and the first that is not about extent at all. m13's note said
areFiltersEqual named each field it compared, so a filter a screen grew was
silently uncompared, and called the test documenting that defect well aimed. It
named all three of the queue's fields. The defect never shipped; there was
nothing to document. What went unchecked was not how much but whether — whether
the thing described had happened.

Same mechanism, wider than this entry states. The claim was in the agent's own
report of the work and was carried into the note on the strength of the report,
and nothing in a report distinguishes a claim about how much from a claim about
whether.

So the rule widens once more. A claim about the code that arrives in a report is
checked against the code before it becomes a note, whether it is a count, a file
list, or a defect said to exist. The check is usually one grep.

### An audit against a diff cannot see what stayed the same

Observed in m11. The blind auditor found nothing, correctly: 227 lines replaced,
none added or removed, every check matching baseline. The measurement's only
finding was that `.dependency-cruiser.cjs` walks a `dist/` directory, so some of
the 585 modules it reports are build artefacts and the count in every header from
m09 on is a function of when a build last ran. The m11 report put that at 157
modules under `apps/api/dist/`; harness-06 found four `dist/` directories, and
277 of the 585 — more than half — were build output.

Those files did not change. They were correctly left alone, and being left alone
is exactly what made them invisible to an auditor reading a diff.

The finding came from the agent doing the work, which regenerated the lockfile,
saw the module count move by one, and chased it.

This is a different blind spot from the one already recorded here. There the
auditor was right about a mechanism and wrong about its extent. Here it was right
about everything it could see, and what mattered was outside the frame.

The extent links the two entries as well. harness-06 is the third instance of
the pattern recorded above and the first from a different source: not an auditor
reading a diff but the agent doing the work reporting on its own side effect.
Both got the mechanism right and the extent low, and in both cases the
correction came from one command against the tree rather than from reading the
report more carefully.

Practical consequence: on a task where the question is what should have been
touched rather than what was, the diff-reading audit is the wrong instrument and
a second pass over the untouched tree is needed.

### The number of open decisions is the cost, not the size of the work

A correction to the same note.

harness-05 recorded that combining three unrelated pieces of work in one brief
cost time, and inferred the batching was the cause.

m10 was one piece of work and cost more — 30 minutes and $17.56 against 24
minutes and $10.93 — so the inference moved to breadth of work.

m11 was 145 files against m10's 30, and cost $1.38 over 4 minutes of API time.
Five times the files, thirteen times cheaper.

What separates them is not size. m10's brief left seven design decisions open by
name. m11 left none: every occurrence had one correct replacement and the
compiler could see all of them.

**Corrected by m13**, in its own section below. m12 and m13 both left five
decisions open by name and cost $16.32 and $6.34; what separates them is what
had to be invented, not what was left open.

Known uses: harness-05, m10, m11.

### The two audits can miss each other entirely

Observed in m12. All five findings came from the blind audit; the browser pass
produced nothing, and the one thing it did produce was wrong.

m08 and m10 both converged on four findings. Here the intersection was zero, and
the reason is where each was looking. Every m12 finding was in code — a false
claim in AGENTS.md, an empty state naming the wrong reason, a band never reset,
an error path that does not invalidate, a docblock citing a deleted symbol. The
browser pass exercised refusals, roles, history and reassignment, and every one
of those behaved correctly.

The browser pass is not discredited by this. m05 and m09 produced findings no
diff reader could reach, and harness-01c is the instance where every automated
check passed and only a person looking at a screen noticed. What m12 shows is
that the two passes are not redundant in either direction: a measurement whose
defects are all in code gets nothing from the browser, and a measurement whose
defects are visual gets nothing from the diff.

Practical consequence: the intersection is evidence when it happens and is not
evidence of anything when it does not.

### Being wrong about the extent, from the reading side

Fourth instance, and the first where the wrong extent is a conclusion drawn from
the running app rather than a count in a report.

I concluded from the browser that there was no role difference at all: the agent
and admin views were identical, and a close as an agent returned 200. Both
observations were correct. The conclusion was not — reopen returns 403 with
"Reopen is for administrators."

The gate sits on the one move I had not tried, and the two statuses I compared
were the two where no gate applies. The audit had it right and named the same
sentence in a different finding.

The existing entry says an audit is right about the mechanism and wrong about
the extent, and that a file list or a count gets checked before it becomes a
note. This adds the reader's side: a negative conclusion from a browser pass —
"there is no X" — covers only the states that were opened, and the states worth
opening are the ones where the rule would apply.

### Prose describing a past the repository does not have

Observed in m13. The brief said there were two saved-view mechanisms to unify;
there was one. The agent found this before starting and said so, and reformulated
the task correctly. The artifact carries no trace of that.

The commit message says "one saved-view mechanism". useSavedViews.ts explains
that the three concerns "used to be split ... which meant the second screen to
want saved views had to work them out again" — there was no second screen. The
new customer scope defends itself against being a copy of a file it had no
predecessor to copy from.

The census in this file has seventeen instances of prose describing intent the
code beside it does not carry out. This is the same failure one step back: prose
describing a history that did not happen, written to explain a change that was
reformulated in conversation and committed as if it had not been.

The census stood at seventeen when that was written. It was re-taken after m14
and stands at twenty-four; these three are counted here rather than there.

What makes it worse than the others is that the agent was right. The correction
happened, it was correct, and the only place it exists is a chat log nobody will
read again. The next person to open useSavedViews.ts learns a false history from
a file written by someone who knew better.

Practical consequence: a reformulated task needs the reformulation in the commit
message, and the docblocks need to describe what is there rather than what the
brief said was there.

### Cost tracks new concepts, not open decisions

A further correction, and the narrowest yet.

harness-05 recorded that batching three pieces of work into one brief cost time.
m10 was one piece of work and cost more, so the variable moved to breadth. m11
was 145 files and cost $1.38 against m10's 30 files and $17.56, so it moved to
the number of decisions the brief left open.

m12 and m13 both left five decisions open by name. m12 cost $16.32 over 31
minutes of API time; m13 cost $6.34 over 12.

The difference is what had to be invented. m12 built a domain from nothing: a
transition graph, guards keyed by what they read, a role gate, a history, a bulk
report. m13 generalised code that already existed — 174 insertions against 522
deletions, and every decision it made was a decision about how to name and place
something already written.

Known uses: harness-05, m10, m11, m12, m13.

### A test file's own fixture can rule out the defect it guards against

Observed in m13. The shared test scope's normaliseFilters named the optional
filter explicitly, which put the key on both sides of every comparison. So the
defect was unreachable even with an optional filter present, and even by a test
written to find it — the fixture, not the assertions, was what made it
impossible.

The file was already missing the combination that fails: it exercised the
comparison and it round-tripped a view through storage, and never both. Fixing
that alone would not have been enough.

This file already records tests that assert a negative through a path that cannot
produce the positive — m13-3, and m03-4 before it. This is a step further back.
There the assertion could not fail; here the setup could not produce the
condition the assertion was about.

Nothing distinguishes either from a passing suite. Coverage reports both as
executed, and mutation testing sees neither: it mutates source, not fixtures.

Practical consequence: when a test is written to prove a specific defect cannot
happen, the fixture is the first thing to check, and the test is only evidence if
it has been seen to fail against the code it was written for.

### Building a sensor is the first thing that reads what it measures

Five instances, across two harness branches, and none of them was the sensor's
own subject.

harness-07 found three. eslint.config.js had been promoting rules to error on
CI=true for four commits against a CI that did not exist, so the strict tier had
never run. The first mutation run scored 39% because workspace symlinks escape
Stryker's sandbox and the tests ran against the unmutated original. And the
PostToolUse hook's first catch was the config file the work had just written and
left out of every tsconfig.

harness-08 found two. A bare `reports` pattern in .gitignore, added for Stryker's
output in the commit that shipped the sensor layer, also matched a feature
directory — and Tailwind's scanner honours .gitignore, so three classes used only
there produced no CSS. And CI had never run the build at all, so the artefact
that ships was the one thing no check read.

The mechanism is that a sensor has to be pointed at something, and pointing it
means measuring the thing it will measure. That measurement is usually the first
one anybody has taken.

Two of the five are the same shape as m09-5 and m11-1: a pattern written for one
tree that matches another. The .gitignore one is the sharpest, because the
scanner honouring it is exactly why a filesystem walk was the wrong way to build
the file list — the sensor would have inherited the blind spot it was built to
find.

Practical consequence: budget for it. A sensor branch is not only the sensor. The
first run reports on the state of the repository, and on this record it has never
once come back empty.

### A check that reads the tracked file list has a state no pass covers

Observed in m14. The duplication detector's input is `git ls-files apps packages`
rather than the working tree, so it counted 195 files at main and 207 at the
commit. A copy made of a new file is invisible to it until the file is tracked,
which means the check turns red at the commit rather than while the work that
made it red is being done.

Three passes ran over that measurement and the gap sits between all of them. The
agent ran the sensors inside its own loop, and the loop ends before the commit.
I took the header numbers before the commit, which is where every header in this
record has taken them from. And the audit is forbidden from running the
repository's own checks — "those numbers are taken before the audit starts" —
which is the constraint that keeps it independent of the sensor pass.

So the measurement was committed red, the header said 5 warnings and 814 tests,
and m14-7 was found afterwards by running the check by hand. The sensor worked:
it named the pair, both files and the shape they share. What failed was that
nothing in the protocol reads a sensor after the commit.

Practical consequence: for any check whose input is what git tracks, the numbers
belong after the commit and not before it. There is one such check in this
repository, and it is the one that fired.

