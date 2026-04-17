# Binary Protocol Parsing & Untrusted Network Input

**Domain:** Network protocol parsing, binary input validation, buffer safety  
**Agent Type:** `explore`  
**Priority:** High

## Purpose

Audit how the application parses binary data received from remote servers or untrusted network peers. Applications that implement custom protocol parsers (TDS, HTTP/2, MQTT, gRPC, Protobuf, etc.) are vulnerable to crafted responses that exploit length field handling, offset calculations, or buffer boundary assumptions. A malicious server — or a man-in-the-middle attacker — can send packets designed to crash the client, corrupt memory, or cause denial of service.

> **⚠️ Authorized Use Only:** Use this prompt only to audit codebases you own or have explicit authorization to assess.

## Prompt

~~~
You are conducting a security assessment of the [APPLICATION_NAME] repository at [REPO_PATH].

Your task: **Audit binary protocol parsing and untrusted network input handling** across the entire codebase.

This application [PROTOCOL_DESCRIPTION].

Specifically investigate:

1. **Length field handling**: Find all locations where a length or size value is read from network data. Check:
   - Are length values validated against buffer bounds before use?
   - Could an oversized length value cause `OutOfMemoryException` or allocation of extremely large buffers?
   - Could a zero or negative length cause underflows, infinite loops, or division by zero?
   - Are length values used in array indexing without bounds checks?
   - [SPECIFIC_LENGTH_FIELD_LOCATIONS]

2. **Offset and pointer arithmetic**: Find all calculated offsets into received data. Check:
   - Are offsets validated to fall within the received buffer?
   - Could overlapping or out-of-order offsets cause data corruption?
   - Are offset+length combinations checked for integer overflow?
   - [SPECIFIC_OFFSET_LOCATIONS]

3. **Packet completeness**: Check all packet/message reading code:
   - What happens if the connection closes mid-packet?
   - What happens if fewer bytes are received than expected?
   - Are partial reads handled correctly (TCP stream reassembly)?
   - Is there a maximum packet size enforced to prevent memory exhaustion?
   - [SPECIFIC_PACKET_READ_LOCATIONS]

4. **Type confusion and magic values**: Check protocol type/version fields:
   - Are unexpected type codes handled gracefully (not just ignored)?
   - Could a type code outside the expected range cause array index-out-of-bounds?
   - Are protocol version mismatches detected and reported?

5. **String extraction from binary data**: Check all locations where strings are read from network data:
   - Are string lengths validated before extraction?
   - Are null terminators required and verified?
   - Could a missing null terminator cause reading past buffer bounds?
   - Is the character encoding validated (UTF-8, ASCII, UTF-16)?
   - [SPECIFIC_STRING_EXTRACTION_LOCATIONS]

6. **Response validation**: Does the application validate that responses correspond to requests?
   - Could a server send an unexpected response type that the parser handles incorrectly?
   - Are sequence numbers or correlation IDs validated?
   - Could a server send responses out of order to confuse the parser?

Search patterns:
- Buffer operations: `Buffer.BlockCopy, Array.Copy, Span<byte>, ReadOnlySpan, MemoryMarshal, BinaryReader, BitConverter, Marshal.Copy`
- Length reads: `ReadInt16, ReadInt32, ReadUInt16, ReadUInt32, [offset], payload.Length, buffer.Length`
- Network reads: `Read, ReadAsync, ReadByte, ReadBytes, Receive, ReceiveAsync, NetworkStream, SslStream`
- Allocation: `new byte[], ArrayPool, stackalloc, Marshal.AllocHGlobal`

Provide a detailed findings report with file paths, line numbers, code snippets, and severity ratings (Critical/High/Medium/Low/Info). For each finding, describe a specific attack scenario a malicious server could use to exploit it. **Do NOT include actual credential values, API keys, or tokens in your output** — use `[REDACTED]` placeholders.
~~~

## Customization Guide

> **⚠️ Placeholder Safety:** Placeholder values are substituted directly into the prompt text. A crafted value could act as a prompt injection. Only use placeholder values you trust — do not accept them from untrusted sources.

| Placeholder | Example Values |
|-------------|---------------|
| `[APPLICATION_NAME]` | sql-cert-inspector, MyMqttClient, CustomHttpParser |
| `[REPO_PATH]` | Full path to the repository root |
| `[PROTOCOL_DESCRIPTION]` | `implements a TDS PRELOGIN client that connects to SQL Server and parses binary TDS responses`, `parses MQTT packets from a broker`, `implements a custom binary RPC protocol` |
| `[SPECIFIC_LENGTH_FIELD_LOCATIONS]` | `Check TDS packet header length at bytes 2-3 in TdsPacket.cs`, `Check MQTT remaining length decoding` |
| `[SPECIFIC_OFFSET_LOCATIONS]` | `Check PRELOGIN option offset/length pairs in TdsPreloginClient.cs`, `Check protobuf field tag parsing` |
| `[SPECIFIC_PACKET_READ_LOCATIONS]` | `Check ReadAsync loop in TdsPreloginStream.cs`, `Check WebSocket frame reader` |
| `[SPECIFIC_STRING_EXTRACTION_LOCATIONS]` | `Check SQL Browser response parsing in SqlBrowserClient.cs`, `Check MQTT topic string extraction` |

## What Good Looks Like

- All length values validated against actual buffer size before use
- Maximum allocation limits enforced (e.g., max packet size of 64KB)
- Integer overflow checks on offset+length calculations
- Partial reads handled correctly with buffering loops
- Unexpected type codes produce clear error messages (not crashes)
- Connection timeouts prevent indefinite hangs on slow/malicious servers
- Strings extracted with explicit length bounds, not relying on null terminators alone
- Memory allocated from received lengths is clamped to reasonable maximums
