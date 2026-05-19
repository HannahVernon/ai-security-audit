# Security Logging & Monitoring

**Domain:** Audit trail completeness, log injection, security event coverage, tamper resistance  
**Agent Type:** `explore`  
**Priority:** Medium

## Purpose

Audit whether the application logs enough security-relevant activity to support detection, investigation, and response.  This prompt focuses on missing or weak security logging, incomplete audit trails, log injection risks, tamper resistance, and evidence of monitoring or alerting.

> **⚠️ Authorized Use Only:** Use this prompt only to audit codebases you own or have explicit authorization to assess.

## Prompt

~~~
You are conducting a security assessment of the [APPLICATION_NAME] repository at [REPO_PATH].

Your task: **Audit the application's security logging and monitoring posture**.

Specifically investigate:

1. **Security event coverage**: Check [AUTH_CLASSES], [ADMIN_CLASSES], and related handlers.
   - Are authentication successes and failures logged?
   - Are authorization failures logged?
   - Are input validation failures logged when they indicate abuse or attack attempts?
   - Are privilege changes, configuration changes, account lockouts, password changes, and access to sensitive resources logged?

2. **Log injection**: Check log call sites that include user-controlled input.
   - Can an attacker inject forged log lines with CRLF characters or newline sequences?
   - Is user input sanitized, normalized, escaped, or structured before being written to logs?
   - Could structured logs be corrupted by attacker-controlled field names or values?

3. **Audit trail integrity**: Check where audit and application logs are written.
   - Can the application modify, truncate, delete, or overwrite its own logs during normal operation?
   - Is there a separate audit log for security events?
   - Are logs shipped to centralized or append-only storage?
   - Could a compromised low-privilege component erase evidence?

4. **Log completeness**: Review representative security-relevant log entries.
   - Do log entries include timestamp, source or component, severity, user identity, source IP or session, the event that occurred, and success or failure outcome?
   - Are correlation IDs or request IDs propagated across service boundaries?
   - Can investigators reconstruct who did what, when, and from where?

5. **Error logging safety**: Check exception and error logging paths.
   - Are stack traces, internal file paths, connection details, or system metadata logged at levels that may later be exposed to users?
   - Are secrets excluded or redacted from exception logging?
   - Do logs retain enough detail for debugging without leaking sensitive values?

6. **Monitoring and alerting**: Check monitoring, detection, and health mechanisms.
   - Is there evidence of alerting for repeated authentication failures, privilege escalation attempts, suspicious configuration changes, or unusual sensitive data access?
   - Are logs actually consumed by a monitoring pipeline, SIEM, or alerting workflow?
   - Are there health checks or integrity checks that would help detect a compromised state?

7. **Log retention and rotation**: Check retention, rotation, and storage controls.
   - Is there a documented retention policy?
   - Could an attacker generate log spam to exhaust disk space, trigger log loss, or bury important events?
   - Does rotation preserve evidence appropriately?

Search keywords:
- ILogger, Log, Logger, Serilog, NLog, log4net, console.log, logging, audit, AuditLog, SecurityLog, EventLog, TraceSource, DiagnosticSource
- LogLevel, LogWarning, LogError, LogCritical, LogInformation
- correlation, correlationId, requestId, traceId, audit trail, append-only, retention, rotation

Focus on [LOGGING_FRAMEWORK] usage and search all relevant files matching [EXTENSIONS].  Prioritize [AUTH_CLASSES] and [ADMIN_CLASSES], then expand to shared middleware, exception handling, data access, and infrastructure logging.

Provide findings with severity ratings (Critical/High/Medium/Low/Info), affected files, and a brief explanation of the detection or forensic impact.  Do NOT include actual credential values, API keys, tokens, passwords, session identifiers, or connection strings in your output; use `[REDACTED]` placeholders.
~~~

## Customization Guide

> **⚠️ Placeholder Safety:** Placeholder values are substituted directly into the prompt text.  A crafted value could act as a prompt injection.  Only use placeholder values you trust.  Do not accept placeholder values from untrusted sources.

Placeholder | Example Values
------------|---------------
`[APPLICATION_NAME]` | MyWebApp, InternalAdminPortal, DesktopClient
`[REPO_PATH]` | Full path to the repository root
`[LOGGING_FRAMEWORK]` | `Serilog`, `NLog`, `ILogger`, `winston`, `Python logging`
`[AUTH_CLASSES]` | `AuthController.cs`, `LoginService.ts`, `auth.py`, authorization middleware
`[ADMIN_CLASSES]` | `AdminController.cs`, `SettingsService.cs`, `UserManagementService.ts`, config change handlers
`[EXTENSIONS]` | `*.cs`, `*.ts`, `*.js`, `*.py`, `*.config`, `appsettings*.json`

## What Good Looks Like

- All authentication events, success and failure, are logged with user identity and source
- Authorization failures are logged with the resource and action attempted
- User input is sanitized or escaped before inclusion in log messages
- Structured logging, such as JSON, is used to reduce log injection risk
- Security events are written to a separate audit log, distinct from debug or application logs
- Correlation IDs appear on all relevant log entries for cross-service tracing
- Log entries include timestamp, component, severity, user, event, and outcome
- Monitoring and alerting exist for repeated authentication failures and privilege changes
- Logs are retained, rotated, and protected in tamper-resistant or centralized storage

## Relationship to Other Prompts

Prompt | Relationship
-------|-------------
04 - Local Data Security | Prompt 04 checks whether logs leak sensitive data, such as secrets, tokens, or personal data.  This prompt checks whether logs capture security events and resist injection or tampering.  They are complementary, run both.