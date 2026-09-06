# Project: agent harness experiment

## What this is

A measurement experiment on `support-desk`, a support-ticket admin panel with
its own design system. I do not write the feature code. I write the task, the
agent builds it, and I measure what it got wrong and which layer should have
caught it.

The question: as a codebase grows, which mechanisms actually constrain a coding
agent, and which only appear to.

## The harness model

- **Layer 1 — primitives.** Components, tokens, variants, utilities.
- **Layer 2 — computational guides.** Lint rules, complexity limits, module
  boundaries.
- **Layer 3 — AGENTS.md.** Only decisions no tool can check. Short. Split into
  conventions (fixed) and product decisions (changeable).
- **Layer 4 — sensors.** Hooks, mutation testing, accessibility checks, CI.

## Measurement protocol

1. Branch `measure/NN-name` off main. Prompt saved to `mNN-prompt.txt`.
2. Prompt states the requirement, not the solution. If it names the library and
   the layout, the measurement is void.
3. Agent runs. No intervention beyond answering its questions.
4. Numbers: typecheck, lint, tests, boundaries, `git status`. Test count must
   not drop.
5. Usage: API time, wall time, cost.
6. Blind audit **before** I read the diff. Order matters — once I have read it I
   cannot brief the auditor without leaking my hypothesis.
7. My own read: exercise it, then read the diff from the design-system side.
8. Compare: intersection, auditor only, me only. The intersection is the
   strongest evidence the method produces.
9. Record.
10. Commit, keep on the branch. Do not merge by default — merged defects get
    paid for daily.

Steps 6 and 7 are separate because the two audits ask different questions: mine
looks for harness gaps, the packaged skill looks for defects. Both are needed.

**The agent works inside the repository.** Every prompt confines the agent to
the subject repo — no sibling directories, no notes, no other repos. The notes
describe what earlier measurements found, and an agent that reads them is not
measuring, it is recalling. This happened once, on m13, and was caught in the
session.

**The branch is clean before the numbers and before the audit.** Tools leave
things behind: Stryker instruments source in place and does not restore it if
the run is interrupted, and its Vitest runner writes a setup file per worker to
the repository root. Numbers taken from a dirty tree are wrong, and an audit
reading one attributes the residue to the work. My own edits — a .gitignore fix,
a config line — go on main, never on the measurement branch.

**Say whether the agent ran the sensors.** From harness-07 the gate runs checks
in the agent's own loop, so an agent now corrects itself before finishing. That
did not exist for m01 through m13, and it changes what a finding count means.
The header records it, and a measurement compared against an earlier one says
so.

Classify before recording: harness gap, rollout problem, product decision,
documentation, or badly specified target. Only the first two change the harness.

## What this protocol does not track

The protocol above has changed while the experiment ran. The blind audit
arrived around m08 and the packaged skill after it, so a file written before
either was produced under a different method than one written after. No header
records which protocol state a run belongs to, save the one field for whether
the agent ran the sensors — the change that most affects what a finding count
means.

Measurements from different protocol states are therefore not directly
comparable on finding count.

## Scaffold or measurement

A scaffold is built from a detailed spec to give later measurements something
to run against, so the conditions are not controlled and the prompt style is
never the variable.

## The notes

Measurement files and their prompts are in `measurements/`, scaffold files in
`scaffolds/`, harness work in `harness/`. README.md, METHOD.md and
observations.md sit at the root.

**`README.md`** — the result. The measurements, what each one showed, the
conclusions that hold across all of them, the root causes, and the index of
files. Everything else is reached from here. A note nobody rereads is worth
nothing, so it is kept current or the rest decays.

**`mNN-name.md`** — one file per measurement. Header carries task, branch,
harness state, prompt style, baseline, codebase size, files touched, lint,
whether the agent ran the sensors, duration and cost. Findings numbered per file
(`m10-1`) so earlier numbers never shift. Each finding ends with:

    Caught by: ...
    Layer: sensor missing | rule broken | rollout | not harness — documentation |
           not harness — badly specified target

Then: what was done well, what was not asked for, what this measurement shows.

Every measurement and scaffold file carries a Codebase line under its baseline,
giving the size of the repo the run happened in. The test count is exact, taken
from the run recorded in that file. The screen and source-file counts are
estimates — that is what the tilde means, and it is said here rather than on
every line.

The Harness state line names which layers were in place when the run happened,
so a run can be read against the repo it actually happened in.

**`observations.md`** — the pattern catalogue. A finding belongs here once it
has appeared in more than one measurement and the mechanism can be named
independently of the feature it appeared in. One instance is an observation.
Two or more is a rule. Each entry: a name, the mechanism, a *known uses* line
linking back to the measurement files, and whether it has generalised. This is
the layer that turns fifteen files into ten readable patterns, and it is where
a finding becomes evidence rather than an anecdote.

**`scaffolds/scaffold-notes.md`** — work that built the app rather than measured it.
Workspace setup, API layer, aliases, migrations, anything driven by a prompt so
detailed that the agent had no decisions left. Kept because it explains how the
baseline got its shape, but never counted in a measurement.

The distinction matters more than it looks: a finding filed as a measurement
when it came from scaffold work inflates the count, and one filed as scaffold
when the agent actually decided something loses evidence.
