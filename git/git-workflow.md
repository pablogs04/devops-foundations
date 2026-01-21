# Git workflow (Sprint 1)

## Setup
- git config (name/email)
- SSH auth to GitHub

## Day-to-day
- clone / init
- status / add / commit
- branch naming
- push and upstream
- PR: review + merge

## Undo / fix
- revert (must know)
- reset (basic awareness)

## Conflicts
- detect
- resolve
## Staging area (index)
- Working directory: tus cambios locales
- Staging area: lo que va al próximo commit
- Commit: snapshot en el historial

## Revert (recommended undo in shared repos)
- Use when you already pushed a bad commit and want to undo it without rewriting history.
- Example:
  - Find commit: `git log --oneline -n 5`
  - Revert it: `git revert <commit_sha>`
  - Push: `git push`
