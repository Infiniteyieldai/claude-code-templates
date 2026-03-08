# CLAUDE.md — claude-code-templates

<!-- PROMPT CACHE BOUNDARY: All content in this file is stable and cache-eligible. -->
<!-- Keep this file consistent across sessions. Add session-specific context at the BOTTOM only. -->

This file provides guidance to Claude Code when working with this repository.

## Project Overview

Node.js CLI tool for managing Claude Code components (agents, commands, MCPs, hooks, settings) with a static website for browsing and installing components. Includes Vercel API endpoints for download tracking and Discord integration.

Key packages: `claude-code-templates` (npm), website at `aitmpl.com`, hosted on Vercel.

## Essential Commands

```bash
# Development
npm install                    # Install dependencies
npm run dev                    # Start dev server
npm test                       # Run tests
npm run lint                   # Run linter

# Build & Deploy
npm run build                  # Build project
vercel --prod                  # Deploy to production

# CLI usage
npx claude-code-templates@latest                    # Interactive mode
npx claude-code-templates@latest --agent <name>     # Install agent
npx claude-code-templates@latest --command <name>   # Install command
npx claude-code-templates@latest --hook <name>      # Install hook
npx claude-code-templates@latest --mcp <name>       # Install MCP
```

## Security Guidelines

### NEVER Hardcode Secrets

**NEVER write API keys, tokens, or passwords in code.**

```javascript
// WRONG
const API_KEY = "AIzaSy...";

// CORRECT
const API_KEY = process.env.GOOGLE_API_KEY;
```

Required environment variables (add to `.env`, never commit):
- `ANTHROPIC_API_KEY`
- `DISCORD_APP_ID`, `DISCORD_BOT_TOKEN`, `DISCORD_PUBLIC_KEY`, `DISCORD_WEBHOOK_URL_CHANGELOG`
- `NEON_DATABASE_URL` (postgresql://user:pass@host/db?sslmode=require)
- `SUPABASE_URL`, `SUPABASE_SERVICE_ROLE_KEY`
- `GITHUB_TOKEN` (public_repo scope)
- `TELEGRAM_BOT_TOKEN`, `TELEGRAM_CHAT_ID`
- `TRIGGER_SECRET` (Cloudflare Worker auth)

## Component Structure

All components live in `cli-tool/components/`:

```
cli-tool/components/
  agents/          # Claude Code sub-agents (.md files)
  commands/        # Custom slash commands (.md files)
  hooks/           # Pre/post tool hooks (.json files)
  settings/        # Claude Code settings (.json files)
  mcps/            # MCP server configs (.json files)
```

Every component requires a valid JSON/Markdown file with:
- `name` — kebab-case identifier
- `description` — clear, specific, trigger-keyword-rich (under 1024 chars)
- `category` — component type

## MCP Configuration

MCP servers configured in `.mcp.json`. Token efficiency rules:
- Use **paginated or filtered calls** — never "list everything" from an MCP tool
- Scope MCP responses to only the fields needed for the current task
- Prefer diff or summary responses over full file content
- Disable unused MCP servers to avoid unnecessary context injection per session

## Installation Patterns

```bash
# Single component
npx claude-code-templates@latest --agent frontend-developer
npx claude-code-templates@latest --command setup-testing
npx claude-code-templates@latest --hook automation/simple-notifications

# Batch installation
npx claude-code-templates@latest --agent security-auditor --command security-audit --setting read-only-mode

# Interactive mode
npx claude-code-templates@latest
```

## Component Review

Use the `component-reviewer` agent before committing component changes. It provides:
- **Critical Issues** — must fix before merge (security, missing fields)
- **Warnings** — should fix (clarity, best practices)
- **Suggestions** — nice to have improvements

```bash
# Before committing agent changes
Use the component-reviewer agent to review cli-tool/components/agents/<name>.md

# Before committing hook changes
Use the component-reviewer agent to review cli-tool/components/hooks/<name>.json
```

## Code Standards

### Path Handling
- Use relative paths: `.claude/scripts/`, `.claude/hooks/`
- Never hardcode absolute paths or home directories
- Use `path.join()` for cross-platform compatibility

### Hooks
- Hooks MUST NOT dump full file content — use diffs or summaries only
- Scope hooks to relevant file types using glob patterns
- Avoid hooks that fire on every tool call — use targeted event triggers

## Publishing (npm)

```bash
# 1. Update version
npm version patch|minor|major

# 2. Authenticate (use granular access token from npmjs.com/settings/~/tokens)
npm config set //registry.npmjs.org/:_authToken=YOUR_GRANULAR_TOKEN
npm publish
npm config delete //registry.npmjs.org/:_authToken   # always clean up

# 3. Tag the release
git tag vX.Y.Z && git push origin vX.Y.Z

# 4. Deploy website
vercel --prod
```

**npm Publishing Notes:**
- Use granular access tokens (classic tokens revoked Dec 2025)
- Token requires Read and Write permissions for `claude-code-templates` and Bypass 2FA enabled

## Deployment & Monitoring

### Vercel
- Production: `vercel --prod`
- Emergency rollback: `vercel ls` then `vercel promote <previous-deployment>`

### Cloudflare Workers (pulse-weekly-report)
```bash
# Trigger report
curl -X POST "https://pulse-weekly-report.SUBDOMAIN.workers.dev/trigger?source=github" \
  -H "Authorization: Bearer $TRIGGER_SECRET"

# Dry run (no Telegram)
curl -X POST "https://pulse-weekly-report.SUBDOMAIN.workers.dev/trigger?send=false" \
  -H "Authorization: Bearer $TRIGGER_SECRET"
```

## Blog Article Generation

```bash
/create-blog-article @cli-tool/components/{type}/{category}/{name}.json
```

This automatically: generates AI cover image, creates HTML with SEO optimisation, updates `docs/blog/blog-articles.json`.

<!-- DYNAMIC SECTION: Add session-specific context below. Not cache-eligible. -->

## Current Task Context

_Replace this section each session with task-specific context. Keep all sections above stable for prompt caching._
