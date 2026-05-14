# Local IPC & Privilege Escalation

**Domain:** Stdio/pipe/socket server authentication, privilege boundaries, local escalation
**Agent Type:** `explore`
**Priority:** High

## Purpose

Audit local inter-process communication (IPC) servers - including MCP servers, language servers, debug adapters, and other stdio/pipe/socket-based tools - for privilege escalation and unauthorized access vulnerabilities. These servers typically run as child processes with the parent's privileges and perform sensitive operations (filesystem access, code execution, database queries) on behalf of the caller. Unlike web APIs, they often lack authentication entirely, relying on the assumption that only the parent process can reach them. On shared systems, misconfigured pipes, predictable socket paths, or missing caller verification can allow a lower-privileged process to hijack the server and escalate privileges.

> **Warning: Authorized Use Only:** Use this prompt only to audit codebases you own or have explicit authorization to assess.

## Prompt

~~~
You are conducting a security assessment of the [APPLICATION_NAME] repository at [REPO_PATH].

Your task: **Audit local IPC security and privilege escalation risks** across the entire codebase.

This application [IPC_DESCRIPTION].

Specifically investigate:

1. **Caller authentication**: How does the server verify who is sending requests?
   - Is there a shared secret, token, or HMAC exchanged at startup?
   - Does the server verify the caller's process identity (PID, user, parent process)?
   - Could any local process with access to the communication channel send requests?
   - [SPECIFIC_AUTH_LOCATIONS]

2. **Privilege boundary analysis**: Does the server grant more access than the caller already has?
   - What privileges does the server process run with? (user account, group memberships, capabilities)
   - Could a less-privileged process use the server to read, write, or delete files it could not access directly?
   - Does the server perform operations that require elevated privileges (e.g., accessing other users' files, system directories, network resources)?
   - Are allowed operations scoped to what the caller legitimately needs?

3. **Runtime configuration expansion**: Can callers modify the server's security boundaries at runtime?
   - Can new directories, resources, or capabilities be added via runtime commands?
   - Is there any authorization check before expanding access?
   - Could an attacker use configuration commands to grant themselves access to sensitive paths?
   - [SPECIFIC_CONFIG_LOCATIONS]

4. **Communication channel security**:
   - **Stdio**: Who can write to the server's stdin? Is the pipe inherited by child processes or accessible via /proc?
   - **Named pipes** (Windows): What DACL/security descriptor is set? Can other users connect? Is `PIPE_REJECT_REMOTE_CLIENTS` set?
   - **Unix domain sockets**: What filesystem permissions are set on the socket file? Is it in a user-private directory or a world-accessible location like `/tmp`?
   - **TCP loopback**: Is it bound to 127.0.0.1 only? Could other users on the same host connect?
   - [SPECIFIC_CHANNEL_LOCATIONS]

5. **Symlink and junction attacks on IPC endpoints**:
   - Could an attacker replace a named pipe, socket file, or working directory with a symlink before the server starts?
   - Are IPC endpoint paths created in predictable locations?
   - Does the server verify it created the endpoint (not an attacker)?

6. **Process identity and inheritance**:
   - Does the server inherit sensitive environment variables, file handles, or tokens from the parent?
   - Could a compromised parent process use inherited handles to bypass server-side checks?
   - Are child processes spawned by the server properly sandboxed?
   - Does the server drop privileges after initialization if it starts elevated?

7. **Request scoping and least privilege**:
   - Are all operations necessary for the server's purpose? Could any be removed to reduce attack surface?
   - Is there a read-only mode or capability restriction mechanism?
   - Can destructive operations (delete, overwrite, execute) be disabled via configuration?
   - Are operations logged with caller identity for audit trails?

Search patterns:
- IPC setup: stdin, stdout, Console.OpenStandardInput, Console.OpenStandardOutput, NamedPipeServerStream, NamedPipeClientStream, Socket, UnixDomainSocketEndPoint, TcpListener, IPAddress.Loopback, PipeOptions, PipeSecurity, FileSystemWatcher
- Auth: token, secret, apikey, Bearer, Authorization, HMAC, handshake, authenticate, verify, identity, credential, PipeAccessRule, SocketPermission
- Privilege: Process.Start, Environment.UserName, WindowsIdentity, getuid, geteuid, setuid, capabilities, RunAs, impersonate, AllowedDirectories, IsPathAllowed
- Config expansion: AddAllowed, Configure, Register, Enable, SetPermission, grant, expand

Provide a detailed findings report with file paths, line numbers, code snippets, and severity ratings (Critical/High/Medium/Low/Info). For each finding, describe a specific local escalation scenario. **Do NOT include actual credential values, API keys, tokens, or passwords in your output** - use `[REDACTED]` placeholders.
~~~

## Customization Guide

> **Warning: Placeholder Safety:** Placeholder values are substituted directly into the prompt text. A crafted value could act as a prompt injection. Only use placeholder values you trust - do not accept them from untrusted sources.

Placeholder | Example Values
------------|---------------
`[APPLICATION_NAME]` | FileSystemMcpServer, vscode-languageserver, debug-adapter
`[REPO_PATH]` | Full path to the repository root
`[IPC_DESCRIPTION]` | `is a .NET MCP server that exposes filesystem read/write/delete operations over JSON-RPC via stdio`, `is a Node.js language server communicating via named pipes`, `is a Python debug adapter using Unix domain sockets`
`[SPECIFIC_AUTH_LOCATIONS]` | `Check Program.cs RunMcpLoop for any authentication handshake before accepting requests`, `Check server.ts initialize handler for token validation`
`[SPECIFIC_CONFIG_LOCATIONS]` | `Check HandleConfigureDirectories in Program.cs - allows runtime directory expansion via configureDirectories method`, `Check workspace/didChangeConfiguration handler`
`[SPECIFIC_CHANNEL_LOCATIONS]` | `Check Program.cs lines 109-113 where stdin/stdout are opened`, `Check createServer() in transport.ts for pipe security settings`

### Stack-specific notes

**MCP servers (.NET, Node.js, Python):**
- Typically communicate via stdio (stdin/stdout)
- Run as child processes of LM Studio, Ollama, Claude Desktop, VS Code, etc.
- Often perform filesystem, database, or network operations on behalf of the AI client
- The parent process is the trust boundary, but other local processes may access the pipe

**Language servers (LSP):**
- Communicate via stdio, named pipes, or TCP
- Often have access to the entire workspace filesystem
- May execute build commands, linters, or formatters

**Debug adapters (DAP):**
- Communicate via stdio or TCP
- Have process control capabilities (attach, execute, memory read/write)
- Highest privilege escalation risk if unauthenticated

## What Good Looks Like

- Startup handshake with a shared secret passed via environment variable (not command line, which is visible in process listings)
- Server verifies caller PID/user matches expected parent process
- Named pipes created with restrictive DACLs (current user only)
- Unix sockets created in user-private directories with 0700 permissions
- TCP listeners bound to 127.0.0.1 with a random port communicated to the parent
- Runtime configuration expansion requires re-authentication or is disabled entirely
- Destructive operations (delete, overwrite) can be disabled via a read-only mode flag
- Server drops unnecessary privileges after initialization
- All operations logged with enough context for security audit
- Symlink checks on IPC endpoint paths before binding
- Documentation clearly states the trust model and expected deployment environment
