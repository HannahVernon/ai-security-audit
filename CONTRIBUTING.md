# Contributing to AI Security Audit

Thank you for your interest in contributing! This document explains how to get started, what we expect from contributions, and how we work together.

## Code of Conduct

This project adopts the [Contributor Covenant Code of Conduct v2.1](https://www.contributor-covenant.org/version/2/1/code_of_conduct/). By participating, you agree to uphold its standards.

**In short:** Be respectful, be constructive, be welcoming. Harassment, trolling, and personal attacks are not tolerated.

Instances of unacceptable behavior may be reported by:

- Opening a [GitHub Issue](https://github.com/HannahVernon/ai-security-audit/issues)
- Contacting the maintainer via [GitHub (@HannahVernon)](https://github.com/HannahVernon)

All reports will be reviewed promptly and handled with discretion.

## How to Contribute

### Improving Existing Prompts

The best contributions come from real-world usage. If you used a prompt against a codebase and found it missed something important or produced false positives:

1. Open an issue describing the gap
2. Submit a PR with the improvement, including what you tested it against

### Adding New Prompts

New security domains are welcome. Before writing a prompt, check that it doesn't overlap significantly with an existing one. Good candidates:

- **Web-specific:** XSS, CSRF, CORS, session management, API authentication
- **Cloud/Infrastructure:** IAM, secrets management, network exposure, container security
- **Mobile:** Keychain/Keystore usage, certificate pinning, intent injection
- **Language-specific:** Pickle deserialization (Python), prototype pollution (JS), unsafe reflection (Java)

### Prompt Quality Standards

Every prompt file should include:

1. **Purpose** — what security domain it covers and why it matters
2. **Prompt** — the genericized agent prompt with `[PLACEHOLDER]` syntax
3. **Customization Guide** — table mapping placeholders to example values per stack
4. **What Good Looks Like** — expected findings categories so users know what a clean result means

### Submitting Pull Requests

1. **Fork and branch** from `dev` (not `main`). Use descriptive branch names: `feature/xss-prompt`, `fix/sql-injection-false-positive`, etc.
2. **Keep changes focused.** One logical change per PR.
3. **Test against a real codebase.** Mention in the PR what you tested against and what findings the prompt produced.
4. **Placeholders must be clearly marked** with `[BRACKETS]` and documented in the customization guide.
5. **Write a clear commit message.** First line is a concise summary; body explains *why*, not just *what*.

### Branching

- `dev` — active development branch; PRs target here
- `main` — stable releases only
- Feature branches are deleted after merge

## License

By contributing, you agree that your contributions will be licensed under the [MIT License](LICENSE) that covers this project.
