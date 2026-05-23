---
type: extract
date: 2025-03-18
session_id: openrouter-auth-test-remove-inline-headers
surface: claude-code
environment: env2-workflow-lab
topics: [n8n, security]
source_file: TESTRUN_credstrip_2026-05-24.jsonl
processed_at: 2026-05-23T23:20:28.484Z
---

# Session Extract: openrouter-auth-test-remove-inline-headers

## Decisions
- **Removed inline Authorization headers from n8n workflows and replaced them with credential references**: To improve security by centralizing API key management in n8n's credential system and prevent accidental exposure in workflow exports

## Problems Solved
- **OpenRouter API key was hardcoded in HTTP Request node headers, creating a security risk**: Created an OpenRouter credential in n8n and updated all HTTP Request nodes to reference it via {{ $credentials.openRouterApiKey }}

## Errors Encountered
none

## Patterns Identified
- n8n credential references use {{ $credentials.credentialName }} syntax in node fields
- HTTP Request nodes must have 'Authentication' set to 'Generic Credential' to use credential references

## Files Modified
- modified: n8n/workflows/ -- Updated multiple workflow JSON files to replace inline Authorization headers with credential references

## Next Session Must Know
- OpenRouter credential is now stored in n8n credential system, not in workflow JSON
- All HTTP Request nodes to OpenRouter must use 'Authentication: Generic Credential' with reference to openRouterApiKey
- Workflow exports no longer contain API keys in plain text
- Credential references use {{ $credentials.openRouterApiKey }} syntax in header fields

## Skill Candidates
- n8n-credential-migration: Scans n8n workflow JSON for hardcoded API keys in headers and replaces them with credential references

## Token Waste Flags
none

## Knowledge Base Updates
- [update] sop_n8n_api-integration.md: Add section on credential management best practices and migrating from inline headers

## Links
related:: [[_INDEX]]
tags: n8n, security
