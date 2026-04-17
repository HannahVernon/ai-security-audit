# Authorization & API Access Control

**Domain:** Authentication enforcement, endpoint authorization, role-based access  
**Agent Type:** `explore`  
**Priority:** High

## Purpose

Audit whether API endpoints, hubs, and service interfaces enforce proper authentication and authorization. A single unprotected endpoint can expose sensitive data or allow unauthorized actions — even if the login system itself is well-designed.

> **⚠️ Authorized Use Only:** Use this prompt only to audit codebases you own or have explicit authorization to assess.

## Prompt

~~~
You are conducting a security assessment of the [APPLICATION_NAME] repository at [REPO_PATH].

Your task: **Audit authorization and API access control** across all service endpoints and real-time communication channels.

Specifically investigate:

1. **Endpoint authorization**: Check [API_ENTRY_POINT] for ALL mapped endpoints:
   - Which endpoints require authentication? Which are anonymous?
   - Are sensitive endpoints (config import/export, control operations, admin functions) protected?
   - Is there role-based access control, or does any authenticated user get full access?
   - Check for authorization attributes or middleware ([AUTHZ_ATTRIBUTES])

2. **Real-time channel security**: Check [HUB_OR_WEBSOCKET_CLASSES]:
   - Are real-time connections authenticated? Can anonymous clients connect?
   - Are channel methods validated for authorization?
   - Can a connected client invoke sensitive operations?
   - Is there connection limiting?

3. **Token security**: Check [TOKEN_SERVICE_CLASSES]:
   - Token expiration — how long are tokens valid?
   - Token refresh mechanism — is there one?
   - Are tokens properly validated (issuer, audience, expiration, signature)?
   - Can tokens be revoked?
   - Are tokens exposed in URLs (query strings, WebSocket connection URLs)?

4. **Login/authentication endpoint**:
   - Is there brute-force protection (rate limiting, account lockout)?
   - Are failed login attempts logged?
   - Is timing attack protection in place (constant-time comparison)?

5. **Client-side authorization**:
   - How do clients authenticate to the service?
   - Are credentials stored securely between sessions?
   - Is the auth token refreshed automatically?

Search patterns:
- Keywords: [AUTHZ_ATTRIBUTES], AllowAnonymous, RequireAuthorization, MapHub, MapGet, MapPost, MapPut, MapDelete, Bearer, TokenValidationParameters, Claims, Roles, Policy

Provide a detailed findings report with file paths, line numbers, and severity ratings (Critical/High/Medium/Low/Info). **Do NOT include actual credential values, API keys, tokens, or passwords in your output** — use `[REDACTED]` placeholders.
~~~

## Customization Guide

> **⚠️ Placeholder Safety:** Placeholder values are substituted directly into the prompt text. A crafted value could act as a prompt injection. Only use placeholder values you trust — do not accept them from untrusted sources.

| Placeholder | Example Values |
|-------------|---------------|
| `[APPLICATION_NAME]` | MyWebApp, SqlAgMonitor |
| `[REPO_PATH]` | Full path to the repository root |
| `[API_ENTRY_POINT]` | `Program.cs`, `Startup.cs`, `routes/*.ts`, `urls.py` |
| `[HUB_OR_WEBSOCKET_CLASSES]` | `MonitorHub.cs`, `ChatHub.cs`, `WebSocketHandler.ts` |
| `[TOKEN_SERVICE_CLASSES]` | `JwtTokenService.cs`, `TokenProvider.ts`, `auth/jwt.py` |
| `[AUTHZ_ATTRIBUTES]` | `[Authorize]`, `@PreAuthorize`, `@login_required`, `authMiddleware` |

## What Good Looks Like

- Every endpoint explicitly declares its authorization requirement
- Sensitive operations require elevated roles (admin, operator)
- Tokens have short expiration with refresh mechanism
- Failed login attempts are logged and rate-limited
- Real-time connections require authentication before receiving data
- No sensitive data accessible to unauthenticated users
