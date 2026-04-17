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

## Usage Tips

- **Launch all agents in parallel** — they're independent and don't share state
- **Provide domain context** — the more you know about the codebase architecture, the more targeted the prompts become. Replace generic placeholders with specific class names, file paths, and data flow descriptions.
- **Use `explore` agents** for read-only investigation, **`task` agents** when CLI commands are needed (e.g., `dotnet list package --vulnerable`, `npm audit`)
- **Request severity ratings** — ask for Critical/High/Medium/Low/Info with file paths and line numbers
- **Cross-reference findings** — some issues span multiple domains (e.g., a credential stored in a config file touches both credentials and local data security)

## Adapting for Your Stack

These prompts were battle-tested on a .NET/WPF/DuckDB desktop application. To adapt:

- **Web apps:** Add prompts for XSS, CSRF, CORS, session management, API authentication
- **Node.js:** Swap NuGet references for npm, adjust SQL patterns for your ORM
- **Python:** Adjust for pip, Django/Flask patterns, pickle deserialization
- **Java:** Adjust for Maven/Gradle, Spring Security, JDBC patterns
- **Cloud/Infrastructure:** Add prompts for IAM, secrets management, network exposure

## License

[MIT](LICENSE)
