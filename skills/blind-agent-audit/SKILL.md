---
name: blind-agent-audit
description: Run a blind audit of work a coding agent produced on a git branch — read the diff against a base, judge it against the exact task prompt that produced it, and surface findings that tests, typecheck, lint and human review would all miss. Use this whenever the user wants to audit, check, grade or sanity-check what an AI agent (Claude, Codex, Cursor, a background or async agent) actually did on a branch, asks whether the agent did what it was told, suspects that all-green code is still wrong, or asks for an audit rather than a review for merge. Prefer it over ordinary code review whenever the work being examined was produced by an agent from a known prompt.
---

# Blind agent audit

An agent was handed a task prompt and produced work on a branch. This skill audits that work — not "is this good code for merge", but "does this do what was asked, and where is it quietly wrong". The most valuable finding is code that works, passes every check, looks right, and is still wrong.

The audit is read-only. Nothing in the repo is edited, and no report file is written; the report goes in chat.

## Step 1 — get all three inputs

Ask for, and do not start without:

1. **Branch** — where the work is.
2. **Base** — what to diff against.
3. **The task prompt that produced the work, in full.**

For branch and base you can propose what you detect (`git branch --show-current`, the repo's default branch) and have the user confirm.

The task prompt is different: **stop and ask for it, and do not proceed until you have it.** Half the judgement in this audit is whether something was asked for or invented, and there is no way to make that call without the instructions the agent was working from. A summary will not do — the omissions and asymmetries live in the details. And never reconstruct it from commit messages, a PR description or the diff itself: a reconstruction is derived from the output being audited, so it can only ever agree with it.

If the user doesn't have the prompt, say plainly that this audit can't be run without it, and offer an ordinary code review instead.

## Step 2 — keep the audit blind

**The auditor must not have seen findings from any earlier audit of this work.**

This is the property the whole exercise rests on. If a pattern turns up in an audit that was told about it, nothing has been learned — the auditor went looking and found it. If it turns up in an auditor that had never heard of it, the pattern genuinely recurred. Priming destroys the only thing that separates those two cases.

In practice:

- If this session has already seen audit output for this branch — its own, or pasted in by the user — or if this session wrote the code, don't audit here. Spawn a fresh agent (the Agent tool, `general-purpose`; **not** `fork`, which inherits your whole context) and let it run the audit.
- When you're unsure whether your context is clean, delegate. A fresh agent on an already-clean session costs one extra hop; a contaminated audit looks exactly like a good one and is worth nothing.
- Give the auditor only the three inputs and the brief. Never pass along earlier findings, a summary of them, the user's suspicions, or a hint about what to check. Even a helpful-sounding "see whether the naming thing happened again" ends the audit's usefulness.
- If the user offers previous findings up front, decline them for this run and say why. They become useful in step 4.

## Step 3 — run the audit

The auditor's brief is `references/audit-prompt.md`, next to this file (user-scope install: `~/.claude/skills/blind-agent-audit/references/audit-prompt.md`). It is the contract for what counts as a finding, how to work, and the output format. Either follow it yourself exactly, or hand its path to the fresh agent along with the branch, the base, and the task prompt verbatim — substituting for `<BRANCH>`, `<BASE>` and `<TASK PROMPT>`.

Practical notes for whoever runs it:

- Read the real diff, never file names or a summary of what was done. If the branch isn't checked out, `git diff <base>...<branch>` reads it without switching. `git status` catches work that was never committed.
- Don't run the repository's own checks — see below. If the user supplies their results, report what they say; "all green and still wrong" is the point of the exercise. Nothing in the repo changes: source, config and test files must come out unchanged, and a throwaway probe lives in a scratch file outside the repo.

### The audit does not run the repository's own checks

**The auditor does not run typecheck, lint, tests, boundary checks, duplication detection, mutation testing, or any other check the repository already ships.** Those numbers are taken before the audit starts, and producing them is not what an audit is for.

Three reasons:

- The audit's value is that it is a second independent reader. A finding derived from a check the user has already run is not independent — it is the same source read twice, and it makes the intersection between the two passes look larger than it is.
- Checks are slow. A mutation run can take an hour on a mid-sized repository, and the audit is not the place to spend it.
- It makes audits inconsistent with each other. One audit runs the checks, the next does not, and the difference in findings has nothing to do with the work.

What the auditor may still do: read what those checks produce, if the user supplies it; and write and run a throwaway probe to test a specific hypothesis about the code — copying a function into a scratch file and driving it, for instance. That is not the same thing. A probe answers a question the auditor formed from the diff; the repository's checks answer questions somebody else already asked.

If you delegated, relay the report to the user **in full and unedited** — subagent reports aren't shown to the user, so anything you drop is lost. Don't re-rank it, compress it, or quietly remove findings you disagree with. Put your own commentary after the report, clearly marked as yours.

## Step 4 — only now, compare with earlier audits

With the report in hand, comparing it against previous audits of the same work is legitimate and valuable. A finding two independent blind audits reached separately is far stronger evidence than one either produced alone. Say explicitly which findings repeated and which are new.
