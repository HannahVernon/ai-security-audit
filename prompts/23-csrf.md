# Cross-Site Request Forgery (CSRF)

**Domain:** Anti-forgery tokens, SameSite cookies, state-changing GET requests, CORS preflight  
**Agent Type:** `explore`  
**Priority:** High

## Purpose

Audit whether the application resists browser-driven cross-origin request abuse.  CSRF weaknesses often appear when cookie-backed authentication, overly broad cookie scope, permissive cross-origin handling, or unsafe endpoint design lets an attacker trigger authenticated actions without the victim's intent.

> **⚠️ Authorized Use Only:** Use this prompt only to audit codebases you own or have explicit authorization to assess.

## Prompt

~~~
You are conducting a security assessment of the [APPLICATION_NAME] repository at [REPO_PATH].

Your task: **Audit Cross-Site Request Forgery (CSRF) defenses** across browser-facing endpoints, form handlers, session cookies, and API routes.  Search relevant source files matching [EXTENSIONS].

Specifically investigate:

1. **Anti-forgery token implementation**: Check [FORM_HANDLER_CLASSES] and [API_CONTROLLER_CLASSES]:
   - Are state-changing endpoints (POST, PUT, DELETE, PATCH) protected by anti-forgery tokens?
   - Is the anti-forgery middleware or filter globally applied, or do individual endpoints opt in?
   - Are tokens validated server-side on every state-changing request?
   - Are there endpoints excluded from CSRF protection?  Are those exclusions justified?
   - Check framework-specific patterns such as `[ValidateAntiForgeryToken]`, `[AutoValidateAntiforgeryToken]`, `csrf_exempt`, `csurf`, and `authenticity_token`

2. **Cookie configuration**: Check [COOKIE_CONFIG_CLASSES]:
   - Do session or auth cookies use `SameSite=Strict` or `SameSite=Lax` with justification, rather than `SameSite=None`?
   - Are sensitive cookies marked `Secure` so they are only sent over HTTPS?
   - Are sensitive cookies marked `HttpOnly` so JavaScript cannot read them?
   - Is cookie path or domain scope broader than necessary?
   - Do session cookies use the `__Host-` prefix (strongest: requires `Secure`, no `Domain`, path `/`) or at minimum `__Secure-` (requires `Secure`)?  These prefixes prevent cookie injection from insecure subdomains or sibling paths.  See: Scott Helme, "Tough Cookies" (https://scotthelme.co.uk/tough-cookies/)

3. **State-changing GET requests**: Check [FORM_HANDLER_CLASSES] and [API_CONTROLLER_CLASSES]:
   - Do any GET endpoints modify data, trigger actions, or change server state?
   - Can links, image tags, or page loads trigger state changes when loaded by a browser?
   - Does logout happen via GET, enabling CSRF logout attacks?

4. **CORS and CSRF interaction**: Check [COOKIE_CONFIG_CLASSES] and [API_CONTROLLER_CLASSES]:
   - Does the CORS configuration allow credentials from other origins?
   - Could a permissive CORS policy undermine or bypass CSRF protections?
   - Are preflight (`OPTIONS`) requests handled correctly?

5. **Custom headers as CSRF defense**: Check [API_CONTROLLER_CLASSES]:
   - Do SPA endpoints rely on `X-Requested-With`, `X-CSRF-Token`, `X-XSRF-TOKEN`, or `Authorization` headers as the sole CSRF defense?
   - If custom headers are used, could misconfigured CORS make that defense ineffective?
   - Are there endpoints that accept both cookie-based and header-based authentication?

6. **API endpoint input handling**: Check [API_CONTROLLER_CLASSES]:
   - Do JSON or API endpoints accept `application/x-www-form-urlencoded` input, enabling cross-origin form submission?
   - Is `Content-Type` validation enforced?
   - Are there endpoints that accept both browser form submissions and API calls?

Search patterns:
- Keywords: AntiForgery, ValidateAntiForgeryToken, AutoValidateAntiforgeryToken, csrf, _csrf, csrfToken, csrf_exempt, csurf, authenticity_token, SameSite, HttpOnly, Secure, Set-Cookie, Cookie, X-Requested-With, X-CSRF-Token, X-XSRF-TOKEN, CookiePolicy, CookieOptions, __Host-, __Secure-, CookieSecurePolicy

Provide a detailed findings report with file paths, line numbers, and severity ratings (Critical/High/Medium/Low/Info).  For each finding, explain the browser-based attack path and whether it depends on cookies, permissive CORS, unsafe GET behavior, or missing anti-forgery validation.  **Do NOT include actual credential values, API keys, tokens, or passwords in your output** - use `[REDACTED]` placeholders.
~~~

## Customization Guide

> **⚠️ Placeholder Safety:** Placeholder values are substituted directly into the prompt text.  A crafted value could act as a prompt injection.  Only use placeholder values you trust - do not accept them from untrusted sources.

Placeholder | Example Values
------------|---------------
`[APPLICATION_NAME]` | MyWebApp, CustomerPortal, AdminConsole
`[REPO_PATH]` | Full path to the repository root
`[FORM_HANDLER_CLASSES]` | `Controllers/*.cs`, `Pages/**/*.cshtml.cs`, `views.py`, `routes/web.php`
`[COOKIE_CONFIG_CLASSES]` | `Program.cs`, `Startup.cs`, `CookiePolicyOptions`, `session.py`, `config/initializers/session_store.rb`
`[API_CONTROLLER_CLASSES]` | `ApiController.cs`, `routes/*.ts`, `views.py`, `controllers/api/*.rb`
`[EXTENSIONS]` | `*.cs`, `*.cshtml`, `*.ts`, `*.js`, `*.py`, `*.rb`

## What Good Looks Like

- Anti-forgery tokens protect all state-changing endpoints, and protection is applied globally rather than by easy-to-miss opt-in attributes
- `SameSite=Strict` is used on auth and session cookies, or `SameSite=Lax` is used with clear justification
- `Secure` and `HttpOnly` flags are set on all sensitive cookies
- Session cookies use `__Host-` prefix (preferred) or `__Secure-` prefix to prevent cookie injection from insecure subdomains
- No state-changing GET endpoints exist, including logout-via-GET patterns
- CORS does not allow credentials from untrusted origins
- API endpoints reject form-encoded input when they are intended to require `application/json`
- CSRF exemptions are documented, minimal, and justified
- Synchronizer token or double-submit cookie patterns are implemented correctly and validated server-side

## Relationship to Other Prompts

This prompt complements, not replaces, related prompts:

Prompt | Relationship
-------|-------------
13 - CORS & Error Information Disclosure | Prompt 13 checks CORS policy broadly and error leakage.  This prompt checks anti-forgery token mechanics, SameSite cookie posture, and whether CORS settings undermine CSRF defenses.
17 - Network Attacker & Wire-Level Threats | Prompt 17 focuses on token replay and traffic abuse on the wire.  This prompt focuses on browser-driven cross-origin request abuse using the victim's existing authenticated session.
10 - Authorization & API Access Control | Prompt 10 checks whether endpoints require authentication and authorization.  This prompt checks whether an already-authenticated browser can be tricked into invoking those endpoints.
