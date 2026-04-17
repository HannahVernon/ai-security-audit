# Deserialization & Input Validation

**Domain:** JSON/XML deserialization safety, input validation, injection via structured data  
**Agent Type:** `explore`  
**Priority:** High

## Purpose

Audit how the application deserializes external input and whether deserialized objects are validated before use. Unsafe deserialization can enable remote code execution, while missing input validation can allow injection of malicious configuration, paths, or connection targets.

> **⚠️ Authorized Use Only:** Use this prompt only to audit codebases you own or have explicit authorization to assess.

## Prompt

~~~
You are conducting a security assessment of the [APPLICATION_NAME] repository at [REPO_PATH].

Your task: **Audit deserialization and input validation** across the codebase.

Specifically investigate:

1. **API input deserialization**: Check [API_ENTRY_POINT] for all endpoints that accept request bodies:
   - How is structured data deserialized? ([SERIALIZATION_LIBRARIES])
   - Are there type discriminators that could enable polymorphic deserialization attacks?
   - Is there input size limiting on request bodies?
   - Are deserialized objects validated before use?

2. **Configuration import/loading validation**: Check endpoints or functions that accept external configuration:
   - What validation is performed on imported objects?
   - Could malicious values (server names, connection strings, file paths) be injected?
   - Are there limits on the number/size of imported objects?
   - Could an attacker import a configuration that causes the application to connect to a malicious server?

3. **Real-time message deserialization**: Check [HUB_OR_WEBSOCKET_CLASSES]:
   - What messages does the application accept from clients?
   - Are message payloads validated and size-limited?
   - Could oversized or malformed messages cause crashes or resource exhaustion?

4. **Configuration file deserialization**: Check [CONFIG_CLASSES]:
   - How are configuration files deserialized?
   - Are there any custom type converters that could be exploited?
   - What happens if the config file is malformed or contains unexpected types?
   - Could a corrupted config file crash the application on startup?

5. **General input validation**: Search for places where external input flows into the system without validation:
   - HTTP request parameters (query strings, route parameters, headers)
   - Real-time client messages
   - Configuration values used in security-sensitive operations
   - File paths, server names, or URLs from user settings

Search patterns:
- Keywords: [DESERIALIZATION_KEYWORDS], FromBody, ReadFromJsonAsync, Bind, ModelState, Validate, DataAnnotations, FluentValidation, MinLength, MaxLength, Range, Required, TypeNameHandling, JsonPolymorphic

Provide a detailed findings report with file paths, line numbers, and severity ratings (Critical/High/Medium/Low/Info). **Do NOT include actual credential values, API keys, tokens, or passwords in your output** — use `[REDACTED]` placeholders.
~~~

## Customization Guide

> **⚠️ Placeholder Safety:** Placeholder values are substituted directly into the prompt text. A crafted value could act as a prompt injection. Only use placeholder values you trust — do not accept them from untrusted sources.

| Placeholder | Example Values |
|-------------|---------------|
| `[APPLICATION_NAME]` | MyWebApp, SqlAgMonitor |
| `[REPO_PATH]` | Full path to the repository root |
| `[API_ENTRY_POINT]` | `Program.cs`, `Startup.cs`, `routes/*.ts`, `views.py` |
| `[SERIALIZATION_LIBRARIES]` | `System.Text.Json`, `Newtonsoft.Json`, `Jackson`, `pickle`, `Marshal` |
| `[HUB_OR_WEBSOCKET_CLASSES]` | `MonitorHub.cs`, `ChatHub.cs`, `WebSocketHandler.ts` |
| `[CONFIG_CLASSES]` | `JsonConfigurationService.cs`, `config.ts`, `settings.py` |
| `[DESERIALIZATION_KEYWORDS]` | `JsonSerializer.Deserialize, JsonConvert.DeserializeObject, JSON.parse, yaml.safe_load, pickle.loads` |

## What Good Looks Like

- Request body size limits enforced at middleware level
- Deserialized objects validated with schema validation or data annotations
- No polymorphic deserialization of untrusted input (no `TypeNameHandling.All`)
- Configuration imports validated against a schema with field-level constraints
- Malformed input returns clear error responses without leaking internals
- Server names and paths from external input validated against allowlists
