# Agent Tools and MCP Trust

**Domain:** Tool authorization, MCP server trust, tool-description poisoning, approval binding, confused deputy, egress scoping  
**Agent Type:** `explore`  
**Priority:** High

## Purpose

Audit applications where a model can invoke tools, including anything wired to Model Context Protocol servers.

The distinguishing risk is that the caller of a tool is a model, and the model's instructions can come from data.  A tool layer therefore has to answer a question a conventional API layer does not: not merely "is this request well formed", but "is the entity that decided to make this call authorised to, and does the human who approved it know what they approved".

Prompt 16 covers local IPC and privilege boundaries.  This prompt covers the trust relationship between an agent, its tool registry, remote servers, the user's identity, and the downstream services those tools reach.

> **⚠️ Authorized Use Only:** Use this prompt only to audit codebases you own or have explicit authorization to assess.

## Prompt

~~~
You are conducting a security assessment of the [APPLICATION_NAME] repository at [REPO_PATH].

Your task: **Audit the tool layer available to the model: what it can do, on whose authority, and what an untrusted tool result can cause.**

### Step 1: Inventory tools and servers

Enumerate and state explicitly:

- Every tool the model can invoke, from [TOOL_REGISTRY], including its parameters and what it ultimately touches
- Every MCP or external tool server configured, from [MCP_SERVERS], including transport, endpoint, and whether it is local or remote
- Which tools mutate state, spend money, send messages, or read data the calling user might not be entitled to

For each tool, record the effective privilege: the credentials or identity it acts under, which is frequently not the end user's.

### Step 2: Authorization on every invocation

1. **Caller identity**: determine whether each tool call is authorised against the *end user's* identity and entitlements, or against a service account shared by all users. A tool that reads records using a service credential, with the record id chosen by the model, is a confused deputy: the model was persuaded, the service account was authorised, and the user was neither.
2. **Per-resource checks**: for tools taking an id, path, tenant, or account, determine whether authorization is re-checked for that specific object at call time. Cross-reference prompt 10 (Authorization) and prompt 21 (Business Logic).
3. **Parameter validation**: determine whether tool arguments are validated deterministically before use, independent of the model having produced them.
4. **Enumeration**: determine whether a tool can be called repeatedly with varying ids to enumerate data, and whether rate limiting or scoping prevents that.

### Step 3: Untrusted tool results

Tool output re-enters model context, so it is untrusted input to the next turn.

1. Determine whether tool results are structurally separated from instructions when appended to context.
2. **Tool-description poisoning**: where tool names, descriptions, or schemas come from a remote server, determine whether that text reaches model context. A server that can change its own tool descriptions can attempt to redirect the agent, and the description is usually treated as trusted configuration rather than as remote data.
3. **Result-driven escalation**: determine whether a tool result can cause another tool to be invoked without a further authorization check, and whether a chain of tools can reach an effect that none of them individually would be permitted.
4. Cross-reference prompt 29 (LLM Application Trust Boundaries) for output sinks.

### Step 4: Approval binding

Where the design requires human approval before a sensitive action:

1. Determine whether approval is bound to the **exact arguments** that will execute, or to a summary of them. Approving "send an email" is not approving its recipient and body.
2. Determine whether arguments can change between approval and execution.
3. Determine whether an approval can be replayed, or whether one approval covers subsequent calls.
4. Determine what the user is actually shown: a rendered description produced by the model, or the literal call.
5. Determine whether approval state is stored somewhere the model or a tool can write to.

### Step 5: Server trust and lifecycle

1. **Onboarding**: determine how an MCP server is added, who can add one, and whether configuration is a file a lower-privileged process could write. Check [MCP_SERVERS] configuration paths and their permissions.
2. **Identity and transport**: for remote servers, determine whether TLS is verified, whether the endpoint is pinned, and whether the server authenticates itself.
3. **Token audience**: determine which credentials are forwarded to a server, whether the token is scoped to that server, and whether a malicious or compromised server could replay it elsewhere.
4. **Updates**: determine whether server code or tool definitions can change without review, and whether a version or digest is recorded. Cross-reference prompt 06 (Supply Chain).
5. **Local servers**: determine what user a local server runs as, and whether its socket, pipe, or port is reachable by other local users. Cross-reference prompt 16 (Local IPC and Privilege).

