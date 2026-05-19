# Privacy & PII Handling

**Domain:** Personally identifiable information, data minimization, consent, retention, right to deletion, analytics tracking  
**Agent Type:** `explore`  
**Priority:** Medium

## Purpose

Audit how the application collects, processes, stores, shares, and deletes personally identifiable information.  This prompt focuses on privacy principles and personal data handling, including data minimization, consent, retention, deletion, and downstream sharing.

> **⚠️ Authorized Use Only:** Use this prompt only to audit codebases you own or have explicit authorization to assess.

## Prompt

~~~
You are conducting a security assessment of the [APPLICATION_NAME] repository at [REPO_PATH].

Your task: **Audit privacy and personally identifiable information handling** across the application.

Specifically investigate:

1. **PII inventory**: What personal data does the application collect, process, or store?  Check:
   - User models and entities for PII fields, such as name, email, phone, address, date of birth, SSN or SIN, IP address, and device identifiers
   - Database schemas and migration files for PII columns
   - API request and response models, especially [API_RESPONSE_CLASSES], that include PII
   - Search indices, caches, or denormalized stores that duplicate PII
   - [USER_MODEL_CLASSES]

2. **Data minimization**: Does the application collect more PII than needed?  Check:
   - Whether optional PII fields, such as phone, address, or date of birth, are collected when not required for functionality
   - Whether full datasets are returned from APIs when only a subset is needed
   - Whether PII fields appear in list or search endpoints that only need identifiers and display names
   - Whether analytics or telemetry events in [ANALYTICS_CLASSES] collect PII unnecessarily

3. **PII in logs and error reports**: Check:
   - Whether user names, emails, IP addresses, or other PII are written to application logs
   - Whether PII values are included in exception messages or stack traces
   - Whether crash reporting services, such as Application Insights or Sentry, receive PII
   - Whether structured log objects or JSON payloads include PII fields
   - [LOGGING_CLASSES]

4. **Retention and deletion**: Check:
   - Whether a data retention policy is implemented in code, including automatic deletion after a defined period
   - Whether user data can be fully deleted when requested, including right to erasure or right to be forgotten scenarios
   - Whether deletion cascades to all copies, including logs, caches, backups, search indices, and analytics stores
   - Whether soft-deleted records are purged after a defined period

5. **Consent and purpose limitation**: Check:
   - Whether there is evidence of consent collection before PII processing
   - Whether PII is used only for the purpose it was collected, and not repurposed for marketing, analytics, or unrelated processing
   - Whether users can manage consent preferences
   - Whether PII is shared with third-party services, and if so, which ones and with what justification

6. **PII in URLs and referrer headers**: Check:
   - Whether PII values, such as emails, user IDs, or names, are included in URL paths or query strings
   - Whether referrer headers could leak PII to third-party sites
   - Whether PII in URLs could be logged by web servers, proxies, or CDNs

7. **Data anonymization and pseudonymization**: Check:
   - Whether analytics or reporting datasets are anonymized before use
   - Whether PII is pseudonymized, or replaced with tokens, where full identity is not needed
   - Whether pseudonymized data could be re-identified by combining it with other available data

Search patterns:
- Files: `**/*.[EXTENSIONS]`
- Keywords: email, name, firstName, lastName, phone, address, dateOfBirth, dob, ssn, sin, nationalId, ipAddress, userAgent, deviceId, geoLocation, latitude, longitude, PersonalData, PII, GDPR, CCPA, consent, retention, delete, purge, anonymize, pseudonymize, redact, mask, obfuscate, tracking, analytics, telemetry

Provide a detailed findings report with file paths, line numbers, and severity ratings (Critical/High/Medium/Low/Info).  **Do NOT include actual credential values, personal data values, email addresses, phone numbers, access tokens, or other sensitive identifiers in your output.**  Report them as `[REDACTED]` or `[PII_REDACTED]`.
~~~

## Customization Guide

> **⚠️ Placeholder Safety:** Placeholder values are substituted directly into the prompt text.  A crafted value could act as a prompt injection.  Only use placeholder values you trust - do not accept them from untrusted sources.

Placeholder | Example Values
------------|---------------
`[APPLICATION_NAME]` | MyPortal, CustomerCenter
`[REPO_PATH]` | Full path to the repository root
`[USER_MODEL_CLASSES]` | `User.cs`, `CustomerProfile`, `PersonEntity`, `AccountHolder` 
`[API_RESPONSE_CLASSES]` | `UserDto`, `ProfileResponse`, `CustomerSummary`, `SearchResultDto`
`[LOGGING_CLASSES]` | `ILogger`, `Serilog`, `RequestLoggingMiddleware`, `AuditLogger`
`[ANALYTICS_CLASSES]` | `TelemetryService`, `AnalyticsEvent`, `TrackingClient`, `AppInsightsInitializer`
`[EXTENSIONS]` | `cs,json,sql,ts,js,py,yaml,config`

## What Good Looks Like

- PII inventory documented, each field has a stated purpose
- Only necessary PII collected, aligned with data minimization
- PII excluded from application logs, use user IDs instead of names or emails
- Data retention policy implemented with automatic purging
- Right-to-deletion workflow that cascades to all data stores
- PII never appears in URL paths or query strings
- Analytics data anonymized or pseudonymized where possible
- Consent recorded before PII processing, users can manage preferences
- Third-party PII sharing documented and justified
- Crash and error reports scrub PII before transmission

## Relationship to Other Prompts

Prompt 01 covers credential handling.  Prompt 04 covers data at rest security, including encryption and file permissions.  Prompt 19 covers logging completeness.  This prompt specifically audits personal data handling from a privacy and compliance perspective.
