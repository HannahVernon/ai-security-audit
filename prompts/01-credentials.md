# Credential & Connection String Handling

**Domain:** Authentication, credential storage, connection security  
**Agent Type:** `explore`  
**Priority:** High

## Purpose

Audit how the application stores, transmits, and protects credentials. This is typically the highest-value target for attackers — a single leaked credential can compromise an entire environment.

> **⚠️ Authorized Use Only:** Use this prompt only to audit codebases you own or have explicit authorization to assess.

## Prompt

~~~
You are conducting a security assessment of the [APPLICATION_NAME] repository at [REPO_PATH].

Your task: **Audit credential and connection string handling** across the entire codebase.

Specifically investigate:

1. **Credential storage**: How are database/service credentials stored? Are passwords stored in plaintext? Check:
   - Configuration files (appsettings.json, .env, config.xml, etc.)
   - Source code (hardcoded strings, constants)
   - User settings or preferences files
   - Local databases or caches
   - [SPECIFIC_MODELS_OR_CLASSES]

2. **Connection string construction**: Search for all connection/client building. Are they using:
   - Builder classes (e.g., SqlConnectionStringBuilder, ConnectionStringBuilder) or manual concatenation?
   - Encryption/TLS settings (Encrypt, TrustServerCertificate, sslmode, etc.)?
   - Secure credential injection vs. inline passwords?

3. **Credential leakage**: Check if credentials appear in:
   - Log output ([LOGGING_CLASSES])
   - Exception messages displayed to users
   - Telemetry or structured logging
   - Database tables that store configuration
   - Serialized objects (JSON, XML, binary)

4. **Authentication methods**: How are different auth methods handled?
   - Which methods are supported (password, token, certificate, SSO, MFA)?
   - Is the most secure method the default?
   - Are less secure methods discouraged in the UI?

Search patterns:
- Files: `**/*.[EXTENSIONS]`
- Keywords: password, Password, credential, Credential, connectionstring, secret, apikey, api_key, token, Bearer, Authorization, Encrypt, TrustServerCertificate, SecureString, ProtectedData, DPAPI, keychain

Provide a detailed findings report with file paths, line numbers, and severity ratings (Critical/High/Medium/Low/Info). **Do NOT include actual credential values, API keys, tokens, or passwords in your output.** Report them as `[REDACTED]` — for example: "Line 42: Hardcoded password found in appsettings.json (value redacted)."
~~~

## Customization Guide

> **⚠️ Placeholder Safety:** Placeholder values are substituted directly into the prompt text. A crafted value could act as a prompt injection. Only use placeholder values you trust — do not accept them from untrusted sources.

| Placeholder | Example Values |
|-------------|---------------|
| `[APPLICATION_NAME]` | MyWebApp, PerformanceMonitor |
| `[REPO_PATH]` | Full path to the repository root |
| `[SPECIFIC_MODELS_OR_CLASSES]` | `ServerConnection model`, `UserCredential entity`, `DbContext` |
| `[LOGGING_CLASSES]` | `Logger, AppLogger, Serilog, NLog, ILogger` |
| `[EXTENSIONS]` | `cs,xaml,json,config` (.NET), `ts,js,json` (Node), `py,yaml,json` (Python) |

## What Good Looks Like

- Passwords in OS-level secure storage (Credential Manager, Keychain, DPAPI)
- Connection strings built with builder classes, never concatenated
- Encryption enabled by default on database connections
- `[JsonIgnore]` / `@JsonIgnore` on credential properties
- No credentials in log output, even at debug level
- Least-secure auth method is not the default