### Step 6: Blast containment

1. **Filesystem scope**: determine which paths tools can read and write, and whether traversal outside an intended root is prevented. Cross-reference prompt 03 (File and Process).
2. **Command execution**: determine whether any tool runs a shell, and how arguments are constructed.
3. **Network egress**: determine whether tools can reach arbitrary hosts, and whether internal ranges are blocked. Cross-reference prompt 20 (SSRF).
4. **Credential reach**: determine which secrets are available in the tool execution environment, and whether a single compromised tool exposes credentials for unrelated systems.
5. **Auditability**: determine whether tool invocations are logged with arguments, caller identity, and approval state, and whether that log is tamper-resistant. Cross-reference prompt 19 (Logging and Monitoring).

Search the codebase for:
- Tool, function, or capability registration and schema definitions
- MCP client and server configuration files, and their transports
- Approval, confirmation, or consent handling around tool execution
- Credential lookup inside tool implementations
- Subprocess, filesystem, and HTTP calls within tool bodies

Provide a findings report with file paths, line numbers, code snippets, and severity ratings (Critical/High/Medium/Low/Info), each with a confidence rating and the attacker prerequisites required.  For each finding state whether it requires the attacker to control model context, control a tool server, or already hold a local account.  **Do NOT include actual credential values, API keys, tokens, or passwords in your output** - use `[REDACTED]` placeholders.
~~~

## Customization Guide

> **⚠️ Placeholder Safety:** Placeholder values are substituted directly into the prompt text.  A crafted value could act as a prompt injection.  Only use placeholder values you trust - do not accept them from untrusted sources.

| Placeholder | Example Values |
|-------------|---------------|
| `[APPLICATION_NAME]` | ops-agent, DeployBot, support-copilot |
| `[REPO_PATH]` | Full path to the repository root |
| `[TOOL_REGISTRY]` | `src/tools/`, `ToolRegistry.cs`, the `@tool` decorated functions in `agent/` |
| `[MCP_SERVERS]` | `.mcp.json`, `mcp-servers.yaml`, `~/.config/agent/servers.json` |
| `[AUTHORIZATION_POLICY]` | `Authorization/PolicyProvider.cs`, the tenant filter in `data/repository.py` |
| `[APPROVAL_BOUNDARIES]` | `RequiresApproval` attribute, the confirmation modal in `web/ApprovalDialog.tsx` |

## What Good Looks Like

- Every tool invocation is authorised against the end user's identity, per resource, at call time
- No tool acts under a shared service credential with a model-chosen target, or where it must, the authorization is enforced separately and deterministically
- Tool results and remote tool descriptions are treated as untrusted input, not as configuration
- Approval is bound to the literal arguments that will execute, cannot be replayed, and shows the user the actual call
- MCP servers are pinned, authenticated, and version-recorded; configuration files are not writable by lower-privileged users
- Tokens forwarded to a server are scoped to that server's audience
- Filesystem, command, and network reach are scoped per tool rather than inherited from the agent process
- Tool invocations are logged with arguments, identity, and approval state, to storage the agent cannot rewrite
- An untrusted tool response cannot expand capability, grant permission, or authorise another call

## Relationship to Other Prompts

- **Prompt 29 (LLM Application Trust Boundaries)** covers the model call itself and its output sinks.  Use both for agentic applications.
- **Prompt 16 (Local IPC and Privilege)** covers local transport, named pipes, and privilege boundaries in depth; this prompt covers the trust relationship on top.
- **Prompt 06 (Supply Chain)** covers installing a tool server as a dependency.
- **Prompt 10 (Authorization)** and **prompt 21 (Business Logic)** cover the underlying access checks that tools must not bypass.
- **Prompt 20 (SSRF)** covers tools that fetch URLs.
