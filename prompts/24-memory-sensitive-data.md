# Memory & Sensitive Data Lifecycle

**Domain:** Sensitive data in memory, secure disposal, crash dump exposure, GC considerations, SecureString patterns  
**Agent Type:** `explore`  
**Priority:** Medium

## Purpose

Audit how the application handles sensitive data while it is live in memory.  This includes how secrets are created, retained, copied, logged, exposed in UI flows, and disposed of after use.

> **⚠️ Authorized Use Only:** Use this prompt only to audit codebases you own or have explicit authorization to assess.

## Prompt

~~~
You are conducting a security assessment of the [APPLICATION_NAME] repository at [REPO_PATH].

Your task: **Audit the in-memory lifecycle of sensitive data** across the application.

Specifically investigate:

1. **Sensitive data in strings**: In garbage-collected languages such as C#, Java, Python, and JavaScript, strings are immutable and cannot be securely wiped.  Check:
   - Are passwords, tokens, keys, or connection strings stored as plain strings?
   - Are they held in memory longer than necessary, such as in caches, static fields, or long-lived collections?
   - In .NET, is `SecureString` used, and is it actually more secure in this context or just security theater?
   - Are sensitive values passed as method parameters that may remain on the stack?
   - Review [CREDENTIAL_CLASSES].

2. **Secure disposal**: When sensitive data is no longer needed, check:
   - Are byte arrays containing keys or passwords zeroed out with `Array.Clear`, `CryptographicOperations.ZeroMemory`, `memset`, `explicit_bzero`, or equivalent APIs?
   - Are `IDisposable`, `AutoCloseable`, `using`, or `try-with-resources` patterns used for objects holding sensitive data?
   - Are cryptographic key objects disposed after use in [CRYPTO_CLASSES]?
   - In native or unmanaged code, are buffers both freed and zeroed?

3. **Crash dump exposure**: If the application crashes, check:
   - Could a memory dump, minidump, core dump, or heap dump contain plaintext secrets?
   - Is Windows Error Reporting, or an equivalent crash-dump facility, configured to limit dump contents?
   - Are crash reporting services such as Sentry, Application Insights, or Crashlytics configured to exclude sensitive data?
   - In .NET, do `DebuggerDisplay` attributes or similar debugging helpers expose secrets?

4. **Logging of in-memory values**: Check whether debug, trace, or structured logging writes sensitive values:
   - `ToString()` overrides on types holding sensitive data
   - Object serialization in logging, including request objects containing auth headers or tokens
   - `Debug.WriteLine`, `Console.WriteLine`, or equivalent diagnostics with sensitive values

5. **Clipboard and UI exposure**: For desktop or mobile apps, check [UI_CLASSES]:
   - Are passwords displayed in plaintext instead of masked fields?
   - Does copy and paste of sensitive fields put plaintext on the system clipboard?
   - Is clipboard content cleared after a timeout or when no longer needed?

6. **Memory-mapped files and shared memory**: Check:
   - Are memory-mapped files used for IPC with sensitive data?
   - Are shared memory segments properly access-controlled?
   - Is sensitive data in shared memory cleared when no longer needed?

Search patterns:
- Files: `**/*.[EXTENSIONS]`
- Keywords: `SecureString`, `ZeroMemory`, `RtlZeroMemory`, `SecureZeroMemory`, `Array.Clear`, `CryptographicOperations.ZeroMemory`, `Marshal.ZeroFreeGlobalAllocUnicode`, `Dispose`, `IDisposable`, `using`, `try-with-resources`, `AutoCloseable`, `close`, `finalize`, `memset`, `explicit_bzero`, `password`, `secret`, `key`, `token`, `credential`, `PasswordBox`, `UseSystemPasswordChar`, `PasswordChar`, `Clipboard`, `SetText`, `GetText`

Provide a detailed findings report with file paths, line numbers, and severity ratings (Critical/High/Medium/Low/Info).  Do NOT include actual credential values, API keys, tokens, passwords, or other secret material in your output.  Report them as `[REDACTED]`.
~~~

## Customization Guide

> **⚠️ Placeholder Safety:** Placeholder values are substituted directly into the prompt text.  A crafted value could act as a prompt injection.  Only use placeholder values you trust.  Do not accept placeholder values from untrusted sources.

Placeholder | Example Values
----------- | --------------
`[APPLICATION_NAME]` | `MyDesktopApp`, `AuthGateway`, `MobileClient`
`[REPO_PATH]` | Full path to the repository root
`[CREDENTIAL_CLASSES]` | `LoginViewModel`, `TokenProvider`, `CredentialCache`, `AuthRequest`
`[CRYPTO_CLASSES]` | `EncryptionService`, `KeyManager`, `JwtSigner`, `CertificateLoader`
`[UI_CLASSES]` | `LoginForm`, `PasswordDialog`, `SettingsView`, `AccountScreen`
`[EXTENSIONS]` | `cs,xaml,config`, `java,xml,properties`, `py,pyi,json`, `ts,tsx,js`

## What Good Looks Like

- Sensitive data is kept in byte arrays or similarly controllable buffers when feasible, not immutable strings.
- Sensitive buffers are zeroed immediately after use.
- Cryptographic key objects are disposed via `using`, `try-with-resources`, or equivalent lifetime controls.
- Sensitive values are not stored in static fields, global caches, or other long-lived containers.
- Crash dump configuration excludes, minimizes, or tightly controls sensitive data exposure.
- Password fields are masked in the UI, and clipboard handling is minimized or cleared after use.
- `ToString()` and debugger helpers on sensitive types do not reveal secret values.
- Debug and trace logging do not serialize objects containing secrets.
- Shared memory segments have restrictive ACLs and are cleared when released.

## Relationship to Other Prompts

Prompt 01 checks credential storage and transmission.  Prompt 04 checks sensitive data at rest on disk.  This prompt checks the in-memory lifecycle of sensitive data from creation to disposal.
