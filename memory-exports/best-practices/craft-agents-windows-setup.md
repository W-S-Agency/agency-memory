# Craft Agents - Windows Setup & Troubleshooting

## Overview

Craft Agents is an Electron app that uses `@anthropic-ai/claude-agent-sdk` to run Claude Code as a Bun subprocess. On Windows, there are several environment issues that prevent proper startup.

## Setup Checklist

1. Git for Windows installed (`C:\Program Files\Git\bin\bash.exe`)
2. `CLAUDE_CODE_GIT_BASH_PATH` env var set (or auto-detected by our `options.ts` patch)
3. Claude Max/Pro OAuth token configured
4. `cli.js` patched (see below)

## Known Issue: bash.exe Detection Fails

**Symptom:** `Claude Code was unable to find CLAUDE_CODE_GIT_BASH_PATH`

**Cause:** SDK uses `execSync('dir "path"')` which fails in Electron/Bun subprocess on Windows (SHELL env var conflict with MSYS).

**Fix:** Patch the minified bash-check function in `cli.js`:
```
BEFORE: function XXXX(A){try{return YY(`dir "${A}"`,{stdio:"pipe"}),!0}catch{return!1}}
AFTER:  function XXXX(A){try{return require("fs").existsSync(A)}catch{return!1}}
```

Function name (XXXX) changes per SDK version. Find it with:
```bash
grep -oP '.{0,100}bash\.exe.{0,100}' node_modules/@anthropic-ai/claude-agent-sdk/cli.js
```

**Must reapply after every SDK update.**

Full documentation: `C:\Users\alexa\craft-agents-oss\WINDOWS-BASH-FIX.md`

## Known Issue: ~/.claude.json Corruption

**Symptom:** `CLI output was not valid JSON`

**Cause:** UTF-8 BOM, empty file, or invalid JSON in `~/.claude.json`

**Fix:** Already handled in `options.ts` → `ensureClaudeConfig()` function. Runs once per process, auto-repairs.

## Known Issue: Bun .env Loading

**Symptom:** API calls charged to personal key instead of Max subscription

**Cause:** Bun auto-loads `.env` from CWD, overriding OAuth auth

**Fix:** `--env-file=NUL` flag in `executableArgs` (already in `options.ts`)

## Environment Patches Applied

### `packages/shared/src/agent/options.ts`
- ComSpec set to `cmd.exe`
- MSYS-style SHELL removed
- Auto-detect `CLAUDE_CODE_GIT_BASH_PATH` from common Git install paths
- `--env-file=NUL` to block Bun's .env loading

### `packages/shared/src/network-interceptor.ts`
- SHELL cleanup (preload, runs before SDK)
- ComSpec fallback

## Date

2026-02-09
