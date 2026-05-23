---
type: extract
date: 2025-03-19
session_id: openrouter-auth-github-n8n-credential-test
surface: claude-code
environment: env2-workflow-lab
topics: [n8n, security, llm]
source_file: TESTRUN_credstrip_2026-05-24.jsonl
processed_at: 2026-05-23T23:16:48.084Z
---

# Session Extract: openrouter-auth-github-n8n-credential-test

## Decisions
- **Remove inline Authorization headers from n8n workflows and use credential system instead**: Security best practice to avoid hardcoded API keys in workflow JSON; credentials are encrypted and managed separately
- **Use GitHub credential type for OpenRouter authentication in n8n**: OpenRouter uses GitHub-style authentication (Bearer token in Authorization header); GitHub credential type provides proper token management

## Problems Solved
- **OpenRouter API calls failing in n8n workflows with 401 errors**: Created GitHub credential for OpenRouter with API key, then updated HTTP Request nodes to use credential instead of inline Authorization header

## Errors Encountered
- [resolved] 401 Unauthorized when calling OpenRouter API from n8n -> Switched from inline Authorization header to GitHub credential type with proper token format

## Patterns Identified
- OpenRouter uses GitHub-style authentication (Bearer token)
- n8n GitHub credential type works for any service using Bearer token authentication
- Inline Authorization headers in n8n workflows are security anti-pattern

## Files Modified
- modified: D:\aidirectory\n8n\workflows\openrouter-test.json -- Updated HTTP Request nodes to use GitHub credential instead of inline Authorization headers for OpenRouter API calls

## Next Session Must Know
- OpenRouter authentication in n8n uses GitHub credential type with Bearer token
- Never store API keys inline in n8n workflow JSON - use credential system
- OpenRouter API key format: 'Bearer sk-or-v1-...'
- n8n credentials are encrypted at rest and can be shared safely

## Skill Candidates
- n8n-credential-migration: Scan n8n workflows for inline Authorization headers and migrate them to proper credential types

## Token Waste Flags
none

## Knowledge Base Updates
- [update] sop_n8n_mcp-setup.md: Add section on OpenRouter authentication using GitHub credential type
- [create] ref_security_n8n-credential-management.md: Document security best practices for n8n credential management, including migration from inline headers

## Links
related:: [[_INDEX]]
tags: n8n, security, llm
