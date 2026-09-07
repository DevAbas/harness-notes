You're auditing an AI agent's work, not reviewing code for merge. Read this
framing first — it changes what counts as a finding.

This is a read-only audit. Do not modify any file in the repo.

## What I want you to audit

The work is on branch <BRANCH>, diffed against <BASE>. Read it with `git diff`
and `git status`.

This was the task prompt that produced it, in full:

<TASK PROMPT>

## What counts as a finding

The most valuable finding is code that works, passes typecheck, passes lint,
passes tests, looks right in the browser, and is still wrong. Prioritise those.

Look for:
- A rule in the project's own documentation that was broken
- A primitive that exists and was hand-rewritten instead of imported
- A missing primitive hand-assembled, and assembled differently in two places
  within this same change
- The new work diverging from existing work on something the existing work
  already answered
- Accessibility present in one place and absent in the equivalent one
- Tests that pass while missing the case that matters, or test data unlike real
  data
- A comment or doc that contradicts the code next to it
- Logic bugs no sensor covers
- Asymmetries: a check applied on one side of a boundary and not the other
- Regressions in files that existed before this task

Explicitly NOT findings:
- Style preferences with no consequence
- Anything the task prompt itself asked for
- A decision the prompt did not specify and nothing in the repo answers. If the
  codebase gives no precedent either way, the agent choosing one option is not
  a finding. Only record a deviation when something already in the repo answers
  the question and the output departs from it.
- Problems that predate this change and were only passed through. Check the
  diff; if the file was not touched by this task, it is not this audit.

## How to work

Read the actual diff. Do not judge from file names or from any summary of what
was done.

Do not run typecheck, lint, tests, boundary checks, duplication detection,
mutation testing, or any other check this repository already ships. Those
numbers were taken before the audit started. Running them again is slow, it is
not a second independent read — it is the same source read twice — and doing it
in some audits and not others makes audits incomparable. If the results were
supplied to you, report what they say: "all green and still wrong" is the point.

You may write and run a throwaway probe outside the repo to test a specific
hypothesis — copying a function into a scratch file and driving it, say. A probe
answers a question you formed from the diff; the repository's checks answer
questions somebody else already asked.

For each finding, answer honestly: would tests have caught it? typecheck? lint?
a human reviewer? If the answer to all four is no, say so — that is the finding
worth having.

Be sceptical of your own first read. Where the code is good, say it is good; a
padded list is worse than a short one. Four real findings beat twelve where
eight are filler.

## Output

Report in chat. Write no files.

    ## Finding N — <short title>
    <what it produced, concretely, with the relevant code>
    <what is wrong and why it matters>
    Caught by: <tests / typecheck / lint / human review / nothing>
    Layer: <guide missing / sensor missing / rule broken / not harness>

Then, separately and briefly:
- What was done well
- Anything the agent did that was not asked for
- Anything you looked at closely and found nothing wrong with, so the coverage
  is known
