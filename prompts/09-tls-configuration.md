# TLS Configuration & Certificate Handling

**Domain:** Transport Layer Security, certificate validation, cryptographic configuration  
**Agent Type:** `explore`  
**Priority:** High (for apps that negotiate or inspect TLS connections)

## Purpose

Audit how the application configures TLS connections, validates certificates, and handles cryptographic material. Misconfigurations such as disabled certificate validation, weak cipher suites, or improper certificate callback implementations can enable man-in-the-middle attacks or data interception — even when TLS is nominally "enabled."

> **⚠️ Authorized Use Only:** Use this prompt only to audit codebases you own or have explicit authorization to assess.

## Prompt

~~~
You are conducting a security assessment of the [APPLICATION_NAME] repository at [REPO_PATH].

Your task: **Audit TLS configuration and certificate handling** across the entire codebase.

This application [TLS_USAGE_DESCRIPTION].

Specifically investigate:

1. **Certificate validation callbacks**: Find all `RemoteCertificateValidationCallback`, `ServerCertificateCustomValidationCallback`, or equivalent TLS validation overrides. Check:
   - Does the callback unconditionally return `true`? If so, is this intentional and documented?
   - Could a callback that suppresses validation be reached in production code paths?
   - Are certificate chain errors selectively handled or blanket-ignored?
   - Is there a risk of MITM attack due to disabled validation?
   - [SPECIFIC_CALLBACK_LOCATIONS]

2. **TLS protocol version configuration**: Check all `SslStream`, `HttpClient`, `SslClientAuthenticationOptions`, or equivalent TLS setup:
   - Which TLS versions are enabled? Are deprecated versions (SSL 3.0, TLS 1.0, TLS 1.1) allowed?
   - Is the minimum TLS version explicitly set, or does it rely on OS defaults?
   - Could a server force a protocol downgrade?
   - [SPECIFIC_TLS_CONFIG_LOCATIONS]

3. **Cipher suite selection**: Check for explicit cipher suite configuration:
   - Are weak cipher suites (RC4, DES, 3DES, NULL, EXPORT) allowed?
   - Is cipher suite selection left to OS defaults (acceptable) or explicitly configured?
   - Are forward-secrecy cipher suites preferred (ECDHE, DHE)?

4. **ALPN (Application-Layer Protocol Negotiation)**: If the app uses ALPN:
   - Are ALPN identifiers correct per the relevant RFC/specification?
   - Is the server's ALPN response validated?
   - Could a mismatch cause fallback to an insecure protocol?
   - [SPECIFIC_ALPN_LOCATIONS]

5. **Certificate and key material handling**: Check all X509Certificate2, X509Chain, or equivalent usage:
   - Are certificate objects properly disposed after use?
   - Are private keys ever loaded unnecessarily?
   - Could certificate data be logged or serialized insecurely?
   - Is certificate pinning implemented where appropriate?
   - Are certificates stored securely (not embedded in source code)?

6. **TLS error handling**: Check how TLS handshake failures are handled:
   - Do error messages reveal internal details (cipher negotiation, protocol version, etc.)?
   - Is there automatic fallback to unencrypted connections on TLS failure?
   - Are TLS errors logged appropriately for debugging without leaking secrets?

Search patterns:
- TLS setup: `SslStream, SslClientAuthenticationOptions, SslServerAuthenticationOptions, HttpClientHandler, ServicePointManager, SecurityProtocolType, SslProtocols`
- Callbacks: `RemoteCertificateValidationCallback, ServerCertificateCustomValidationCallback, userCertificateValidationCallback`
- Certificate: `X509Certificate, X509Certificate2, X509Chain, X509Store, StoreName, StoreLocation`
- Protocol: `Tls11, Tls12, Tls13, Ssl3, SslProtocols.None, EnabledSslProtocols`
- ALPN: `ApplicationProtocol, SslApplicationProtocol, alpn`
- Cipher: `CipherSuitesPolicy, CipherAlgorithmType, HashAlgorithmType`

Provide a detailed findings report with file paths, line numbers, code snippets, and severity ratings (Critical/High/Medium/Low/Info). For TLS inspection tools, distinguish between intentional security bypasses (documented, scoped) and accidental misconfigurations. **Do NOT include actual credential values, API keys, or tokens in your output** — use `[REDACTED]` placeholders.
~~~

## Customization Guide

> **⚠️ Placeholder Safety:** Placeholder values are substituted directly into the prompt text. A crafted value could act as a prompt injection. Only use placeholder values you trust — do not accept them from untrusted sources.

| Placeholder | Example Values |
|-------------|---------------|
| `[APPLICATION_NAME]` | sql-cert-inspector, MyWebApp, ApiGateway |
| `[REPO_PATH]` | Full path to the repository root |
| `[TLS_USAGE_DESCRIPTION]` | `negotiates TLS connections to SQL Server for certificate inspection — it intentionally accepts any certificate to inspect it, not to validate trust`, `connects to external APIs over HTTPS`, `terminates TLS for incoming client connections` |
| `[SPECIFIC_CALLBACK_LOCATIONS]` | `Check TdsPreloginClient.cs SslStream creation`, `Check HttpClientFactory configuration` |
| `[SPECIFIC_TLS_CONFIG_LOCATIONS]` | `Check SslClientAuthenticationOptions in TdsPreloginClient.cs`, `Check Kestrel HTTPS configuration` |
| `[SPECIFIC_ALPN_LOCATIONS]` | `Check TDS 8.0 ALPN identifier "tds/8.0" in TdsPreloginClient.cs`, `Check HTTP/2 ALPN "h2"` |

## What Good Looks Like

- Certificate validation disabled ONLY where intentional, with clear documentation
- Minimum TLS version set to 1.2 (or 1.3 where supported)
- No deprecated protocols (SSL 3.0, TLS 1.0/1.1) enabled
- ALPN identifiers match the relevant protocol specification
- X509Certificate2 objects disposed via `using` statements
- No private keys loaded unless required for mutual TLS
- TLS errors don't fall back to unencrypted connections silently
- Cipher suite selection defers to OS defaults or explicitly selects strong suites
- Certificate pinning considered for known server endpoints
