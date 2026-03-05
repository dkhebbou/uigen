---
allowed-tools: Bash(git add:*), Bash(git status:*), Bash(git commit:*), Bash(git push:*)
description: Commit et push les changements vers le dépôt distant
---

## Contexte

- Statut git actuel : !`git status`
- Diff complet (staged et unstaged) : !`git diff HEAD`
- Branche actuelle : !`git branch --show-current`
- Commits récents : !`git log --oneline -10`

## Tâche

En te basant sur les changements ci-dessus :

1. Stage tous les fichiers modifiés (`git add -A`)
2. Crée un commit avec un message clair et descriptif (en français, au présent)
3. Fais un push vers l'origin (`git push`)

Effectue toutes ces opérations en un seul message. Ne fais rien d'autre.
