# CI/CD Pipeline Security

**Domain:** GitHub Actions, build pipeline secrets, workflow injection, artifact integrity, runner security  
**Agent Type:** `explore`  
**Priority:** Medium

## Purpose

Audit the CI/CD pipeline itself for workflow injection, secret exposure, action tampering, artifact integrity gaps, and unsafe runner design.  This prompt covers the CI/CD portion of OWASP A08, Software and Data Integrity Failures.  It complements dependency-focused supply chain review by examining whether the build and deployment pipeline can be abused to execute attacker-controlled logic, expose secrets, or ship untrusted artifacts.

> **⚠️ Authorized Use Only:** Use this prompt only to audit codebases, pipelines, and deployment configurations you own or have explicit authorization to assess.

## Prompt

~~~
You are conducting a security assessment of the [APPLICATION_NAME] repository at [REPO_PATH].

Your task: **Audit the [CI_PLATFORM] pipeline and related build and deployment workflow configuration** for CI/CD security weaknesses.  Focus on workflow injection, secret handling, third-party action trust, token permissions, artifact integrity, runner isolation, and deployment trigger safety.

Inspect these workflow and pipeline files first:
[WORKFLOW_FILES]

Specifically investigate:

1. **Workflow injection**: Check workflow definitions and inline scripts for attacker-controllable data flowing into command execution.
   - Look for expression injection via `${{ github.event.*.title }}`, `${{ github.event.*.body }}`, `${{ github.event.comment.body }}`, `${{ github.event.pull_request.title }}`, `${{ github.event.pull_request.body }}`, or similar event properties used inside `run:` steps
   - Flag `pull_request_target` workflows that check out PR code, run repository scripts from the PR, or otherwise execute untrusted code with elevated permissions
   - Check whether `workflow_dispatch` inputs are used unsafely in shell commands, PowerShell, Bash, or templated script generation
   - Check whether issue bodies, PR comments, commit messages, or other user-controlled event payload fields are passed into scripts without sanitization

2. **Secret exposure**: Review how secrets are injected, scoped, and logged.
   - Are secrets passed as environment variables to steps that do not need them?
   - Could secrets leak through logs, verbose errors, debug output, uploaded artifacts, cache contents, or generated configuration files?
   - Are secrets used inside composite actions or reusable workflows where they might be echoed, transformed, or logged indirectly?
   - Are there hardcoded tokens, keys, connection strings, or credentials in workflow files or action definitions?
   - Are secrets scoped appropriately, repository vs. organization vs. environment, and limited to the workflows that actually need them?

3. **Third-party action supply chain**: Review every external action reference.
   - Are third-party actions pinned to a full commit SHA rather than a mutable tag such as `@v3`?
   - Are any actions from unknown, low-trust, or minimally maintained publishers?
   - Do any actions request or inherit excessive permissions such as `contents: write`, `packages: write`, `pull-requests: write`, or broad OIDC access?
   - Is there evidence that action additions or updates are reviewed before adoption?

4. **Permissions and least privilege**: Review token and workflow permissions.
   - Is there a top-level `permissions:` block restricting default `GITHUB_TOKEN` scope?
   - Do individual jobs specify only the permissions they need?
   - Is the default token permission set to read-only at the repository or organization level?
   - Are there workflows with `permissions: write-all`, job-level over-permissioning, or no explicit permissions block and therefore broad inherited defaults?

5. **Artifact and build integrity**: Review how build outputs are produced and verified.
   - Are build artifacts signed, checksummed, or attested?
   - Could a compromised runner tamper with build outputs before upload or deployment?
   - Are deployment artifacts verified before deployment to [DEPLOYMENT_TARGETS]?
   - Is there a provenance chain from source commit to built artifact to deployed artifact?

6. **Self-hosted runner security**: Review runner trust boundaries.
   - Are self-hosted runners used?  If so, are they ephemeral or persistent?
   - Could a malicious workflow from a fork, pull request, or untrusted branch execute on a self-hosted runner?
   - Are self-hosted runners isolated, dedicated VM or container per job, rather than long-lived shared hosts?
   - Is the runner registration token rotated and protected?

7. **Branch protection and merge requirements**: Review deployment trigger safety.
   - Are protected branches enforced for deployment triggers?
   - Could a workflow be triggered by pushing directly to `main`, `master`, `release`, or other deployment branches without PR review?
   - Are required status checks configured so broken or unreviewed builds cannot be merged?

