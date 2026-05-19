# Regular Expression Safety (ReDoS)

**Domain:** Catastrophic backtracking, regex denial of service, untrusted regex patterns, timeout protection  
**Agent Type:** `explore`  
**Priority:** Medium

## Purpose

Audit the codebase for regular expression denial of service risks, especially catastrophic backtracking patterns that can be triggered by a single crafted input string.  ReDoS is CWE-1333 and is often missed because the vulnerable code can look like ordinary validation or parsing logic.

> **⚠️ Authorized Use Only:** Use this prompt only to audit codebases you own or have explicit authorization to assess.

## Prompt

~~~
You are conducting a security assessment of the [APPLICATION_NAME] repository at [REPO_PATH].

Your task: **Audit regular expression usage for regex denial of service (ReDoS) risks** across the codebase.

Specifically investigate:

1. **Catastrophic backtracking patterns**: Find all regex patterns and check for vulnerable constructs.  Review [VALIDATION_CLASSES] and [EXTENSIONS].
   - Nested quantifiers: `(a+)+`, `(a*)*`, `(a?)*`, `(a|b)*`
   - Overlapping alternations: `(a|a)+`, `(\d+|\d+\.)+`
   - Quantifiers on groups with alternations that share a common prefix
   - Common vulnerable patterns: email validation regexes, URL parsing, IP address matching, HTML tag parsing
   - Determine whether the pattern can enter catastrophic backtracking on near-match or mismatch input

2. **User-supplied regex**: Determine whether the application accepts regex patterns from users.  Check [SEARCH_CLASSES] and [EXTENSIONS].
   - Search or filter features that accept regex input
   - Configuration fields that accept regex patterns
   - Whether user-supplied patterns are compiled and executed without timeout or complexity limits
   - Whether a malicious regex could consume CPU for minutes or hours

3. **Regex timeout or cancellation**: Check whether regex execution is time-bounded.
   - In .NET: is `Regex` constructed with a `MatchTimeout`?  The default is effectively infinite for security purposes.
   - In Java: are `Pattern.matcher` operations wrapped with timeout or cancellation logic?
   - In Python: is the `re` module used without timeout protection?  Python 3.11+ adds `re.TIMEOUT` support.
   - In Node.js: are regex operations isolated in a worker, guarded by input limits, or otherwise protected?
   - Is there a global regex timeout policy or shared helper?

4. **Input length limits**: Before regex matching, determine whether attacker-controlled input is bounded.
   - Is the input string length-bounded before matching?
   - Could an attacker submit a very long string to amplify backtracking cost?
   - Are there different length limits for different regex-validated fields?

5. **Regex compilation caching**: Check whether regex compilation itself can become a performance problem.
   - Are regex patterns compiled once and reused, or recompiled on every call?
   - Could repeated compilation of complex patterns cause performance issues?
   - In .NET: are `RegexOptions.Compiled` patterns created in hot paths without caching?

Search keywords: `Regex`, `RegExp`, `new Regex`, `Regex.Match`, `Regex.Replace`, `Regex.IsMatch`, `Pattern.compile`, `re.compile`, `re.match`, `re.search`, `re.findall`, `.test(`, `.match(`, `.replace(`, `.exec(`, `MatchTimeout`, `RegexMatchTimeoutException`, `RegexOptions`, `PCRE`, `backtrack`.

Provide a detailed findings report with file paths, line numbers, code snippets, and severity ratings.  Explain which pattern is vulnerable, what crafted input shape would trigger pathological backtracking, whether user-controlled input can reach it, and what timeout or length controls exist.  Do NOT include actual credential values, API keys, tokens, passwords, or connection strings in your output - use `[REDACTED]` placeholders.
~~~

## Customization Guide

> **⚠️ Placeholder Safety:** Placeholder values are substituted directly into the prompt text.  A crafted value could act as a prompt injection.  Only use placeholder values you trust.  Do not accept placeholder values from untrusted sources.

Placeholder | Example Values
------------|---------------
`[APPLICATION_NAME]` | `MyWebApp`, `SearchPortal`, `ValidationService`
`[REPO_PATH]` | Full path to the repository root
`[VALIDATION_CLASSES]` | `EmailValidator.cs`, `InputSanitizer.ts`, `validators.py`, `RegexHelpers.java`
`[SEARCH_CLASSES]` | `SearchController.cs`, `FilterService.ts`, `AdvancedSearch.py`, `QueryBuilder.java`
`[EXTENSIONS]` | `Inspect config-driven validation rules`, `Review tenant-defined filters`, `Search for regex use in tests that mirrors production code`

## What Good Looks Like

- All regex patterns reviewed for catastrophic backtracking potential
- Regex operations have explicit timeouts, `MatchTimeout` in .NET and custom timeout wrappers elsewhere
- User-supplied regex patterns rejected or sandboxed, with no arbitrary regex accepted from untrusted input
- Input strings length-bounded before regex matching
- Regex patterns compiled once and cached, such as static `RegexOptions.Compiled` fields or pre-compiled `Pattern` objects
- Atomic groups or possessive quantifiers used where appropriate to prevent backtracking
- Complex validation handled with parser logic instead of regex where practical
- Known-vulnerable email and URL regex patterns replaced with well-tested libraries

## Relationship to Other Prompts

Prompt | Relationship
-------|-------------
11 - Denial of Service Resilience | Prompt 11 covers denial of service broadly, including rate limiting, resource exhaustion, and backpressure.  This prompt specifically audits regex patterns as a denial of service vector, which is a distinct and often overlooked attack surface.
