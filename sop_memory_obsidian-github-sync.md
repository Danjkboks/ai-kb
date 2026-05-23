---
type: sop
domain: memory
source: session
status: active
tags: [obsidian, github, git, sync, memory, windows]
created: 2026-05-12
updated: 2026-05-21
---

# SOP: Obsidian + GitHub as Agent Memory (Windows/Sysadmin Edition)

**Goal**: Establish a permanent, version-controlled knowledge base that auto-syncs and acts as long-term memory for AI agents.

## 1. Prerequisites & Repository Clone
- **Command**: `git clone https://github.com/Danjkboks/workflow-lab.git`
- **Windows Credential Manager**: When prompted by Git Credential Helper, check "Always use this from now on", select **manager**, and log in via the browser popup. This stores the GitHub token locally.

## 2. Fix Git Author Identity (Fresh Installs)
If Git throws an `Author identity unknown` error, define your global identity in PowerShell:
```powershell
git config --global user.name "Danjkboks"
git config --global user.email "your-github-email@example.com"
```

## 3. Install Obsidian & Mount Vault
- **Install**: `winget install Obsidian.Obsidian`
- **Mount**: Open Obsidian -> Click **Open folder as vault** -> Select the cloned `workflow-lab` folder.

## 4. Install & Configure the Git Plugin
1. Open Obsidian Settings (Gear icon) -> **Community plugins** -> Turn off **Safe Mode**.
2. Click **Browse**, search for **"Git" by Vinzent (Denis Olehov)** (2.5M+ downloads).
3. Install and **Enable** the plugin.
4. Open the plugin's options and change exactly these two settings:
   - **Auto commit-and-sync interval (minutes)**: `10`
   - **Auto pull interval (minutes)**: `10`
*(Leave the commit message as default: `vault backup: {{date}}`)*

## 5. Initial Manual Push (Activation)
To prime the loop and verify credentials before relying on automation:
1. Create a new note (e.g., `Test Note.md`).
2. Press `Ctrl + P` to open the command palette.
3. Type and select: **Git: Commit-and-sync**.
4. Verify the top-right notification says "Committed" and "Pushed X files to remote". 

**Result**: The automation is live. Obsidian will silently pull AI-written SOPs from GitHub and push your local notes to GitHub every 10 minutes.
