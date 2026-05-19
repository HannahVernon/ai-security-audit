# Server-Side Request Forgery (SSRF)

**Domain:** Outbound request validation, URL allowlisting, internal network protection, cloud metadata access  
**Agent Type:** `explore`  
**Priority:** High

## Purpose

Audit all outbound HTTP and network request paths for Server-Side Request Forgery risks.  This prompt covers OWASP A10 (Server-Side Request Forgery) and focuses on whether the server can be tricked into making requests to unintended destinations, including internal services and cloud metadata endpoints.

> **⚠️ Authorized Use Only:** Use this prompt only to audit codebases you own or have explicit authorization to assess.

## Prompt

~~~
You are conducting a security assessment of the [APPLICATION_NAME] repository at [REPO_PATH].

Your task: **Audit outbound HTTP and network request handling for SSRF risks** across the entire codebase.

Specifically investigate:

1. **User-controlled URLs**: Find all places where the application makes outbound HTTP or network requests using URLs, URIs, hosts, or ports derived from user input, configuration, or database values.  Check [HTTP_CLIENT_CLASSES], [WEBHOOK_CLASSES], [CONFIG_CLASSES], and [EXTENSIONS].
   - Identify use of `HttpClient`, `WebClient`, `HttpWebRequest`, `RestClient`, `fetch`, `axios`, `requests`, `http.Get`, `url.Parse`, `new URL`, `Uri`, `UriBuilder`, `DownloadString`, `DownloadFile`, `GetAsync`, `PostAsync`, `SendAsync`, and `OpenRead`
   - Determine whether request targets are fully or partially attacker-controlled
   - Check whether configuration injection or database tampering could redirect internal service calls to attacker-controlled destinations

2. **URL validation**: When URLs are user-supplied or indirectly influenced by untrusted data, determine whether validation exists.  Check for:
   - Protocol allowlisting, only `http` and `https`, not `file://`, `gopher://`, `dict://`, or other dangerous schemes
   - Hostname and IP validation against internal or reserved ranges, including `127.0.0.0/8`, `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`, `169.254.169.254`, `::1`, and `fd00::/8`
   - DNS rebinding protection, resolve the hostname before the request and validate the resolved IP address
   - Redirect handling, determine whether the client follows redirects and whether each redirect hop is re-validated before connection

3. **Cloud metadata endpoint access**: Determine whether a crafted URL could reach cloud metadata services such as `169.254.169.254`, `metadata.google.internal`, or `169.254.170.2`.  Treat this as critical in cloud-hosted applications.

4. **Webhook and callback URLs**: Determine whether the application accepts webhook, callback, or notification URLs from users or tenants.  Check whether these URLs can target internal services, loopback addresses, link-local addresses, or metadata endpoints.  Check [WEBHOOK_CLASSES].

5. **Import and fetch features**: Determine whether the application fetches remote images, feeds, documents, APIs, previews, or other resources based on user-provided URLs.  URL preview and unfurling features are classic SSRF vectors.

6. **Internal service communication**: Determine whether internal service URLs are hardcoded, trusted configuration values, or user-controllable.  Check whether configuration injection could redirect internal API calls to attacker-controlled servers.  Check [CONFIG_CLASSES].

7. **DNS resolution and rebinding**: Determine whether the application resolves user-supplied hostnames before connecting.  Check whether IP validation occurs before and after DNS resolution, and whether cached resolution results are used to reduce rebinding risk.

Search keywords: `HttpClient`, `WebClient`, `HttpWebRequest`, `RestClient`, `WebRequest`, `fetch`, `axios`, `request`, `http.Get`, `http.Post`, `urllib`, `requests.get`, `requests.post`, `url.Parse`, `new URL`, `Uri`, `UriBuilder`, `DownloadString`, `DownloadFile`, `GetAsync`, `PostAsync`, `SendAsync`, `OpenRead`, `webhook`, `callback`, `redirect`, `Location`.

Provide a detailed findings report with file paths, line numbers, code snippets, and severity ratings.  Explain the exact SSRF path, what attacker-controlled input reaches the request target, and whether internal network access or cloud metadata access is possible.  Do NOT include actual credential values, API keys, tokens, passwords, connection strings, or other secret material in your output - use `[REDACTED]` placeholders.
~~~

## Customization Guide

> **⚠️ Placeholder Safety:** Placeholder values are substituted directly into the prompt text.  A crafted value could act as a prompt injection.  Only use placeholder values you trust.  Do not accept placeholder values from untrusted sources.

Placeholder | Example Values
------------|---------------
`[APPLICATION_NAME]` | `MyWebApp`, `BillingPortal`, `ApiGateway`
`[REPO_PATH]` | Full path to the repository root
`[HTTP_CLIENT_CLASSES]` | `HttpClientFactory` wrappers, `ApiClient.cs`, `services/http.ts`, `requests_session.py`
`[WEBHOOK_CLASSES]` | `WebhookController.cs`, `WebhookRegistrationService.ts`, `callbacks.py`
`[CONFIG_CLASSES]` | `appsettings.json` loaders, `ServiceEndpointOptions.cs`, `config.ts`
`[EXTENSIONS]` | `Inspect plugins/`, `Check import jobs`, `Review preview and unfurl features`

## What Good Looks Like

- URL allowlist, an explicit list of permitted domains or IPs, rather than a blocklist
- Protocol restricted to `https` only, or `http` and `https` with explicit justification
- Resolved IP validated against internal and reserved ranges before connection
- Redirect following disabled, or each redirect re-validated at every hop
- Cloud metadata endpoints, especially `169.254.169.254`, explicitly blocked
- Webhook URLs validated against an allowlist before registration or delivery
- Internal service URLs hardcoded or loaded only from trusted configuration, never from user-controllable input
- DNS resolution results cached and validated to reduce rebinding risk

## Relationship to Other Prompts

Prompt | Relationship
-------|-------------
03 - File Path & Process Execution | Prompt 03 audits *local* file access and process execution.  This prompt audits *outbound network* requests initiated by the server.
17 - Network Attacker & Wire-Level Threats | Prompt 17 audits the *wire-level* security of network communication.  This prompt audits whether the *server can be tricked into making requests to unintended destinations*.
