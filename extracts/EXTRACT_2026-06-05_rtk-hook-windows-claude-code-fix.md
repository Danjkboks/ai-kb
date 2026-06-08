---
type: extract
date: 2026-06-05
session_id: rtk-hook-windows-claude-code-fix
surface: claude-code
environment: env1-claude-desktop
topics: [claude-code, memory, infra]
source_file: 2026-06-05-232851_0e2b4603-b359-4fec-bdd3-5a791aa19845.jsonl
processed_at: 2026-06-07T16:06:45.770Z
---

# Session Extract: rtk-hook-windows-claude-code-fix

## Decisions
- **Add RTK hook to Windows Claude Code settings.json to enable token compression for WSL2 commands**: RTK (Rust Token Killer) installed in WSL2 can compress tool call outputs, saving 68.2% tokens on commands like docker ps. The hook bridges Windows Claude Code to WSL2 RTK binary.

## Problems Solved
- **RTK hook not configured for Windows Claude Code, missing token savings on WSL2 tool calls**: Added PreToolUse Bash hook in C:\Users\GnReN-PC\claude\settings.json pointing to wsl -d Ubuntu -- /home/gnren-pc/local/bin/rtk hook claude
- **Git bash MSYS2 path conversion interfering with WSL paths in hook**: Use MSYS_NO_PATHCONV=1 environment variable to prevent MSYS2 from converting Windows paths to POSIX paths

## Errors Encountered
- [resolved] RTK binary not found at /home/gnren-pc/local/bin/rtk in WSL -> Verified correct path is /home/gnren-pc/local/bin/rtk and binary exists as ELF 64-bit executable
- [resolved] Hook blocking bash commands or causing hangs -> Tested hook with echo commands and verified it exits cleanly (exit code 0), not blocking

## Patterns Identified
- MSYS2 path conversion interferes with WSL paths in Claude Code hooks - use MSYS_NO_PATHCONV=1
- RTK hook must use exec form: wsl -d Ubuntu -- /home/gnren-pc/local/bin/rtk hook claude
- Hook testing requires checking both exit codes and actual compression stats

## Files Modified
- modified: C:\Users\GnReN-PC\claude\settings.json -- Added PreToolUse Bash hook for RTK token compression to WSL2 binary

## Next Session Must Know
- RTK hook active in Windows Claude Code settings.json at C:\Users\GnReN-PC\claude\settings.json
- Hook uses MSYS_NO_PATHCONV=1 to prevent Git bash path conversion issues
- RTK binary location: /home/gnren-pc/local/bin/rtk in WSL2 Ubuntu
- Current RTK savings: 68.2% on docker ps commands, 2 commands processed
- Hook format: wsl -d Ubuntu -- /home/gnren-pc/local/bin/rtk hook claude
- Test hook with: MSYS_NO_PATHCONV=1 wsl -d Ubuntu -- /home/gnren-pc/local/bin/rtk hook claude

## Skill Candidates
- claude-hook-tester: Test Claude Code hooks for proper execution, exit codes, and side effects

## Token Waste Flags
- Repeated testing of same hook configuration with minor variations
- Multiple bash calls to verify same binary path

## Knowledge Base Updates
- [update] ref_memory_token-optimization.md: Add RTK hook configuration for Windows Claude Code with WSL2 bridge details
- [create] guide_claude-code_hook-configuration.md: Document Claude Code hook patterns, MSYS2 path conversion issues, and testing procedures

## Links
related:: [[_INDEX]]
tags: claude-code, memory, infra
