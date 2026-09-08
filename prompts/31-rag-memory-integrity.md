# RAG, Embeddings and AI Memory Integrity

**Domain:** Ingestion poisoning, retrieval authorization, cross-tenant leakage, provenance, persistent memory, deletion propagation  
**Agent Type:** `explore`  
**Priority:** High

## Purpose

Audit retrieval-augmented generation and any persistent agent memory.

Retrieval moves an authorization decision into a similarity search.  A vector store that returns the nearest chunks does not know who is asking, and an index built from documents with different permissions has already flattened them unless something outside the index re-applies the check.

Persistent memory adds a second problem: content written during one interaction influences later ones, potentially for a different user.  That makes memory a stored-injection surface with a long fuse.

Deletion is the third.  A document removed from its source system may survive in chunks, embeddings, caches, summaries and conversation history, which matters for both correctness and any deletion obligation.

> **⚠️ Authorized Use Only:** Use this prompt only to audit codebases you own or have explicit authorization to assess.

## Prompt

~~~
You are conducting a security assessment of the [APPLICATION_NAME] repository at [REPO_PATH].

Your task: **Audit whether retrieved content should have been retrievable, whether it can be poisoned, and whether removing it actually removes it.**

### Step 1: Map the pipeline

State explicitly:

- **Ingestion**: every path by which content enters the index. Check [INGESTION_PIPELINES]. Include uploads, connectors, crawlers, scheduled syncs, and user-generated content.
- **Stores**: every vector store, index, cache, or memory store. Check [VECTOR_STORES].
- **Source permissions**: how the origin systems express access control. Check [SOURCE_ACLS].
- **Memory writers**: everything that can write persistent memory. Check [MEMORY_WRITERS].

Note for each store whether it holds a tenant or permission attribute at all. If it does not, retrieval cannot be filtered on one.

### Step 2: Who can put content in

1. **Ingestion authorization**: determine who can add or modify indexed content, and whether that is narrower than "any authenticated user".
2. **Poisoning reach**: determine whether content one user contributes can be retrieved for another. If so, an uploaded document is an instruction delivery mechanism into someone else's model context. Cross-reference prompt 29 (LLM Application Trust Boundaries).
3. **Automated ingestion**: for crawlers and connectors, determine what bounds the source. A crawler following links can index a page written specifically to be indexed.
4. **Content handling**: determine whether ingested content is treated as data throughout, or whether any stage interprets it. Check for HTML, Markdown, or script content preserved into chunks.
5. **Volume and cost**: determine whether ingestion is bounded per user, and whether an oversized document can exhaust storage or embedding budget. Cross-reference prompt 11 (Denial of Service).

### Step 3: Retrieval authorization

1. **Enforcement point**: determine whether retrieval filters by the requesting user's entitlements, and whether that filter is applied *in* the query rather than by discarding results afterwards. Post-filtering still ranks and reads documents the user cannot see, and leaks through result counts, latency, and any summary computed before the filter.
2. **Permission freshness**: determine what happens when access is revoked at the source. If permissions were copied into the index at ingestion, determine how they are updated and how stale they can be.
3. **Tenant isolation**: for multi-tenant systems, determine whether tenancy is a filter on a shared index or a hard partition, and whether a missing filter fails closed or returns everything.
4. **Shared caches**: determine whether embedding caches, query caches, or summary caches are keyed by user or tenant. A cache keyed only by query text will serve one user's retrieved content to another.
5. **Chunk provenance**: determine whether each chunk carries its source document, permissions, and version, and whether that survives into the model context and any citation shown to the user.

### Step 4: Persistent memory

1. **Write authorization**: determine what can write memory, whether a model can write it directly, and whether written content is validated.
2. **Scope**: determine whether a memory is scoped to a user, a session, a tenant, or is global. A globally scoped memory writable from a single user's conversation is a cross-user injection channel.
3. **Attribution**: determine whether memory entries record who or what created them, and when.
4. **Review and expiry**: determine whether memories expire, whether the user can inspect and delete them, and whether an operator can audit them.
5. **Poisoned recall**: determine whether a memory written during a compromised interaction can steer later ones, and whether anything would detect that.

