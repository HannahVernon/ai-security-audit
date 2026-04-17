# Local Data Security

**Domain:** Data at rest, configuration files, log exposure, sensitive data handling  
**Agent Type:** `explore`  
**Priority:** Medium

## Purpose

Audit how the application stores data locally — databases, config files, logs, caches. Even desktop/single-user applications can leak sensitive data if files are readable by other users or contain unencrypted secrets.

> **⚠️ Authorized Use Only:** Use this prompt only to audit codebases you own or have explicit authorization to assess.

## Prompt

~~~
You are conducting a security assessment of the [APPLICATION_NAME] repository at [REPO_PATH].

Your task: **Audit local data security** across the application.

Specifically investigate:

1. **Local database files**: Check [DATABASE_INIT_FILES]:
   - Where are database files created? What directory?
   - Are file permissions set on creation?
   - Is the database encrypted at rest?
   - Could another local user or admin read/modify the database?

2. **Archive/export files**: Check [ARCHIVE_SERVICE_FILES]:
   - Where are exported files stored?
   - File permissions on exports?
   - Could files from an untrusted source be loaded? (deserialization attacks)
   - Are old files securely deleted?

3. **Sensitive data in storage**: Check [SCHEMA_OR_MODEL_FILES]:
   - Are credentials stored in any local table or file?
   - Are query texts, user inputs, or API responses stored that could contain sensitive data?
   - Could metadata tables leak operational patterns?

4. **Settings/config persistence**: How does the application persist settings?
   - Are secrets (API keys, webhook URLs, SMTP passwords) stored in plaintext config files?
   - Is there a secure storage alternative (Credential Manager, Keychain, encrypted config)?
   - [SPECIFIC_CONFIG_LOCATIONS]

5. **Log files**: Check [LOGGING_FILES]:
   - What gets logged? Could sensitive data (queries, user data, tokens) appear in logs?
   - Log file location and permissions?
   - Log rotation — are old logs securely handled or just deleted?
   - Are log files accessible to other users?

Search all source files plus config/settings files. Provide findings with severity ratings (Critical/High/Medium/Low/Info). **Do NOT include actual credential values, API keys, or tokens in your output** — use `[REDACTED]` placeholders.
~~~

## Customization Guide

> **⚠️ Placeholder Safety:** Placeholder values are substituted directly into the prompt text. A crafted value could act as a prompt injection. Only use placeholder values you trust — do not accept them from untrusted sources.

| Placeholder | Example Values |
|-------------|---------------|
| `[DATABASE_INIT_FILES]`| `Database/DuckDbInitializer.cs`, `db/setup.py`, `prisma/schema.prisma` |
| `[ARCHIVE_SERVICE_FILES]` | `Services/ArchiveService.cs`, `services/export.ts`, `backup/` |
| `[SCHEMA_OR_MODEL_FILES]` | `Database/Schema.cs`, `models/`, `entities/` |
| `[SPECIFIC_CONFIG_LOCATIONS]` | `App.xaml.cs settings loading`, `config/default.json`, `.env` handling |
| `[LOGGING_FILES]` | `Services/AppLogger.cs`, `utils/logger.ts`, `logging.conf` |

## What Good Looks Like

- Credentials in OS-level secure storage, never in config files or databases
- Sensitive config values (webhook URLs, API keys) encrypted at rest
- Database files in user-private directories with restrictive ACLs
- Query text collection can be disabled for sensitive environments
- Log files don't contain credentials, tokens, or PII
- Log rotation with secure deletion (or documented limitation)
- Export files have restrictive permissions and are cleaned up
