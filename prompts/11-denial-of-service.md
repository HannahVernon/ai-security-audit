# Denial of Service Resilience

**Domain:** Resource exhaustion, rate limiting, backpressure  
**Agent Type:** `explore`  
**Priority:** Medium

## Purpose

Audit whether the application protects against denial-of-service conditions — both from malicious external actors and from internal failure cascades (e.g., a flapping dependency triggering alert storms). Even internal-only services need DoS protection against misconfigured clients and runaway processes.

> **⚠️ Authorized Use Only:** Use this prompt only to audit codebases you own or have explicit authorization to assess.

## Prompt

~~~
You are conducting a security assessment of the [APPLICATION_NAME] repository at [REPO_PATH].

Your task: **Audit for Denial of Service (DoS) vulnerabilities** across the codebase.

Specifically investigate:

1. **API rate limiting**: Check [API_ENTRY_POINT]:
   - Is there rate limiting middleware on any endpoints?
   - Can authentication endpoints be brute-forced?
   - Can expensive endpoints (data import, report generation, bulk operations) be called repeatedly?
   - Are there any endpoints that trigger expensive operations (large queries, file I/O, external calls)?

2. **Connection limits**: Check [CONNECTION_CLASSES]:
   - Is there a maximum number of concurrent connections (HTTP, WebSocket, database)?
   - Can a single client open unlimited connections?
   - Is there message size limiting on real-time channels?
   - Is there backpressure handling?

3. **Database/storage resource exhaustion**: Check [STORAGE_CLASSES]:
   - Are queries time-bounded (query timeout, cancellation tokens)?
   - Can queries return unbounded result sets?
   - Is there disk space monitoring?
   - Can purge/cleanup processes be overwhelmed?

4. **Connection pool exhaustion**: Check [CONNECTION_POOL_CLASSES]:
   - Is there a connection pool limit?
   - Can reconnection loops consume excessive resources?
   - Is there exponential backoff with a cap?

5. **Memory exhaustion**: Search for patterns that could consume unbounded memory:
   - Collections that grow without bounds (List, Dictionary, arrays without capacity limits)
   - Large string concatenation or StringBuilder without limits
   - Unbounded buffering in streaming/reactive pipelines
   - Processing of arbitrarily large input files or datasets

6. **Notification flooding**: Check [NOTIFICATION_CLASSES]:
   - Is there notification/alert rate limiting or cooldown?
   - Can a flapping dependency trigger notification storms?
   - Is there a maximum queue size for outbound notifications?

Search patterns:
- Keywords: MaxConnections, MaxConcurrent, Timeout, CancellationToken, Throttle, Buffer, Take, Limit, capacity, cooldown, backoff, RateLimit, SlidingWindow, FixedWindow

Provide a detailed findings report with file paths, line numbers, and severity ratings (Critical/High/Medium/Low/Info). **Do NOT include actual credential values, API keys, tokens, or passwords in your output** — use `[REDACTED]` placeholders.
~~~

## Customization Guide

> **⚠️ Placeholder Safety:** Placeholder values are substituted directly into the prompt text. A crafted value could act as a prompt injection. Only use placeholder values you trust — do not accept them from untrusted sources.

| Placeholder | Example Values |
|-------------|---------------|
| `[APPLICATION_NAME]` | MyWebApp, SqlAgMonitor |
| `[REPO_PATH]` | Full path to the repository root |
| `[API_ENTRY_POINT]` | `Program.cs`, `Startup.cs`, `app.ts`, `main.py` |
| `[CONNECTION_CLASSES]` | `MonitorHub.cs`, `WebSocketHandler.ts`, `ConnectionManager.cs` |
| `[STORAGE_CLASSES]` | `DbContext`, `DuckDbConnectionManager.cs`, `RedisCache.ts` |
| `[CONNECTION_POOL_CLASSES]` | `ReconnectingConnectionWrapper.cs`, `ConnectionPool.ts`, `pgPool` |
| `[NOTIFICATION_CLASSES]` | `AlertEngine.cs`, `EmailService.ts`, `NotificationWorker.py` |

## What Good Looks Like

- Rate limiting on authentication and expensive endpoints
- Connection limits with graceful rejection
- Query timeouts and cancellation token propagation
- Bounded collections with eviction policies
- Exponential backoff with jitter and maximum cap on retries
- Alert cooldown/deduplication to prevent notification storms
- Disk space monitoring with alerts before exhaustion
