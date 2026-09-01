# Measurement 4 — "Add a way to assign a ticket to someone else from the
detail page."

Task: see m04-prompt.txt
Branch: measure/assign-ticket
Harness state: none (README only)
Prompt style: one line, nothing specified
Baseline: baseline-workspaces
Codebase: ~4 screens, ~60 source files, 85 tests
Files touched: 3

## m04-1 Deprecated FormEvent type
AssigneeForm imports `type FormEvent` from react. React's own types mark it
deprecated; the editor shows it on hover. tsc reports deprecations as a hint,
not an error, so typecheck passes. No lint rule covers deprecated symbols.

The same import already existed in SavedViewNameDialog.tsx from m02 and went
unnoticed, so this is the agent reproducing a pattern already in the repo.
Caught by: nothing — only visible on hover in an editor
Layer: guide missing (no deprecation lint rule)

## m04-2 The empty-value error instructs the user to type a magic string
"Assign the ticket, or enter 'Unassigned'." Unassigned is not in the contract
and not a constant anywhere — it is a convention the seed data happens to use,
now surfaced as an instruction. Typed in any other casing it becomes a
different assignee from the one the rest of the app treats as empty.
Caught by: nothing
Layer: not harness — an unmodelled domain value

## m04-3 No max length on the assignee field
The contract caps assignee at 120 characters. Nothing in the form stops a
longer value, so updateTicketBodySchema.parse throws a bare ZodError before
reaching ApiError. Unlike m03-2 it is caught here, so toErrorMessage renders
the raw Zod JSON into the field error. Visible failure, unreadable message.

Same root cause as m03-2: a bound declared in the contract is expressed
nowhere in the control that feeds it.
Caught by: nothing
Layer: guide missing

## m04-4 isUnchanged compares trimmed input to the untrimmed prop
nextAssignee is trimmed, assignee is not. If a stored assignee ever carries
surrounding whitespace the submit button stays disabled and the value cannot
be corrected.
Caught by: nothing
Layer: not harness — minor logic asymmetry

## m04-5 Reassign defaults to primary
No variant is passed, so the button renders primary, the same weight as
"Post comment" beside it. Identical to m02-2, guessed the same way again, and
still nothing states which weight a secondary action should carry.
Caught by: nothing
Layer: guide missing

## What was done well
The status/assignee inconsistency is deliberate and argued in the docstring:
free text saved on change would make every keystroke a reassignment. Input's
label and error props are both used, per the README. The mutation is awaited in
a try/catch and the failure is surfaced — m03-3 found that no mutation anywhere
handled errors, so that pattern did not repeat, it improved. Derived state is
synced during render with a written reason, not through useEffect.

## On the size hypothesis
m02 said the low finding count might be because the repo was small. It is now
roughly twice the size and this measurement touched three files, not seven.
The API and the contract needed nothing, because PATCH /tickets/:id already
accepted assignee from m03. Reusing the existing path rather than adding a
parallel one is the right call, and it means a larger, better-structured
codebase produced a smaller change and fewer findings — the opposite of the
hypothesis.