### Step 5: Deletion and revocation

1. Trace a document deletion through: source system, ingested copy, chunks, embeddings, index entries, caches, summaries, derived memories, and conversation history.
2. Determine whether embeddings are treated as derived data that must also be removed. An embedding is not the source text but is derived from it, and approximate reconstruction from embeddings is an active research area, so treating it as anonymous is not safe.
3. Determine whether deletion is verified or assumed, and whether a failure is detected.
4. Cross-reference prompt 27 (Privacy and PII) for obligations attaching to that data.

### Step 6: Evidence

Determine whether tests exist that demonstrate the boundaries hold: a cross-tenant retrieval attempt returning nothing, a revoked document disappearing from results, a poisoned document failing to change behaviour for another user, a deletion propagating to embeddings.

Search the codebase for:
- Vector store clients, index creation, and similarity query construction
- Chunking and embedding code, and what metadata it attaches
- Filter, namespace, tenant, or partition arguments on retrieval calls, and any retrieval call lacking one
- Cache key construction around embeddings, queries, and summaries
- Memory read and write APIs, and their scope parameters
- Deletion and revocation handlers

Provide a findings report with file paths, line numbers, code snippets, and severity ratings (Critical/High/Medium/Low/Info), each with a confidence rating and the attacker prerequisites required.  Where a control appears absent, state whether it is genuinely missing or merely enforced somewhere you could not see.  **Do NOT include actual credential values, API keys, tokens, or passwords in your output** - use `[REDACTED]` placeholders.
~~~

## Customization Guide

> **⚠️ Placeholder Safety:** Placeholder values are substituted directly into the prompt text.  A crafted value could act as a prompt injection.  Only use placeholder values you trust - do not accept them from untrusted sources.

| Placeholder | Example Values |
|-------------|---------------|
| `[APPLICATION_NAME]` | knowledge-assistant, DocSearch, support-copilot |
| `[REPO_PATH]` | Full path to the repository root |
| `[INGESTION_PIPELINES]` | `ingest/`, `SharePointConnector.cs`, `crawler/spider.py`, the upload handler in `api/documents` |
| `[VECTOR_STORES]` | Pinecone index `docs-prod`, pgvector table `embeddings`, Azure AI Search index, an in-process FAISS index |
| `[SOURCE_ACLS]` | SharePoint permissions, `documents.tenant_id`, Confluence space restrictions |
| `[MEMORY_WRITERS]` | `MemoryService.Write`, the summariser in `agent/reflect.py`, any tool that persists a note |

## What Good Looks Like

- Retrieval filters by the requesting user's entitlements inside the query, not by discarding results afterwards
- Tenancy is a hard partition, or a filter that fails closed when absent
- Every chunk carries source, permission, and version metadata, and that provenance reaches the citation shown to the user
- Permission changes at the source propagate to the index on a known and bounded schedule
- Embedding, query, and summary caches are keyed by user or tenant
- Ingestion is authorised, bounded in size and rate, and treats content as data at every stage
- Memory writes are scoped, attributed, expiring, and inspectable by the user they belong to
- Deletion propagates to chunks, embeddings, indexes, caches, summaries and history, and is verified rather than assumed
- Tests cover cross-tenant retrieval, revocation, poisoned documents, and deletion propagation

## Relationship to Other Prompts

- **Prompt 29 (LLM Application Trust Boundaries)** assumes retrieval already happened and asks what the retrieved content can do.  This prompt asks whether it should have been retrieved.
- **Prompt 30 (Agent Tools and MCP Trust)** covers tools that perform retrieval as one capability among many.
- **Prompt 10 (Authorization)** covers the underlying access model that retrieval must not bypass.
- **Prompt 27 (Privacy and PII)** covers the obligations attaching to indexed personal data, including deletion.
- **Prompt 11 (Denial of Service)** covers ingestion and embedding cost abuse.
