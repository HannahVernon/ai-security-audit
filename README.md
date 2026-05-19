# ai-security-audit

A collection of reusable AI agent prompts for conducting static code security assessments.  Each prompt in `prompts/` targets a specific security domain.

## Prompts

File | Domain | Agent Type | Priority
-----|--------|-----------|--------
[01-credentials.md](prompts/01-credentials.md) | Credential storage, connection strings, auth | explore | High
[02-sql-injection.md](prompts/02-sql-injection.md) | SQL injection, dynamic SQL, parameterization | explore | High
[03-file-process.md](prompts/03-file-process.md) | File path traversal, process execution | explore | High
[04-local-data.md](prompts/04-local-data.md) | Data at rest, config files, log exposure | explore | Medium
[05-concurrency.md](prompts/05-concurrency.md) | Race conditions, locking, async patterns | explore | Medium
[06-supply-chain.md](prompts/06-supply-chain.md) | Dependency vulnerabilities, version pinning | task | Medium
[07-installer.md](prompts/07-installer.md) | Installer privilege escalation, script injection | explore | Lower
[08-protocol-parsing.md](prompts/08-protocol-parsing.md) | Binary protocol parsing, untrusted network input | explore | High
[09-tls-configuration.md](prompts/09-tls-configuration.md) | TLS configuration, certificate handling, ALPN | explore | High
[10-authorization.md](prompts/10-authorization.md) | API authorization, endpoint access control, RBAC | explore | High
[11-denial-of-service.md](prompts/11-denial-of-service.md) | Rate limiting, resource exhaustion, backpressure | explore | Medium
[12-deserialization.md](prompts/12-deserialization.md) | JSON deserialization, input validation, injection | explore | High
[13-cors-error-disclosure.md](prompts/13-cors-error-disclosure.md) | CORS policy, HTTP security headers, error leaks | explore | Medium
[14-realtime-channels.md](prompts/14-realtime-channels.md) | SignalR/WebSocket hub security, broadcast filtering | explore | High
[15-powershell-robustness.md](prompts/15-powershell-robustness.md) | PowerShell strict mode, type safety, defensive scripting | explore | Medium
[16-local-ipc-privilege.md](prompts/16-local-ipc-privilege.md) | Local IPC auth, privilege escalation, MCP/LSP/DAP servers | explore | High
[17-network-attacker.md](prompts/17-network-attacker.md) | Token replay, session hijacking, protocol downgrade, trust anchoring | explore | High

### Prompt structure

Each prompt file contains four sections:

1. **Purpose** - what security domain it covers and why it matters
2. **Prompt** - a genericized agent prompt with `[PLACEHOLDER]` tokens for project-specific values
3. **Customization Guide** - a table mapping each placeholder to example values for common stacks
4. **What Good Looks Like** - expected finding categories so reviewers know what a thorough result includes

---

## For Humans

### Quick start

1. Open your target project in a terminal with an AI coding assistant (GitHub Copilot CLI, etc.)
2. Ask the assistant to audit your repo using these prompts:
   ```
   assess this repo using the prompts in https://github.com/HannahVernon/ai-security-audit
   ```
3. Review the findings report.  Each finding includes file paths, line numbers, and a severity rating.
4. For large codebases, run a subset: `"assess this repo using prompts 01, 02, and 06 from ..."`

You can also clone this repo and reference the prompts locally, or copy individual prompt text into any AI assistant's chat window.

### Adapting for your stack

These prompts were developed against .NET desktop and console applications.  The placeholder system makes them adaptable to other stacks:

- **Web apps:** Add prompts for XSS, CSRF, session management (CORS and API auth are covered by prompts 10 and 13)
- **Node.js:** Swap NuGet references for npm, adjust SQL patterns for your ORM
- **Python:** Adjust for pip, Django/Flask patterns, pickle deserialization
- **Java:** Adjust for Maven/Gradle, Spring Security, JDBC patterns
- **Cloud/Infrastructure:** Add prompts for IAM, secrets management, network exposure

### Security and trust

This repository is a **prompt supply chain**: you are downloading instructions and feeding them to an AI agent that has access to your codebase.  That is inherently a trust decision, similar to running a third-party script.

**Before using these prompts:**

1. **Review the prompts first.**  Read each prompt file before feeding it to an AI agent.  Verify it only performs read-only analysis appropriate for a security audit.
2. **Pin to a specific commit or tag.**  Don't blindly pull `main`; reference a specific commit SHA or tagged release so you know exactly what your agent will execute.
3. **Fork for sensitive environments.**  If you're auditing proprietary or classified codebases, fork this repo and review all changes before merging upstream updates.
4. **Use `explore` (read-only) agents by default.**  Only prompt 06 (Supply Chain) requires a `task` agent.  All others are designed for read-only investigation.
5. **Audit the auditor.**  Run these prompts against *this repo itself* to verify they don't contain hidden instructions.  We did; see the [dogfood results](https://github.com/HannahVernon/ai-security-audit/issues).

