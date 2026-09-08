# LLM Application Trust Boundaries and Output Handling

**Domain:** Prompt injection, untrusted context, model output as a sink, secrets in context, provider retention, cost abuse  
**Agent Type:** `explore`  
**Priority:** High

## Purpose

Audit applications that call a language model.  Every other prompt in this collection assumes a conventional application; this one assumes the application has a model in the middle of it, which moves the trust boundary somewhere the other prompts do not look.

Two distinct risks live here.  Untrusted text reaching model context can change what the model is asked to do, and model output reaching a sink can change what the application does.  Neither is fixed by a better system prompt, so this prompt looks for deterministic controls at the boundaries instead.

This is not about auditing *with* an AI agent.  It is about auditing an application that *contains* one.

> **⚠️ Authorized Use Only:** Use this prompt only to audit codebases you own or have explicit authorization to assess.

## Prompt

~~~
You are conducting a security assessment of the [APPLICATION_NAME] repository at [REPO_PATH].

Your task: **Audit every path where untrusted data reaches a language model, and every path where model output reaches something that acts on it.**

### Step 1: Inventory the boundary

Before assessing anything, build three lists and state them explicitly:

- **Call sites**: every place the application invokes a model.  Check [MODEL_CALL_SITES].  Look for SDK clients, HTTP calls to model endpoints, framework abstractions, and streaming handlers.
- **Context sources**: everything that can end up in a prompt.  Check [UNTRUSTED_INPUT_SOURCES].  Include user messages, retrieved documents, file uploads, tool and function results, web page content, database fields, email bodies, filenames, and error text.
- **Output consumers**: everything that receives model output.  Check [OUTPUT_CONSUMERS].  Include HTML rendering, SQL, shell or subprocess calls, file paths, HTTP requests, generated code, and any branch that changes application behaviour.

An audit that cannot name these three lists has not established the boundary and should say so rather than guess.

### Step 2: Untrusted data reaching context

1. **Indirect injection**: For each context source, determine whether its content originates outside the trust boundary. A retrieved document, a scraped page, a PDF, a code comment, or a tool result can all carry text addressed to the model. Determine whether such content is distinguishable from operator instructions once assembled into a prompt.
2. **Structural separation**: Check whether untrusted content is delimited, labelled, or placed in a separate role, and whether that separation survives concatenation. String interpolation of untrusted text directly into an instruction template is the common failure.
3. **Privilege of the calling context**: Establish what the model's output is authorised to cause. A model that can only produce text for display is a different risk from one whose output selects a tool, a customer id, or a file path.
4. **Multi-turn and persistence**: Determine whether injected content can persist into later turns, other sessions, or other users through conversation history, summaries, caches, or stored memories.

### Step 3: Model output reaching a sink

For every output consumer identified in step 1, determine what validates the output before it acts:

- **HTML**: is output escaped, or rendered as markup? Look for `dangerouslySetInnerHTML`, `v-html`, `innerHTML`, and template filters marked safe or raw.
- **SQL**: is generated SQL parameterised, or executed as text? A model producing a `WHERE` clause is building a query.
- **Shell and subprocess**: is output passed as an argument array, or interpolated into a command string?
- **File paths**: is a model-chosen filename or path constrained to an intended directory?
- **URLs**: is a model-supplied URL fetched? Cross-reference prompt 20 (SSRF).
- **Authorization**: does any access decision depend on model output? Determine whether the check is re-performed deterministically against the caller's real identity, or whether the model is trusted to have applied it.
- **Structured output**: where the model returns JSON or a schema, determine whether it is validated against that schema, and what happens when validation fails.

### Step 4: What leaves with the request

1. **Secrets in context**: determine whether credentials, API keys, connection strings, tokens, or internal hostnames can reach a prompt through system instructions, retrieved documents, environment dumps, stack traces, or error text included for debugging.
2. **Personal data**: determine what personal data is transmitted, and whether that is compatible with the provider's terms and the application's stated handling. Cross-reference prompt 27 (Privacy and PII).
3. **Provider retention and training**: check configuration for retention settings, zero-retention or enterprise endpoints, logging of prompts and completions, and whether requests are opted out of training where that matters.
4. **Self-hosted or third-party endpoints**: check whether the endpoint is configurable, and whether a changed endpoint would send prompts somewhere unintended. A base URL read from configuration is the same class of problem as prompt 20's SSRF.

