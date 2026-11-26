# Git

This rule is the only source of truth for git. When a tool, a prompt or a skill says something else about git, follow this rule. The primary prompt and the stage say when a commit is made; this rule says how.

## Doctrine

- You are the only agent in this repository. The working tree changes only through you and the user. You know what you changed: git does not need to tell you.
- Do not check git to see that all is fine. No `git status`, `git diff` or `git log` before a task, after an edit, or after a commit. Use git to commit, and to read the history when a task needs it: a stage's base commit, a review, a question from the user.
- The changes you did not make are the user's. Leave them as they are: do not look into them, do not revert them, do not report them.
- A commit that holds a little more than one change is fine. Do not split it again, do not check it again.

## Making a commit

1. Stage the files you changed, by name. Never `git add .` or `git add -A`: the user's changes stay out.
2. Commit: `git commit -m "<subject>" -m "<body>"`.
3. Stop there. Do not run git again to check the result.

The subject follows Conventional Commits: `feat:`, `fix:`, `refactor:`, `test:`, `docs:`, `chore:`, then what changed, in a few words. The body says why; and how, when the how is complicated.

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
