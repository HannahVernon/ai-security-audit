# Network Attacker & Wire-Level Threats

**Domain:** Traffic interception, replay attacks, session hijacking, protocol downgrade, trust anchoring  
**Agent Type:** `explore`  
**Priority:** High

## Purpose

Audit the application from the perspective of an on-path network attacker (sometimes called a man-in-the-middle).  This prompt complements prompt 09 (TLS Configuration) and prompt 10 (Authorization) by shifting focus from *how TLS is configured* and *whether endpoints check auth* to *what happens when an attacker can observe, replay, or modify traffic on the wire*.

Key questions this prompt answers that other prompts do not:

- Can a captured JWT or session token be replayed from a different machine?
- Does the client validate the server's identity, or can it be redirected to a rogue service?
- Are there anti-replay mechanisms (nonces, timestamps, sequence numbers) on sensitive operations?
- Can an attacker force a protocol downgrade from encrypted to unencrypted transport?
- If a token is stolen, can it be revoked before it expires?

> **⚠️ Authorized Use Only:** Use this prompt only to audit codebases you own or have explicit authorization to assess.

## Prompt

~~~
You are conducting a security assessment of the [APPLICATION_NAME] repository at [REPO_PATH].

Your task: **Audit the application's resilience to network-level attacks** — assume an attacker who can observe, intercept, modify, or replay network traffic between the client and server.

Specifically investigate:

1. **Token theft and replay**: Check [TOKEN_SERVICE_CLASSES] and [CLIENT_AUTH_CLASSES]:
   - If an attacker captures a valid JWT or session token from the wire, can they reuse it from a different machine or IP?
   - Are tokens bound to any client-specific context (IP address, client fingerprint, TLS channel binding)?
   - Is there a token revocation mechanism?  How quickly does revocation take effect?
   - Are refresh tokens (if any) single-use or reusable?
   - What is the token lifetime?  Could a stolen token be useful for hours or days?
   - Are tokens transmitted in URL query strings where they could appear in server logs, browser history, or referrer headers?

2. **Session hijacking**: Check how authenticated sessions are maintained:
   - After initial authentication, what proves the client's identity on subsequent requests?
   - Can an attacker with a captured token establish a new connection and impersonate the user?
   - Are there any session-binding mechanisms (mutual TLS, channel binding tokens)?
   - Is there session activity monitoring that would detect concurrent sessions?

3. **Protocol downgrade attacks**: Check [CONNECTION_SETUP_CLASSES]:
   - Can the application fall back from HTTPS/WSS to HTTP/WS?
   - Is HSTS configured to prevent downgrade on web endpoints?
   - Does the client enforce TLS, or does it accept unencrypted connections if TLS fails?
   - Are there configuration options that could disable encryption (e.g., a "use TLS" toggle)?
   - If TLS negotiation fails, does the client retry without TLS?

4. **Server identity validation (trust anchoring)**: Check [CLIENT_CONNECTION_CLASSES]:
   - Does the client validate the server's TLS certificate?
   - Could a user or config file redirect the client to a rogue server?
   - Is there certificate pinning or any trust anchor beyond the OS certificate store?
   - If the client stores the server URL in a config file, could an attacker who gains write access to that file redirect all traffic?
   - Are there warnings when the server identity changes unexpectedly?

5. **Message replay and tampering**: Check [API_AND_HUB_CLASSES]:
   - Are sensitive operations (failover, configuration changes, control commands) protected against replay?
   - Could an attacker capture a "failover AG" command and replay it later to cause disruption?
   - Are messages signed or integrity-protected beyond TLS?
   - Is there any nonce, timestamp, or sequence number mechanism for critical operations?
   - Are idempotency keys used for state-changing operations?

6. **Credential exposure on the wire**: Check all network communication:
   - Are passwords ever transmitted in cleartext (even over TLS, are they in URL parameters)?
   - Are credentials sent in HTTP headers, request bodies, or query strings?
   - Could a TLS-terminating proxy or load balancer expose credentials in access logs?
   - Are WebSocket/SignalR connection URLs logged server-side (including any auth tokens in the query string)?