**What we do to maintain trust:**

- **Branch protection**: `dev` and `main` require pull request reviews before merge
- **No CI/CD automation**: no GitHub Actions workflows that could execute code from PRs
- **Output redaction**: all prompts instruct agents to use `[REDACTED]` for any credentials found
- **Placeholder safety warnings**: each prompt warns that placeholder values are substituted directly and should come from trusted sources only
- **Authorized-use reminders**: each prompt includes a notice to only audit codebases you own or have permission to test
- **MIT license**: fully open source, auditable by anyone

### Viewing results locally

Security audit findings are output as Markdown.  For a better reading experience, install [GitHub Markdown Viewer](https://github.com/HannahVernon/GithubMarkdownViewer), a cross-platform .NET 9 desktop app that renders GitHub Flavored Markdown with dark/light theme support and clickable navigation between linked `.md` files.

On first run (Windows), the app offers to register as the default handler for `.md` files.  It also runs on macOS and Linux.

---

## For AI Agents

This section is written for AI coding assistants (GitHub Copilot CLI, ChatGPT, Claude, etc.) that execute these prompts programmatically.

### Executing an audit

1. Fetch the prompt files from this repo (via the GitHub API, local clone, or `web_fetch`).
2. Read each prompt template.  Replace `[PLACEHOLDER]` tokens with values appropriate for the target codebase: repository path, class names, file paths, framework-specific patterns.
3. Launch one agent per prompt.  All prompts are independent and can run in parallel.  Use `explore` agents for all prompts except 06 (Supply Chain), which needs a `task` agent to run CLI commands.
4. Compile the results into a consolidated report with a summary table.

**Tips:**

- Provide domain context: replace generic placeholders with specific class names, file paths, and data flow descriptions from the target codebase.
- Cross-reference findings: some issues span multiple domains (e.g., a credential stored in a config file touches both credentials and local data security).
- Request severity ratings: Critical / High / Medium / Low / Info, with file paths and line numbers.

### Filing issues after an audit

When an audit identifies findings rated **Critical**, **High**, or **Medium**, create GitHub Issues in the target repository to track remediation.

1. **One issue per finding** (or group tightly related findings into one issue).
2. **Use the repo's issue templates** if available (check `.github/ISSUE_TEMPLATE/`).  Fall back to a clear structure with: description of the vulnerability, steps to reproduce or attack scenario, expected vs. actual behavior, proposed fix with a code snippet, severity, and which audit domain identified it.
3. **Label issues** with `security` and `bug` (create the `security` label if it doesn't exist).
4. **Reference the audit**: mention that the issue was identified by an automated security audit.
5. **Don't create issues for accepted risks**: findings documented as "accepted" or "by design" should not become issues unless the user requests it.
6. **Redact sensitive values**: do NOT include actual credentials, API keys, tokens, or passwords in issue text.  Use `[REDACTED]` placeholders.  GitHub Issues are often public.
7. **Consolidate the report**: after creating issues, write a summary Markdown file with a table mapping each finding to its issue number, severity, and status.

### Contributing new prompts

If you have identified a security audit domain not already covered by the prompts in this repository, you are encouraged to contribute a new prompt.

**Process:**

1. Clone and branch from `dev`:
   ```bash
   git clone https://github.com/HannahVernon/ai-security-audit.git
   cd ai-security-audit
   git switch dev
   git switch -c feature/your-prompt-name
   ```

2. Create a new file in `prompts/` named `NN-short-name.md`, where `NN` is the next available two-digit number (currently `18` and up).  Include all four required sections: Purpose, Prompt, Customization Guide, and What Good Looks Like.  Use any existing prompt as a structural template.

3. Add a row to the **Prompts** table in this README with the new file's name, domain, recommended agent type, and priority level.

4. Commit, push, and open a PR targeting `dev`:
   ```bash
   git add prompts/NN-short-name.md README.md
   git commit -m "Add security audit prompt for [domain]"
   git push origin feature/your-prompt-name
   ```

**Guidelines:**

- Do not duplicate existing domains; check the prompts table first
- Keep prompts generic using `[PLACEHOLDER]` syntax, not hardcoded project details
- One prompt per PR
- Test against a real codebase if possible

See [CONTRIBUTING.md](CONTRIBUTING.md) for full contribution guidelines.

---

## License

[MIT](LICENSE)