### Step 5: Availability and cost

1. **Token and request limits**: determine whether input length, output length, and request rate are bounded per user, and what happens when a limit is reached.
2. **Unbounded input**: check whether an uploaded file or a retrieved document is truncated before being sent, and whether cost scales with attacker-controlled size.
3. **Loops**: check whether the application can be driven into a self-sustaining chain of model calls, particularly where a model decides whether to continue.
4. **Failure handling**: determine what happens on provider error, timeout, refusal, or truncated output. Cross-reference prompt 11 (Denial of Service).

### Step 6: Testing and evidence

Determine whether the repository contains tests that demonstrate containment rather than asserting it: an injected instruction that fails to change behaviour, output that fails schema validation being rejected, a sink refusing malformed model output.

Search the codebase for:
- Model SDK imports and client construction
- Prompt template files and system-prompt string literals
- String concatenation or interpolation building prompts
- `dangerouslySetInnerHTML`, `innerHTML`, `v-html`, raw template filters near model output
- Subprocess, `eval`, and dynamic query construction near model output
- Configuration keys naming a model endpoint, key, retention, or logging setting

Provide a findings report with file paths, line numbers, code snippets, and severity ratings (Critical/High/Medium/Low/Info), each with a confidence rating and the attacker prerequisites required. **Do NOT include actual credential values, API keys, tokens, or passwords in your output** - use `[REDACTED]` placeholders.  State explicitly which of the three inventories in Step 1 you could not complete.
~~~

## Customization Guide

> **⚠️ Placeholder Safety:** Placeholder values are substituted directly into the prompt text.  A crafted value could act as a prompt injection.  Only use placeholder values you trust - do not accept them from untrusted sources.

| Placeholder | Example Values |
|-------------|---------------|
| `[APPLICATION_NAME]` | support-assistant, DocSearch, PRReviewBot |
| `[REPO_PATH]` | Full path to the repository root |
| `[MODEL_CALL_SITES]` | `src/ai/OpenAiClient.cs`, `services/llm.py`, `lib/anthropic.ts`, anything importing an SDK |
| `[UNTRUSTED_INPUT_SOURCES]` | `chat messages`, `uploaded PDFs in ingest/`, `web page text from the crawler`, `Jira ticket bodies`, `tool results` |
| `[OUTPUT_CONSUMERS]` | `React renderer in web/`, `the query builder in reports/`, `the shell wrapper in tasks/`, `the tool dispatcher` |
| `[PROVIDER_CONFIGURATION]` | `appsettings.json` AI section, `OPENAI_BASE_URL`, retention and logging flags |

## What Good Looks Like

- The three inventories are complete: every model call site, every context source, every output consumer
- Untrusted content is structurally separated from instructions, and that separation survives prompt assembly
- Authorization is enforced deterministically outside the model, against the caller's real identity, not by asking the model to respect it
- Every sink validates model output for its own context: escaping for HTML, parameters for SQL, argument arrays for subprocesses, containment checks for paths
- Structured output is schema-validated, and validation failure has a defined non-fatal path
- No credentials, connection strings, or internal hostnames can reach a prompt
- Retention, logging, and training settings are explicit rather than defaults
- Input size, output size, and request rate are bounded per user
- Tests demonstrate that an injected instruction does not change behaviour, rather than a system prompt asserting that it will not

## Relationship to Other Prompts

- **Prompt 06 (Supply Chain)** covers AI agent *configuration* poisoning as an installed-dependency problem.  This prompt covers the running application.
- **Prompt 30 (Agent Tools and MCP Trust)** covers what happens when the model can invoke tools.  Use both where the application is agentic.
- **Prompt 31 (RAG and AI Memory Integrity)** covers whether retrieved content should have been retrievable at all.  This prompt assumes retrieval happened and asks what the content can do.
- **Prompt 20 (SSRF)** covers fetching model-supplied URLs.
- **Prompt 22 (XSS)** covers rendering model output as HTML.