7. **Reconnection security**: Check [RECONNECTION_CLASSES]:
   - When a connection drops and reconnects, is re-authentication required?
   - Could an attacker hijack a reconnection attempt?
   - Is the reconnection token (if any) different from the original auth token?
   - Are reconnection attempts rate-limited to prevent brute-force?

Search patterns:
- Token handling: Bearer, Authorization, access_token, token, jwt, refresh, revoke, blacklist, HubConnectionBuilder, WithUrl, accessTokenProvider
- TLS enforcement: https, http, wss, ws, Scheme, UseHttps, RequireHttps, HttpsRedirection, HSTS, Strict-Transport-Security
- Session: session, cookie, Set-Cookie, SessionId, channel_binding
- Replay: nonce, timestamp, sequence, idempotency, replay, MessageId, RequestId
- Config URLs: ServerUrl, BaseAddress, ServiceUrl, Endpoint, Host, Port

Provide a detailed findings report with file paths, line numbers, code snippets, and severity ratings (Critical/High/Medium/Low/Info).  For each finding, describe the specific attack scenario an on-path attacker could execute.  **Do NOT include actual credential values, API keys, tokens, or passwords in your output** — use `[REDACTED]` placeholders.
~~~

## Customization Guide

> **⚠️ Placeholder Safety:** Placeholder values are substituted directly into the prompt text.  A crafted value could act as a prompt injection.  Only use placeholder values you trust — do not accept them from untrusted sources.

Placeholder | Example Values
------------|---------------
`[APPLICATION_NAME]` | MyWebApp, SqlAgMonitor, ChatService
`[REPO_PATH]` | Full path to the repository root
`[TOKEN_SERVICE_CLASSES]` | `JwtTokenService.cs`, `TokenProvider.ts`, `auth/jwt.py`
`[CLIENT_AUTH_CLASSES]` | `ServiceMonitoringClient.cs`, `ApiClient.ts`, `http_client.py`
`[CONNECTION_SETUP_CLASSES]` | `Program.cs` (Kestrel config), `ServiceHost.cs`, `server.ts`
`[CLIENT_CONNECTION_CLASSES]` | `ServiceConnectionViewModel.cs`, `ApiClient.cs`, `HubConnection` setup
`[API_AND_HUB_CLASSES]` | `MonitorHub.cs`, `ApiController.cs`, `routes/*.ts`
`[RECONNECTION_CLASSES]` | `HubConnection` retry policy, `ReconnectingConnectionWrapper.cs`, `WebSocketClient.ts`

## What Good Looks Like

- Tokens have short lifetimes (minutes, not hours) with secure refresh
- Token revocation mechanism exists and takes effect within seconds
- Tokens are bound to client context where possible (IP, TLS channel)
- No protocol downgrade path — TLS failure = connection failure, not fallback
- HSTS configured with long max-age on web endpoints
- Client validates server certificate; rogue server redirect requires more than config file access
- Sensitive operations have anti-replay protection (nonces, idempotency keys, or timestamps)
- Passwords never appear in URLs, query strings, or server access logs
- Reconnection requires re-authentication or uses a single-use reconnection token
- Auth tokens sent in HTTP headers, not query strings (to avoid log exposure)

## Relationship to Other Prompts

This prompt is designed to work alongside, not replace, related prompts:

Prompt | Relationship
-------|-------------
09 - TLS Configuration | Prompt 09 checks *how TLS is configured* (protocol versions, cipher suites, cert validation callbacks).  This prompt checks *what an attacker can do despite TLS* (token replay, downgrade, trust anchoring).
10 - Authorization | Prompt 10 checks *whether endpoints enforce auth*.  This prompt checks *whether auth tokens can be stolen and reused*.
14 - Real-Time Channels | Prompt 14 checks *hub authorization and broadcast filtering*.  This prompt checks *whether SignalR tokens can be intercepted and replayed*.
01 - Credentials | Prompt 01 checks *how credentials are stored*.  This prompt checks *how credentials are exposed on the wire*.
