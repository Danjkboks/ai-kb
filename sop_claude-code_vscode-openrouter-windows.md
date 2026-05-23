---
type: sop
domain: claude-code
source: session
status: active
tags: [claude-code, vscode, openrouter, windows, haiku]
created: 2026-05-12
updated: 2026-05-21
---

# SOP — Claude Code + VS Code + OpenRouter (Windows)

## Contexte

Setup validé sur Windows 11, PowerShell, avec OpenRouter comme backend API à la place d'Anthropic direct. Objectif : utiliser Claude Code au coût le plus bas possible avec ses propres crédits OpenRouter.

---

## Prérequis

- Compte OpenRouter actif avec crédits (`openrouter.ai`)
- Node.js 18+ installé (via `nvm-windows` recommandé)
- VS Code installé (`code.visualstudio.com`)

---

## Étape 1 — Installer Claude Code CLI

```powershell
npm install -g @anthropic-ai/claude-code
```

Vérifier :
```powershell
claude --version
```

---

## Étape 2 — Variables d'environnement permanentes

À exécuter dans PowerShell. **"User" est le mot-clé .NET — ne pas remplacer par son nom d'utilisateur.**

```powershell
[System.Environment]::SetEnvironmentVariable("ANTHROPIC_BASE_URL", "https://openrouter.ai/api/v1", "User")
[System.Environment]::SetEnvironmentVariable("ANTHROPIC_API_KEY", "sk-or-v1-VOTRE_CLE_ICI", "User")
[System.Environment]::SetEnvironmentVariable("ANTHROPIC_MODEL", "claude-haiku-3-5", "User")
```

Fermer PowerShell, rouvrir, vérifier :
```powershell
[System.Environment]::GetEnvironmentVariable("ANTHROPIC_BASE_URL", "User")
[System.Environment]::GetEnvironmentVariable("ANTHROPIC_API_KEY", "User")
[System.Environment]::GetEnvironmentVariable("ANTHROPIC_MODEL", "User")
```

Les trois lignes doivent retourner les valeurs.

---

## Étape 3 — Supprimer le fichier `.claude.json` par défaut

Le fichier `%USERPROFILE%\.claude.json` est créé automatiquement par Claude Code lors du premier lancement et contient un modèle hardcodé (ex: `claude-opus-4-7`) qui **override** les variables d'environnement.

```powershell
# Backup de sécurité (optionnel)
Copy-Item "$env:USERPROFILE\.claude.json" "$env:USERPROFILE\.claude.json.manual-backup"

# Suppression
Remove-Item "$env:USERPROFILE\.claude.json" -Force
```

Claude Code va recréer ce fichier automatiquement au prochain lancement sans le modèle hardcodé.

> **Note :** Si le problème revient après une mise à jour de Claude Code, répéter cette étape.

---

## Étape 4 — Tester le CLI

```powershell
cd D:\Claudi  # ou ton dossier de travail
claude --model claude-haiku-3-5 "say hello in one word"
```

Le flag `--model` force le modèle pour ce test. Si ça fonctionne, les variables d'env sont correctement lues.

---

## Étape 5 — Installer l'extension VS Code

1. Ouvrir VS Code
2. `Ctrl+Shift+X` → chercher **"Claude Code"** → installer (éditeur : Anthropic)
3. L'extension utilise automatiquement le CLI déjà configuré

---

## Étape 6 — Forcer le modèle dans VS Code

L'extension VS Code peut ignorer la variable `ANTHROPIC_MODEL` et revenir au modèle Anthropic par défaut.

Fix permanent : **File → Preferences → Settings** → icône `{}` (Open Settings JSON) en haut à droite → ajouter :

```json
{
  "claude.defaultModel": "claude-haiku-3-5"
}
```

Sauvegarder (`Ctrl+S`), fermer VS Code complètement, rouvrir.

Vérifier : le bas du panel Claude doit afficher **"Claude Haiku 4.5"**.

---

## Modèles disponibles via OpenRouter

| Modèle | Coût input | Coût output | Usage recommandé |
|--------|-----------|-------------|-----------------|
| `claude-haiku-3-5` | $0.80/Mtok | $4/Mtok | Défaut — 90% des tâches |
| `claude-sonnet-4-5` | $3/Mtok | $15/Mtok | Tâches complexes ponctuelles |
| `deepseek/deepseek-r1` | $0.55/Mtok | $2.19/Mtok | Raisonnement, alternative pas |

Pour basculer temporairement sur Sonnet :
```powershell
claude --model claude-sonnet-4-5 "ta question"
```

---

## Étape 7 — Créer le CLAUDE.md du workspace

Dans le dossier de travail, créer `.claude\CLAUDE.md` ou `CLAUDE.md` à la racine :

```markdown
# CLAUDE.md

## Comportement
- Répondre en français sauf si code
- Compact et direct, pas de filler
- Toujours proposer un plan avant de coder
- Petits commits atomiques

## Stack
- Windows 11, PowerShell
- OpenRouter API (budget limité, préférer Haiku)
- n8n self-hosted

## Ne JAMAIS faire
- Remove-Item sans confirmation explicite
- git push sans review
- Modifier des fichiers hors du workspace
```

Ce fichier est lu automatiquement par Claude Code à chaque session dans ce dossier.

---

## Résumé de l'architecture finale

```
VS Code
  └── Extension Claude Code
        └── Claude Code CLI
              └── ANTHROPIC_BASE_URL → OpenRouter API
                    └── Modèle : claude-haiku-3-5
```

---

## Problèmes fréquents

| Symptôme | Cause | Fix |
|----------|-------|-----|
| Modèle reste sur Opus | `.claude.json` hardcodé | Supprimer `%USERPROFILE%\.claude.json` |
| Variables env ignorées | Session PowerShell active | Fermer/rouvrir PowerShell |
| VS Code affiche Opus | Extension ignore env vars | Ajouter `claude.defaultModel` dans settings.json |
| `$env:VAR = "..."` ne persiste pas | Scope session uniquement | Utiliser `SetEnvironmentVariable(..., "User")` |

