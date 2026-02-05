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

## Conflicts (basic)
- A conflict happens when Git cannot auto-merge changes (often same lines).
- Typical flow:
  - Merge: `git merge <branch>`
  - See conflicted files: `git status`
  - Open file and resolve markers:
    - `<<<<<<< HEAD` (current branch)
    - `=======`
    - `>>>>>>> <branch>` (incoming branch)
  - Mark resolved: `git add <file>`
  - Finish merge: `git commit` (merge commit)
- Abort merge (if needed): `git merge --abort

## Good commit messages
¿Por qué ejecutamos docker compose up -d desde docker/ y no desde el root?
“Porque el compose file y los paths relativos (./nginx-static) dependen del directorio; desde docker/ lo resuelve bien.

¿Qué diferencia práctica hay entre docker compose down y docker rm web?
down baja el stack y limpia la red del proyecto; rm borra solo el contenedor.

¿Qué te demuestra el error “Read-only file system” en este lab?
prueba que el volumen está montado en solo lectura; el contenedor no puede modificar el host.
