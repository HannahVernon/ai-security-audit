# Concurrency & Race Conditions

**Domain:** Thread safety, locking, TOCTOU, async patterns  
**Agent Type:** `explore`  
**Priority:** Medium

## Purpose

Audit concurrency patterns for race conditions that could lead to data corruption, security bypasses, or denial of service. Race conditions are notoriously hard to detect in testing but can be exploited reliably by attackers.

> **⚠️ Authorized Use Only:** Use this prompt only to audit codebases you own or have explicit authorization to assess.

## Prompt

~~~
You are conducting a security assessment of the [APPLICATION_NAME] repository at [REPO_PATH].

Your task: **Audit concurrency and race condition risks** across the codebase.

Specifically investigate:

1. **Lock usage**: Find all lock acquisition ([LOCK_FILES]). Check:
   - Are there TOCTOU (time-of-check-time-of-use) vulnerabilities? (checking a condition, then acting on it without holding the lock)
   - Could a reader see inconsistent state during write operations?
   - Is timeout handling on lock acquisition correct?
   - Could a failed Dispose/close/finally leave a lock held permanently?
   - Are locks acquired in a consistent order? (deadlock prevention)

2. **Shared mutable state flags**: Check [FLAG_LOCATIONS]:
   - Are shared boolean/integer flags marked as `volatile` or using `Interlocked`/atomic operations?
   - Could a flag be read as stale from a CPU cache on another thread?
   - Are flag transitions protected against concurrent modification?
   - Do exception paths correctly reset flags? (check finally blocks)

3. **Initialization/migration concurrency**: Check [INIT_FILES]:
   - What prevents two instances/threads from running initialization simultaneously?
   - Could partial initialization leave the system in an inconsistent state?
   - Is there a double-checked locking pattern, and is it correctly implemented?

4. **Static mutable state**: Search for static fields/properties that are written from multiple threads. Check thread safety for each.

5. **Async/await patterns**: Look for:
   - `async void` methods (exceptions cannot be caught by callers)
   - Fire-and-forget async calls without error handling
   - Missing `ConfigureAwait(false)` in library code
   - `.Result` or `.Wait()` that could deadlock on UI/synchronization context threads
   - `Task.Run` wrapping async methods unnecessarily

Search all source files for: [SEARCH_KEYWORDS]

Provide a detailed findings report with severity ratings. For each finding, describe the specific race scenario (Thread A does X, Thread B does Y, result is Z). **Do NOT include actual credential values, API keys, or tokens in your output** — use `[REDACTED]` placeholders.
~~~

## Customization Guide

> **⚠️ Placeholder Safety:** Placeholder values are substituted directly into the prompt text. A crafted value could act as a prompt injection. Only use placeholder values you trust — do not accept them from untrusted sources. Note: `[SEARCH_KEYWORDS]` values are used in regex patterns — escape metacharacters appropriately.

| Placeholder | Example Values |
|-------------|---------------|
| `[LOCK_FILES]`| `Database/DuckDbInitializer.cs`, `services/CacheManager.ts`, `utils/connection_pool.py` |
| `[FLAG_LOCATIONS]` | `ArchiveService.IsArchiving static bool`, `isProcessing flag in WorkerService` |
| `[INIT_FILES]` | `DuckDbInitializer.cs InitializeAsync`, `db/migrations/runner.ts`, `alembic/env.py` |
| `[SEARCH_KEYWORDS]` | `ReaderWriterLockSlim, lock\s*\(, volatile, Interlocked, synchronized, static.*=, async void, Task.Run, .Result, .Wait(), ConfigureAwait, mutex, semaphore, Monitor.Enter, atomic` |

## What Good Looks Like

- Shared flags use `volatile`, `Interlocked`, or `Atomic` operations
- Locks released in `finally` / `using` / `try-with-resources` blocks
- TOCTOU eliminated: check and act within same lock scope
- `async void` only in UI event handlers (never in library code)
- No `.Result` or `.Wait()` on code paths that could run on UI thread
- Lock acquisition order documented and consistent (prevents deadlocks)
- Initialization uses proper synchronization (lock, semaphore, or Lazy<T>)
