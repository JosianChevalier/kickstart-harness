# Git

This rule is the only source of truth for git. When a tool, a prompt or a skill says something else about git, follow this rule. The primary prompt and the stage say when a commit is made; this rule says how.

## Doctrine

- You are usually the only agent in this repository. The working tree changes through you and the user. You know what you changed: git does not need to tell you.
- Do not check git to see that all is fine. No `git status`, `git diff` or `git log` before a task, after an edit, or after a commit. Use git to commit, and to read the history when a task needs it: a stage's base commit, a review, a question from the user.
- The changes you did not make are the user's. Leave them as they are: do not look into them, do not revert them, do not report them.

## Making a commit

1. Stage the files you changed, by name. Never `git add .` or `git add -A`: the user's changes stay out.
2. Commit: `git commit -m "<subject>" -m "<body>"`.
3. Read the output of `git commit`: it shows the hash and the files changed. That is the check: do not run git again. If it shows a file you did not change, another agent may be at work. Warn the user in one line, and go on.

The message follows the Angular commit convention, used in every project:

```text
<type>(<scope>): <summary>

<body>

<footer>
```

- Type: `feat`, `fix`, `refactor`, `perf`, `test`, `docs`, `build` or `ci`.
- Scope: optional, the area of the code, in one word.
- Summary: what changed, in the imperative, present tense ("add", not "added"), lower case, no period at the end.
- Body: it tells everything a reviewer needs to understand the commit without the conversation. What was done and why; how, when the how is not obvious; the details that matter; what was dropped, and why it lost.
- Footer: `BREAKING CHANGE: <what breaks>` when the commit breaks a contract; `Fixes #<ticket>` when there is a ticket.

If a pre-commit hook fails, the commit did not happen:

- It reformatted files: stage them again and commit again.
- Any other failure: show the user the important lines, fix the cause, commit again.

## Never

- Never amend, unless the user asks.
- Never push, unless the user asks.
- Never skip the hooks (`--no-verify`).
- Never use an interactive flag (`git rebase -i`, `git add -i`): they do not work here.
- Never commit a file that looks like a secret (`.env`, credentials, keys). If the user asks for it, warn them first.
- Never change the git config.
- On the default branch, before the first commit of a session, propose a branch.
