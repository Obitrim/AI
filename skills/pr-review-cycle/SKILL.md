---
name: pr-review-cycle
description: Paul's PR review cycle, from pinned head to posted inline comments. Use when Paul asks to review a PR, to QA or production-filter review findings, to draft inline review comments, or to re-review a PR after its author pushes.
---

# PR review cycle

Each finding has to **survive** every step before it reaches the PR. The cycle has two **gates** where you stop and wait for Paul: after the report (step 6) and after the drafts (step 8). Enter at the step Paul names; a request to draft or filter starts from the findings already in the conversation, after repeating step 1.

A finding's **evidence** is one of:

- **Reproduced**: you ran it and saw the wrong result.
- **Read**: you traced the code and ran nothing.

A problem is **in scope** when the PR introduces it, makes it worse, or leaves its linked issue unmet. Every other problem is a **possible new issue**: it gets reported to Paul and stays off the PR.

## Cycle

### 1. Pin

Record the head SHA, the merge-base with the target branch, the author, the linked issue and its labels, the existing reviews and threads, CI status, and the open PRs that touch the same code. Work in copies of the head and the base outside Paul's working tree.

Repeat this step before drafting and again immediately before posting. When the head has moved, read the new commits and re-run every finding on the new head: some will be fixed, some narrowed, and the new commits can add their own.

Done when: both SHAs are recorded and both copies exist.

### 2. Review two axes

Run the `mattpocock-skills:code-review` skill: Standards and Spec as parallel read-only sub-agents. Give each the SHAs, the copy's path, the full issue text, and the standards files. Read the risky code yourself while they run: migrations, money postings, webhooks, locks, permissions.

Done when: both reports are in and you have read every changed migration and write path.

### 3. Reproduce

Run each finding on the base and on the head. The base result is what makes it a regression, so name the base by its SHA. Run each migration on an empty database and on seeded data.

Apply each suggested fix in a scratch copy, re-run the reproduction, then run the affected suite. Check the fix against the open PRs from step 1 too, since a fix that works on this branch can break on theirs. A suite that fails under parallel load gets re-run alone, with and without the fix, before anything is blamed.

Done when: every finding has its evidence label, and every suggested code change has run or is labelled untested.

### 4. QA

Try to disprove each finding. Look for a database constraint, trigger, guard, caller or existing test that already prevents it, and for documentation that makes it intended behaviour.

Then sort by scope. The base run from step 3 decides: the same result on the base and the head, with the linked issue not asking for it, makes it a possible new issue. Search the existing issues and open PRs for each one, so a known problem is linked instead of reported as new. Unrequested work inside the PR's own diff is in scope, as a Spec finding.

Done when: every finding is kept, narrowed, moved to possible new issues or dropped, with a reason for each move and drop.

### 5. Filter by environment

Establish from the repo what each environment runs: which adapters or providers register, what data exists, how migrations and rollouts happen, and whether production has ever been deployed. For each finding and possible new issue, name the traffic or staff action that triggers it there.

| Result        | Meaning                                                                           |
| ------------- | --------------------------------------------------------------------------------- |
| Stands        | Real traffic or a normal staff action triggers it in production                   |
| Go-live gate  | Nothing fails today, but merging marks a control as done before it exists         |
| Conditional   | Needs data or a setting you couldn't see; give the query or check that settles it |
| Doesn't stand | Dev only, sandbox only, or no runtime effect                                      |

Dev data can be dirty, so evidence for a comment comes from clean seeded data. Set severity after this filter, from the production impact.

Done when: each of them has a trigger, an impact and one of the four results.

### 6. Report, then stop

Write the report below. **Gate: wait for Paul.** Answer follow-ups such as "does this affect staging?" by measuring where you can, and correct earlier claims openly.

### 7. Draft the comments

Draft in-scope findings marked Stands or Go-live gate. Hold back Conditional ones, listed with what would release them. Then review each comment on its own:

