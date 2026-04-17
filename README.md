# ai-security-audit

A collection of reusable AI agent prompts for conducting static code security assessments. Designed for use with GitHub Copilot CLI's sub-agent system, but adaptable to any AI coding assistant that can search and read codebases.

## How It Works

Each prompt in `prompts/` targets a specific security domain. Launch them **in parallel** against your codebase for a comprehensive assessment in minutes rather than days.

### Quick Start

1. Clone this repo
2. Open your target project in a terminal with GitHub Copilot CLI
3. Reference these prompts when asking Copilot to conduct a security review
4. Customize the `[PLACEHOLDERS]` in each prompt for your specific codebase

### Prompt Structure

Each prompt file contains:
- **Purpose** — what security domain it covers
- **Prompt** — the genericized agent prompt with placeholders
- **Customization Guide** — how to adapt it for your stack
- **What Good Looks Like** — expected findings categories

## Prompts

| File | Domain | Agent Type | Priority |
|------|--------|-----------|----------|
| [01-credentials.md](prompts/01-credentials.md) | Credential storage, connection strings, auth | explore | High |
| [02-sql-injection.md](prompts/02-sql-injection.md) | SQL injection, dynamic SQL, parameterization | explore | High |
| [03-file-process.md](prompts/03-file-process.md) | File path traversal, process execution | explore | High |
| [04-local-data.md](prompts/04-local-data.md) | Data at rest, config files, log exposure | explore | Medium |
| [05-concurrency.md](prompts/05-concurrency.md) | Race conditions, locking, async patterns | explore | Medium |
| [06-supply-chain.md](prompts/06-supply-chain.md) | Dependency vulnerabilities, version pinning | task | Medium |
| [07-installer.md](prompts/07-installer.md) | Installer privilege escalation, script injection | explore | Lower |
| [08-protocol-parsing.md](prompts/08-protocol-parsing.md) | Binary protocol parsing, untrusted network input | explore | High |
| [09-tls-configuration.md](prompts/09-tls-configuration.md) | TLS configuration, certificate handling, ALPN | explore | High |

## Usage Tips

- **Launch all agents in parallel** — they're independent and don't share state
- **Provide domain context** — the more you know about the codebase architecture, the more targeted the prompts become. Replace generic placeholders with specific class names, file paths, and data flow descriptions.
- **Use `explore` agents** for read-only investigation, **`task` agents** when CLI commands are needed (e.g., `dotnet list package --vulnerable`, `npm audit`)
- **Request severity ratings** — ask for Critical/High/Medium/Low/Info with file paths and line numbers
- **Cross-reference findings** — some issues span multiple domains (e.g., a credential stored in a config file touches both credentials and local data security)

## After the Audit: Filing Issues

When an audit identifies findings rated **Critical**, **High**, or **Medium**, the AI agent should **create GitHub Issues** in the target repository to track remediation. This ensures findings aren't lost in chat history.

### Guidelines for AI agents

1. **Create one issue per finding** (or group tightly related findings into one issue)
2. **Use the repo's issue templates** if available (check `.github/ISSUE_TEMPLATE/`). Fall back to a clear structure with these sections:
   - **Description** of the vulnerability or weakness
   - **Steps to reproduce** or attack scenario
   - **Expected vs. actual behavior**
   - **Proposed fix** with a code snippet where possible
   - **Severity** and which audit domain identified it
3. **Label issues** with `security` and `bug` (create the `security` label if it doesn't exist)
4. **Reference the audit** — mention that the issue was identified by an automated security audit
5. **Don't create issues for accepted risks** — findings documented as "accepted" or "by design" in the repo's security review should not become issues unless the user requests it
6. **Consolidate the report** — after creating issues, write a summary markdown file with a table mapping each finding to its issue number, severity, and status

## Example: Running a Full Audit from Another Repo

You don't need to clone this repo or copy prompts. Just point your AI assistant at this GitHub repo and ask it to fetch the prompts. Here's a real example using GitHub Copilot CLI against a .NET console app:

**User prompt:**
```
assess this repo using the prompts in https://github.com/HannahVernon/ai-security-audit
```

**What Copilot CLI does:**

1. Fetches the prompt files from this repo via the GitHub API
2. Reads each prompt template and customizes the `[PLACEHOLDERS]` for your codebase
3. Launches all audit agents in parallel (one per prompt)
4. Compiles findings into a consolidated report

**Behind the scenes**, each prompt is customized and dispatched as a background agent. For example, the credentials prompt becomes:

```
You are conducting a security assessment of the my-app repository at C:\Dev\my-app.

Your task: Audit credential and connection string handling across the entire codebase.

Specifically investigate:
1. Credential storage: Check MyDbContext, appsettings.json, UserCredential entity...
2. Connection string construction: Search for SqlConnectionStringBuilder...
3. Credential leakage: Check ILogger, Serilog output, exception messages...
...
```

All 9 agents run simultaneously and complete in 1–3 minutes. Results include file paths, line numbers, code snippets, and severity ratings. You'll typically see a summary table like:

| # | Domain | Result | Key Findings |
|---|--------|--------|--------------|
| 01 | Credentials | ✅ PASS | No secrets stored; auth tokens use secure storage |
| 02 | SQL Injection | ✅ PASS | All queries parameterized; LDAP filters escaped |
| 03 | File/Process | ⚠️ MEDIUM | Output files created with default permissions |
| 04 | Local Data | ⚠️ MEDIUM | X509Certificate2 objects not disposed |
| 05 | Concurrency | 🔴 HIGH | SslStream resource leak in error paths |
| 06 | Supply Chain | ✅ PASS | Zero CVEs; all packages pinned |
| 07 | Build/Deploy | ⚠️ MEDIUM | No release checksums or code signing |
| 08 | Protocol Parsing | ⚠️ MEDIUM | Length fields not bounds-checked |
| 09 | TLS Config | ✅ PASS | Intentional cert bypass documented |

**Tips for best results:**

- Run from the root of the repo you want to audit
- The AI will automatically detect your language/framework and adapt the placeholders
- For large codebases, you can run a subset: `"assess this repo using prompts 01, 02, and 06 from ..."`
- After the audit, ask follow-up questions: `"fix the HIGH findings from the concurrency audit"`

## Adapting for Your Stack

These prompts were battle-tested on .NET console and desktop applications. To adapt:

- **Web apps:** Add prompts for XSS, CSRF, CORS, session management, API authentication
- **Node.js:** Swap NuGet references for npm, adjust SQL patterns for your ORM
- **Python:** Adjust for pip, Django/Flask patterns, pickle deserialization
- **Java:** Adjust for Maven/Gradle, Spring Security, JDBC patterns
- **Cloud/Infrastructure:** Add prompts for IAM, secrets management, network exposure

## License

[MIT](LICENSE)
