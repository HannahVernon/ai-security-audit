# Real-Time Channel Security (SignalR / WebSocket)

**Domain:** SignalR hubs, WebSocket endpoints, real-time data broadcast security  
**Agent Type:** `explore`  
**Priority:** High

## Purpose

Audit the security of real-time communication channels. These are often overlooked during security reviews because they don't follow the traditional request-response pattern — but they can expose sensitive data streams, accept unauthorized commands, and bypass standard HTTP security controls.

> **⚠️ Authorized Use Only:** Use this prompt only to audit codebases you own or have explicit authorization to assess.

## Prompt

~~~
You are conducting a security assessment of the [APPLICATION_NAME] repository at [REPO_PATH].

Your task: **Audit real-time communication channel security** across the service and client applications.

The application uses [REALTIME_TECHNOLOGY] for real-time updates.

Specifically investigate:

1. **Channel definition**: Find the real-time endpoint classes ([HUB_OR_HANDLER_CLASSES]):
   - What methods are exposed to clients?
   - Are methods decorated with authorization attributes?
   - Can unauthenticated clients invoke methods?
   - What data do methods return or broadcast?

2. **Channel registration**: Check [API_ENTRY_POINT]:
   - How is the endpoint mapped?
   - Is authentication required for the endpoint?
   - What transport protocols are allowed? (WebSockets, ServerSentEvents, LongPolling)
   - Are transport-specific security considerations addressed?

3. **Server-to-client messages**: Check what data the service pushes to clients:
   - [BROADCAST_CLASSES] — what data is broadcast?
   - Could sensitive data (credentials, connection strings, internal paths) be included in broadcasts?
   - Is there any filtering based on client identity or role?
   - Could an eavesdropping client learn about infrastructure topology?

4. **Client-to-server messages**: Check if the channel accepts input from clients:
   - Can clients trigger actions (state changes, configuration, control operations)?
   - Are client messages validated and sanitized?
   - Could a malicious client send crafted messages to exploit server logic?
   - Are message sizes limited?

5. **Client connection**: Check [CLIENT_CLASSES]:
   - How does the client authenticate to the channel?
   - Is the auth token in the query string (visible in logs) or in headers?
   - Are reconnection attempts rate-limited?
   - Is the connection encrypted (wss:// not ws://)?

6. **Connection lifecycle**:
   - Are connect/disconnect events handled and logged?
   - Are connections tracked for monitoring?
   - Is there a maximum connection limit?
   - Are stale connections cleaned up?

Search patterns:
- Keywords: Hub, MapHub, HubConnection, HubConnectionBuilder, WithUrl, Clients.All, SendAsync, InvokeAsync, OnConnectedAsync, OnDisconnectedAsync, WebSocket, ws://, wss://, Socket.IO, emit, on(

Provide a detailed findings report with file paths, line numbers, and severity ratings (Critical/High/Medium/Low/Info).
~~~

## Customization Guide

> **⚠️ Placeholder Safety:** Placeholder values are substituted directly into the prompt text. A crafted value could act as a prompt injection. Only use placeholder values you trust — do not accept them from untrusted sources.

| Placeholder | Example Values |
|-------------|---------------|
| `[APPLICATION_NAME]` | MyWebApp, SqlAgMonitor |
| `[REPO_PATH]` | Full path to the repository root |
| `[REALTIME_TECHNOLOGY]` | `SignalR`, `WebSockets`, `Socket.IO`, `gRPC streaming` |
| `[HUB_OR_HANDLER_CLASSES]` | `MonitorHub.cs`, `ChatHub.cs`, `WebSocketHandler.ts` |
| `[API_ENTRY_POINT]` | `Program.cs`, `Startup.cs`, `app.ts` |
| `[BROADCAST_CLASSES]` | `MonitoringWorker.cs`, `NotificationService.ts`, `EventBroadcaster.py` |
| `[CLIENT_CLASSES]` | `ServiceMonitoringClient.cs`, `hubConnection.ts`, `socket_client.py` |

## What Good Looks Like

- Real-time endpoints require authentication before receiving data
- Auth tokens sent in headers, not query strings (to avoid log exposure)
- Server-to-client broadcasts contain only data the client is authorized to see
- Client-to-server methods are authorized and input-validated
- Connection limits prevent resource exhaustion
- WebSocket connections use `wss://` (encrypted), never `ws://`
- Reconnection has exponential backoff with maximum retry limits
- Connect/disconnect events are logged for audit trail
