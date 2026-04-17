# CORS & Error Information Disclosure

**Domain:** Cross-origin policy, HTTP security headers, error response safety  
**Agent Type:** `explore`  
**Priority:** Medium

## Purpose

Audit whether the application's cross-origin policy is appropriately restrictive and whether error responses leak internal implementation details. Overly permissive CORS enables cross-site attacks, and verbose error messages give attackers a roadmap of the application's internals.

> **⚠️ Authorized Use Only:** Use this prompt only to audit codebases you own or have explicit authorization to assess.

## Prompt

~~~
You are conducting a security assessment of the [APPLICATION_NAME] repository at [REPO_PATH].

Your task: **Audit CORS configuration and error information disclosure** across the service API.

Specifically investigate:

1. **CORS configuration**: Check [API_ENTRY_POINT]:
   - Is CORS middleware configured?
   - What origins are allowed? Is it wildcard (`*`)?
   - Are credentials allowed with CORS? (Wildcard + credentials is a critical misconfiguration)
   - Are specific HTTP methods and headers restricted?
   - If no CORS is configured, document the implications for the deployment model

2. **Error information disclosure**: Check the entire [SERVICE_PROJECT]:
   - Is there global exception handling middleware?
   - Do API responses include stack traces in production?
   - Are internal server names, file paths, or connection details leaked in error responses?
   - Check try-catch blocks in endpoint handlers — what error details are returned to the client?
   - Is there a difference between Development and Production error handling?

3. **HTTP security headers**: Check [API_ENTRY_POINT] middleware pipeline:
   - Is HSTS configured?
   - Are security headers set? (X-Content-Type-Options, X-Frame-Options, Content-Security-Policy, Referrer-Policy)
   - Is the Server header suppressed to avoid version disclosure?
   - Are cache-control headers set appropriately for sensitive responses?

4. **Response content safety**: Check API responses:
   - Do successful responses leak internal implementation details (class names, internal IDs, debug info)?
   - Are error codes generic enough to not reveal system internals?
   - Is sensitive data exposed via API responses (even to authenticated users) beyond what's necessary?

5. **Real-time channel CORS**: Check if real-time communication ([HUB_OR_WEBSOCKET_CLASSES]) has separate CORS handling:
   - Can any origin connect to real-time endpoints?
   - Are WebSocket connections restricted by origin?

Search patterns:
- Keywords: AddCors, UseCors, AllowAnyOrigin, WithOrigins, UseExceptionHandler, ProblemDetails, DeveloperExceptionPage, UseHsts, X-Frame-Options, X-Content-Type-Options, Content-Security-Policy, ServerHeader, app.UseStatusCodePages

Provide a detailed findings report with file paths, line numbers, and severity ratings (Critical/High/Medium/Low/Info).
~~~

## Customization Guide

> **⚠️ Placeholder Safety:** Placeholder values are substituted directly into the prompt text. A crafted value could act as a prompt injection. Only use placeholder values you trust — do not accept them from untrusted sources.

| Placeholder | Example Values |
|-------------|---------------|
| `[APPLICATION_NAME]` | MyWebApp, SqlAgMonitor |
| `[REPO_PATH]` | Full path to the repository root |
| `[API_ENTRY_POINT]` | `Program.cs`, `Startup.cs`, `app.ts`, `main.py` |
| `[SERVICE_PROJECT]` | `src/MyApp.Service/`, `server/`, `api/` |
| `[HUB_OR_WEBSOCKET_CLASSES]` | `MonitorHub.cs`, `ChatHub.cs`, `WebSocketHandler.ts` |

## What Good Looks Like

- CORS restricted to specific known origins (never wildcard in production)
- Credentials not allowed with wildcard origins
- Global exception handler returns generic error responses in production
- Stack traces only shown in Development environment
- HSTS enabled with appropriate max-age
- Security headers set via middleware (X-Content-Type-Options: nosniff, etc.)
- Server header suppressed or genericized
- Error responses use problem details format without internal details