8. **Build-environment credential exposure**: Review whether dependency installation steps can access secrets or sensitive environment state.
   - Are credentials, cloud tokens (AWS, Azure, GCP), Docker registry credentials, or npm/PyPI publish tokens available as environment variables during dependency install steps (`npm install`, `pip install`, `dotnet restore`)?
   - Could a compromised dependency's install script (e.g., npm `postinstall`, Python `setup.py`) read and exfiltrate these credentials?
   - Are dependency install steps isolated from steps that require secrets (separate jobs, separate runners, or credential injection only after install)?
   - Does the build runner have network access to cloud metadata endpoints (169.254.169.254) that could expose instance credentials to install scripts?
   - Could a malicious dependency escape container boundaries on the build runner to access host-level secrets or other jobs?
   - See: "Shai-Hulud" npm supply chain attacks (https://safedep.io/mini-shai-hulud-strikes-again-314-npm-packages-compromised/) where compromised packages exfiltrated environment variables, cloud tokens, and attempted container escape

9. **AI coding agent configuration injection**: Review whether build or install processes could inject configuration files targeting AI coding agents.
   - Could a dependency's install script write files to `.claude/`, `.codex/`, `.cursor/`, `.github/copilot-instructions.md`, or similar AI agent configuration paths within the repository or developer home directory?
   - Are AI agent configuration files in the repository tracked by version control and reviewed in PRs, or could they be silently modified?
   - Does the CI pipeline run AI-assisted code generation or review tools that could be influenced by injected configuration?

Search patterns:
- Files: `.github/workflows/*.yml`, `.github/workflows/*.yaml`, `.github/actions/*/action.yml`, `Jenkinsfile`, `.gitlab-ci.yml`, `azure-pipelines.yml`, `bitbucket-pipelines.yml`, `.circleci/config.yml`
- Keywords: `${{`, `github.event`, `pull_request_target`, `workflow_dispatch`, `secrets.`, `GITHUB_TOKEN`, `permissions:`, `uses:`, `actions/`, `run:`, `env:`, `with:`, `self-hosted`, `artifact`, `upload-artifact`, `download-artifact`, `deploy`, `publish`, `postinstall`, `preinstall`, `setup.py`, `169.254.169.254`, `.claude`, `.codex`, `.cursor`, `copilot-instructions`

Provide a detailed findings report with file paths, line numbers, severity ratings, and concrete remediation guidance.  Explain the exploit path for each issue, including whether it enables workflow injection, secret disclosure, privilege escalation, artifact tampering, or unsafe deployment.  Do NOT include actual credential values, API keys, tokens, passwords, private keys, or other secret material in your output - use `[REDACTED]` placeholders.
~~~

## Customization Guide

> **⚠️ Placeholder Safety:** Placeholder values are substituted directly into the prompt text.  A crafted value could act as a prompt injection.  Only use placeholder values you trust.  Do not accept placeholder values from untrusted sources.

Placeholder | Example Values
------------|---------------
`[APPLICATION_NAME]` | `MyApp`, `CustomerPortal`, `BuildService`
`[REPO_PATH]` | Full path to the repository root
`[CI_PLATFORM]` | `GitHub Actions`, `GitLab CI`, `Azure DevOps`, `Jenkins`, `Bitbucket Pipelines`
`[WORKFLOW_FILES]` | `.github/workflows/build.yml`, `.github/workflows/release.yml`, `Jenkinsfile`, `.circleci/config.yml`
`[DEPLOYMENT_TARGETS]` | `npm`, `NuGet`, `Docker Hub`, `production servers`, `Kubernetes cluster`

## What Good Looks Like

- Top-level `permissions: read-all`, or explicit minimal permissions, on every workflow
- All third-party actions pinned to full commit SHA values, not mutable tags
- No expression injection, attacker-controllable values never interpolated into `run:` steps
- Secrets scoped to specific environments and workflows, not exposed broadly
- No `pull_request_target` workflows that check out and execute PR code
- Self-hosted runners are ephemeral, fresh VM or container per job, or not used
- Build artifacts are signed, checksummed, or attested with provenance from source commit to deployment
- Branch protection prevents direct pushes to deployment branches
- Workflow files are reviewed in pull requests like any other code change
- Dependency install steps run in isolation from secrets (credentials injected only in later steps)
- Build runners block access to cloud metadata endpoints during install phases
- AI agent configuration files (`.claude/`, `.codex/`, `.cursor/`, `.github/copilot-instructions.md`) are tracked in version control and reviewed in PRs

## Relationship to Other Prompts

Prompt | Relationship
-------|-------------
06 - Dependency Supply Chain | Prompt 06 covers dependency supply chain risk, vulnerable packages, version pinning, and source verification.  This prompt covers the build pipeline supply chain itself, workflow injection, action tampering, secret exposure, artifact integrity, and deployment workflow trust.
