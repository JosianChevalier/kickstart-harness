---
name: commit
description: Making a commit, or cutting work into commits. One reviewable change per commit, a Conventional Commits subject, a body that says why.
---

# Commit

## Doctrine

- One reviewable change = one commit. A commit is small: a unit of thought, a quick functional change or a quick refactor.
- A commit has two readers. The author follows what they can. The reviewer knows backend and reads deeper. Cut the commits for the reviewer: several commits may sit below the author's level.
- Every commit leaves the branch working, its tests passing. The tests come in the same commit as the change they prove.
- The body says why. It also says how, when the how is complicated.
- The work happens on a branch, not on trunk.
- When the user validates a commit, it lands: the validation and the landing belong to the stage, not to this skill. This skill says what a commit holds and how to make it.

## What counts as one change

- A step of a stage, or of a plan.
- A bug fix.
- A quick refactor.
- Adding or updating a rule or a skill.
- Removing dead code.

## What does not count as one change

- A feature and a refactor that touch the same file.
- A bug fix and a cleanup you saw near it.
- A step and the removal of a TODO comment.

When you see something else to change while you work (a refactor, a cleanup, a comment to remove, a TODO to update), do not put it in the current commit. Make it a separate commit, or note it as its own step: in the plan, or in the notebook when there is no plan.

## Red flags

- "While I am here, I will also…": stop. Separate commit.
- "It is small, I will include it": small is not the same concern.
- "It touches the same file": the same file is not the same change.
- "I will add the tests later": no. The tests come with the change.
- "It works, I will clean it next": if the branch does not work after it, do not commit.

## The message

```
<type>: <subject>

<body: why; and how, when the how is complicated>
```

The subject follows Conventional Commits:

- `feat:` a new behaviour. `fix:` a bug fix. `refactor:` a change of structure, behaviour kept. Also `test:`, `docs:`, `chore:`, `build:`, `ci:`, when the change is only that.
- A scope is allowed when the repository uses scopes: `feat(booking): …`. Read `git log --oneline -20` and follow the types and scopes the repository already uses.
- Imperative mood: "add", "fix", "rename"; not "added", "fixes".
- No period at the end.
- About 72 characters at most.
- Describe the change, not the file.

The body is separated from the subject by a blank line. Write it for the reviewer: the reason the change exists, the choice it makes, and the cost of that choice when there is one.

## Making the commit

1. Assess. Run `git status` (never with `-uall`), `git diff`, and `git log --oneline -5`. If there is nothing to commit, say so and stop.
2. Stage the files by name. Never use `git add .` or `git add -A`. Never stage a file that looks like a secret (`.env`, credentials, keys). Never stage changes that are not part of this commit: another person's work, another agent's work, unrelated edits.
3. Read the staged diff (`git diff --staged`). It must hold exactly the one change, with its tests. If it holds more, unstage the rest.
4. Commit with `git commit -m "<subject>" -m "<body>"`.
5. If a pre-commit hook fails:
   - It reformatted files: stage them again and make a NEW commit. Never amend.
   - Any other failure (tests, compilation, lint): the commit did not happen. Report the output. The change does not work yet: fix its cause inside the same change, then commit again. If the fix needs a choice the user did not make, stop and bring the choice to the user.
6. Report the commit hash and the subject.

## Rules

- Never amend, unless the user says so. Always make a new commit.
- Never push, unless the user says so.
- Never stage with `git add .` or `git add -A`.
- Never skip the hooks (`--no-verify`).
