# Installer & Deployment Security

**Domain:** Privilege escalation, script injection, deployment safety  
**Agent Type:** `explore`  
**Priority:** Lower

## Purpose

Audit installation, deployment, and build scripts for privilege escalation, unsigned code execution, and injection vulnerabilities. Installers often run with elevated privileges, making any vulnerability in the install path a potential system compromise.

> **⚠️ Authorized Use Only:** Use this prompt only to audit codebases you own or have explicit authorization to assess.

## Prompt

~~~
You are conducting a security assessment of the [APPLICATION_NAME] repository at [REPO_PATH].

Your task: **Audit installer and deployment security** including privilege escalation, script injection, and execution safety.

Specifically investigate:

1. **Dependency/package installation**: Check [DEPENDENCY_INSTALLER_FILES]:
   - Does it download external resources? Are downloads verified (checksum, signature)?
   - Does it run with elevated privileges? Is privilege escalation properly scoped?
   - Is there unsigned code execution?

2. **Script loading and ordering**: Check [SCRIPT_PROVIDER_FILES]:
   - How are installation/migration scripts loaded and ordered?
   - Could a malicious script file be injected into the execution sequence?
   - Is there path traversal risk in script discovery?
   - Are scripts validated (checksum, signature) before execution?

3. **Database setup scripts**: Check [INSTALL_SCRIPT_DIRECTORIES]:
   - Do they use dynamic SQL that could be injection vectors?
   - Do they create logins/users with hardcoded passwords?
   - What permissions do they grant? (least privilege?)
   - Do they enable dangerous features (xp_cmdshell, OPENROWSET, eval, etc.)?

4. **Upgrade/migration scripts**: Check [UPGRADE_SCRIPT_DIRECTORIES]:
   - Same checks as install scripts
   - Could an attacker inject a script that runs first? (ordering bypass)
   - Is there rollback capability if a script fails partway?

5. **User input handling**: Check [INSTALLER_ENTRY_POINTS]:
   - How are user-provided values (server names, paths, credentials) handled?
   - Are credentials visible in process lists (CLI arguments)?
   - Is input validated before use?

6. **Build scripts**: Check [BUILD_SCRIPT_FILES]:
   - Do they download unsigned dependencies?
   - Are there hardcoded paths or credentials?
   - Could environment variables be poisoned?

Search all relevant files. Provide detailed findings with severity ratings. **Do NOT include actual credential values, API keys, or tokens in your output** — use `[REDACTED]` placeholders.
~~~

## Customization Guide

> **⚠️ Placeholder Safety:** Placeholder values are substituted directly into the prompt text. A crafted value could act as a prompt injection. Only use placeholder values you trust — do not accept them from untrusted sources.

| Placeholder | Example Values |
|-------------|---------------|
| `[DEPENDENCY_INSTALLER_FILES]`| `Installer.Core/DependencyInstaller.cs`, `scripts/setup.sh`, `Dockerfile` |
| `[SCRIPT_PROVIDER_FILES]` | `Installer.Core/ScriptProvider.cs`, `db/migrate.ts`, `alembic/env.py` |
| `[INSTALL_SCRIPT_DIRECTORIES]` | `install/*.sql`, `db/init/`, `migrations/` |
| `[UPGRADE_SCRIPT_DIRECTORIES]` | `upgrades/`, `db/migrations/`, `flyway/sql/` |
| `[INSTALLER_ENTRY_POINTS]` | `Installer/Program.cs`, `scripts/install.sh`, `setup.py` |
| `[BUILD_SCRIPT_FILES]` | `build-all.cmd`, `Makefile`, `Dockerfile`, `.github/workflows/` |

## What Good Looks Like

- Downloads verified with checksums or signatures
- Privilege escalation scoped to minimum necessary operations
- Scripts loaded from trusted paths only (no user-influenced directories)
- No hardcoded passwords in setup scripts
- Least-privilege permissions granted (dedicated service accounts, minimal roles)
- Credentials prompted interactively, never passed as CLI arguments
- Build scripts use pinned tool versions from trusted sources
- Migration rollback capability on partial failure
