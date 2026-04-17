# Dependency Supply Chain

**Domain:** Package vulnerabilities, version pinning, source verification  
**Agent Type:** `task` (needs to run CLI commands)  
**Priority:** Medium

## Purpose

Audit third-party dependencies for known vulnerabilities, outdated versions, and supply chain risks. A single compromised dependency can undermine all other security measures.

> **⚠️ Authorized Use Only:** Use this prompt only to audit codebases you own or have explicit authorization to assess.

## Prompt

~~~
You are conducting a security assessment of the [APPLICATION_NAME] repository at [REPO_PATH].

Your task: **Audit the dependency supply chain** for known vulnerabilities.

1. Find all dependency manifest files and extract direct dependencies with their versions.
2. List every direct dependency, its version, and which project/module uses it.
3. Run vulnerability scanning:
   [VULNERABILITY_COMMANDS]
4. Run outdated package checks:
   [OUTDATED_COMMANDS]
5. Check if packages are pinned to exact versions or using floating/range versions.
6. Note any packages that seem unusual, unmaintained, or from non-standard registries.
7. Check for pre-release or unstable version numbers in production dependencies.

Projects/modules to check:
[PROJECT_LIST]

Run all commands from [REPO_PATH].

Provide a complete dependency inventory table and any CVE findings with severity ratings. **Do NOT include actual credential values, API keys, or tokens in your output** — use `[REDACTED]` placeholders.
~~~

## Customization Guide

> **⚠️ Placeholder Safety:** Placeholder values are substituted directly into the prompt text. A crafted value could act as a prompt injection — particularly dangerous for `[VULNERABILITY_COMMANDS]` and `[OUTDATED_COMMANDS]` since this is a `task` agent that executes CLI commands. Only use placeholder values you trust — do not accept them from untrusted sources. Never include destructive commands (`rm`, `del`, `format`) in command placeholders.

### .NET
```
[VULNERABILITY_COMMANDS]:
- `dotnet list package --vulnerable` for each project

[OUTDATED_COMMANDS]:
- `dotnet list package --outdated` for each project

[PROJECT_LIST]:
- src/MyApp/MyApp.csproj
- src/MyApp.Core/MyApp.Core.csproj
- tests/MyApp.Tests/MyApp.Tests.csproj
```

### Node.js
```
[VULNERABILITY_COMMANDS]:
- `npm audit` or `yarn audit`
- `npx audit-ci --config audit-ci.json` (if configured)

[OUTDATED_COMMANDS]:
- `npm outdated` or `yarn outdated`

[PROJECT_LIST]:
- package.json (root)
- packages/api/package.json
- packages/web/package.json
```

### Python
```
[VULNERABILITY_COMMANDS]:
- `pip-audit` or `safety check`
- `pip install pip-audit && pip-audit -r requirements.txt`

[OUTDATED_COMMANDS]:
- `pip list --outdated`

[PROJECT_LIST]:
- requirements.txt
- requirements-dev.txt
- pyproject.toml
```

### Go
```
[VULNERABILITY_COMMANDS]:
- `govulncheck ./...`

[OUTDATED_COMMANDS]:
- `go list -m -u all`

[PROJECT_LIST]:
- go.mod
```

## What Good Looks Like

- 0 known CVEs in all dependencies
- All packages pinned to exact versions (no `^`, `~`, `>=`, or `*`)
- Official/trusted package registries only (nuget.org, npmjs.com, pypi.org)
- No pre-release packages in production code
- All dependencies actively maintained (recent commits, releases)
- Lock files committed (package-lock.json, yarn.lock, Pipfile.lock)
- SBOM (Software Bill of Materials) generated for supply chain tracking