- the path and lines are in the current diff, and the code there is what the comment describes
- quoted errors, docs and issue text match the source exactly
- every number and "before/after" claim comes from a run, on the SHA it names
- the trigger and its limits are stated, including when the case can't happen today
- the fix is the version that passed, and covers every case the comment lists
- the severity matches the filter's result
- all four parts are there, and every pointer opens the code it names
- the problem is in scope, and the fix asks for nothing beyond it

Done when: each comment passes every line above, with the payload's anchors checked by script.

### 8. Show the drafts, then stop

Show the review summary and every comment in full, what changed since the last report, what was held back, and the account it will post from. **Gate: post when Paul says to post.** "Write", "draft" and "prepare" mean stop here.

### 9. Post and verify

Re-pin. When the head has moved, return to step 1 and tell Paul. Otherwise submit one review with all comments inline, pinned to the head SHA, so each can be resolved on its own. Read it back: state, comment count, lines. Give Paul the link to the review and to each comment, with anything still open.

A correction after posting edits the original and says what changed.

## The four parts

Every finding carries all four, in this order, wherever it appears: report, judgement call, possible new issue, follow-up answer or inline comment. A finding missing a part goes back to the step that produces it.

1. **Problem**: what triggers it and what goes wrong, with a base/head table when results differ.
2. **Where**: a clickable pointer to every place involved, as the file and line range at the head SHA. Name the line to change first, then the cause and any other sites that need the same change. For something missing, such as a test, point to where it belongs. For a migration or doc, point to the lines in that file.
3. **Suggested fix**: the concrete change at those lines, as code when short.
4. **Why this fixes it**: how the change removes the cause given in Problem, and the result seen with it applied (or "untested"). Add why it beats the obvious alternative when there is one.

## Report format

Lead with the verdict and the result count, then the scope: both SHAs, what was run, what was only read. Keep Standards and Spec under separate headings, each ranked on its own, ending with one line giving the count and worst finding per axis.

Each finding:

> **S1. 🔴 One-line claim (Reproduced)**
> **Problem:** …
> **Where:** [`file.ts:10-20`](link) (change here), [`other.ts:44`](link) (cause)
> **Suggested fix:** …
> **Why this fixes it:** …

Link to the code on GitHub at the head SHA (`/blob/<sha>/<path>#L10-L20`), so the pointer still lands on the right lines after the author pushes. A summary table can come first, but it never replaces the four parts.

Severity: 🔴 blocking, 🟠 should fix, 🟡 judgement call. Number findings once (S for Spec, T for Standards) and keep the numbers for the whole review.

After the findings, list **Possible new issues** in their own section, left out of the verdict, the counts and the severity ranking. Number them N1, N2. Each one has a suggested issue title, the evidence label, the four parts, the environment result, and the existing issue or PR if there is one. A serious one is called out here as serious and still stays off the PR. Offer to open them; create issues only when Paul says so.

Close with: dropped in QA and why, corrections to earlier answers, what you couldn't check, and what you left behind (scratch copies, databases). Use short sentences, plain words, and tables for comparisons.

## Comment format

The audience is the PR's author: no S/T/N numbers, no possible new issues, no mention of these steps. One problem per comment, at the line to change.

> **Blocking: the claim in one sentence.**
>
> **Problem:** … Say when it can't happen yet and what will make it possible.
>
> **Where:** this line, plus links to the cause and other sites
>
> **Suggested fix:** …
>
> **Why this fixes it:** …

Labels: **Blocking**, **Should fix**, or **Before closing #N** for a go-live gate. Frame a design choice as a decision for the author.

The summary names what improved, the SHA reviewed and what was run, which comments block, and any PR that needs a merge order. Submit as `REQUEST_CHANGES` when any comment blocks, otherwise as `COMMENT`; approve only when Paul says so.

## Running reproductions

- Give each run its own throwaway database, and drop it after. Measure against real data on a copy, with the original opened read-only.
- Read credentials at run time inside the script, unprinted, and keep them out of files.
- Put commands in a script run with `bash`: zsh fails on unmatched globs such as `--include=*.ts`.
- Delete with literal absolute paths. When a removal is refused, leave it and tell Paul where it is.
- Temporary git refs and worktrees go when the review ends. Confirm Paul's working tree is unchanged.
