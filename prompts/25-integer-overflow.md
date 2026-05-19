# Integer Overflow, Numeric Truncation, and Precision Safety

**Domain:** Integer overflow/underflow, type truncation, floating-point precision, signed/unsigned confusion, size calculations  
**Agent Type:** `explore`  
**Priority:** Medium

## Purpose

Audit whether the application handles numeric values safely across parsing, arithmetic, allocation sizing, conversions, and precision-sensitive logic.  Numeric mistakes can cause incorrect allocations, buffer boundary errors, financial inaccuracies, logic bypasses, or denial-of-service conditions.  The exact failure mode varies by language: C and C++ may invoke undefined behavior, C# and Java often wrap silently unless checked arithmetic is used, and JavaScript uses floating-point numbers for all numeric values.

> **⚠️ Authorized Use Only:** Use this prompt only to audit codebases you own or have explicit authorization to assess.

## Prompt

~~~
You are conducting a security assessment of the [APPLICATION_NAME] repository at [REPO_PATH].

Your task: **Audit integer and numeric safety** across parsing paths, size calculations, conversions, precision-sensitive code, and arithmetic operations.  Search relevant source files matching [EXTENSIONS].

Specifically investigate:

1. **Integer overflow in size and length calculations**: Check [SIZE_CALC_CLASSES]:
   - Buffer size calculations using multiplication, such as `width * height * bytesPerPixel`, that could overflow before allocation
   - Array or collection size calculations derived from user input or external data
   - File size, stream length, or offset calculations, especially where 32-bit integers are used with large files
   - Checked versus unchecked arithmetic contexts, such as C# `checked` or `unchecked`, Java `Math.addExact`, `Math.multiplyExact`, and similar safe arithmetic helpers

2. **Type narrowing and truncation**: Check [SIZE_CALC_CLASSES] and [PARSER_CLASSES]:
   - `long` or `int64` values cast to `int` or `int32`
   - `double` or floating-point values converted to integer types, truncating the fractional portion
   - Implicit narrowing conversions in assignments, return values, or constructor arguments
   - Database identifiers, file lengths, counts, or external numeric values that may exceed application-side integer limits

3. **Signed and unsigned confusion**: Check [SIZE_CALC_CLASSES] and [PARSER_CLASSES]:
   - Negative values accepted where only positive sizes, counts, indices, or offsets are valid
   - Signed integers used for sizes or lengths and then compared or cast to unsigned types
   - User input parsed as a signed integer and later used as an unsigned offset, length, or array index
   - Boundary checks that validate upper bounds but forget to reject negative values

4. **Floating-point precision issues**: Check [FINANCIAL_CLASSES]:
   - Financial or monetary calculations using `float` or `double` instead of `decimal`, `BigDecimal`, or equivalent precise numeric types
   - Equality comparisons on floating-point values
   - Accumulated rounding error inside loops, aggregation logic, or percentage calculations
   - Currency conversion, tax, discount, or interest calculations that rely on imprecise numeric types

5. **Division and modulo safety**: Check [PARSER_CLASSES] and [SIZE_CALC_CLASSES]:
   - Division by zero, especially when the divisor is influenced by user input or external data
   - Integer division truncation where the truncation may be unintentional
   - Modulo operations with negative operands, where language semantics may surprise the developer
   - Percentage, ratio, or averaging calculations that can fail on empty or zero-valued inputs

6. **Parsing and conversion safety**: Check [PARSER_CLASSES]:
   - `int.Parse`, `Integer.parseInt`, `Convert.ToInt32`, `Convert.ToInt16`, and similar conversion APIs used without explicit range validation
   - User input parsed as numeric values without overflow, underflow, or sign checks
   - String-to-number conversions that could yield `NaN`, `Infinity`, `-Infinity`, or other unexpected special values
   - Culture-sensitive number parsing where decimal separators, thousands separators, or locale-specific formatting could change behavior

Search patterns:
- Keywords: `int.Parse`, `int.TryParse`, `Integer.parseInt`, `Convert.ToInt32`, `Convert.ToInt16`, `(int)`, `(short)`, `(byte)`, `unchecked`, `checked`, `Math.addExact`, `Math.multiplyExact`, `BigInteger`, `decimal`, `float`, `double`, `Math.Floor`, `Math.Ceiling`, `Math.Round`, `overflow`, `MaxValue`, `MinValue`, `Int32.MaxValue`, `Int16.MaxValue`, `sizeof`, `Length`, `Count`, `Size`, `capacity`, `offset`

Provide a detailed findings report with file paths, line numbers, and severity ratings (Critical/High/Medium/Low/Info).  For each finding, explain the numeric failure mode, the affected data path, the realistic security impact, and the safest remediation.  **Do NOT include actual credential values, API keys, tokens, or passwords in your output** - use `[REDACTED]` placeholders.
~~~

## Customization Guide

> **⚠️ Placeholder Safety:** Placeholder values are substituted directly into the prompt text.  A crafted value could act as a prompt injection.  Only use placeholder values you trust - do not accept them from untrusted sources.

Placeholder | Example Values
------------|---------------
`[APPLICATION_NAME]` | MyImageProcessor, BillingService, FileImportTool
`[REPO_PATH]` | Full path to the repository root
`[SIZE_CALC_CLASSES]` | `ImageDecoder.cs`, `BufferBuilder.cpp`, `ArchiveExtractor.java`, `ChunkAssembler.ts`
`[FINANCIAL_CLASSES]` | `InvoiceCalculator.cs`, `PricingService.java`, `TaxEngine.ts`, `LedgerTotals.py`
`[PARSER_CLASSES]` | `RequestModelBinder.cs`, `ImportParser.java`, `QueryStringParser.ts`, `CsvReader.py`
`[EXTENSIONS]` | `*.cs`, `*.cpp`, `*.h`, `*.java`, `*.ts`, `*.js`, `*.py`

## What Good Looks Like

- Checked arithmetic is used in security-sensitive calculations, such as C# `checked` contexts and Java `Math.xxxExact` helpers
- User-supplied numeric input is validated against acceptable ranges before use
- Financial calculations use `decimal`, `BigDecimal`, or equivalent precise numeric types, never `float` or `double`
- No implicit narrowing conversions exist in security-critical paths
- Division operations guard against zero divisors
- Size calculations are validated against maximum bounds before allocation
- Culture-invariant number parsing is used for machine-to-machine data
- Overflow is handled explicitly through checked blocks, exception handling, or range validation

## Relationship to Other Prompts

This prompt complements, not replaces, related prompts:

Prompt | Relationship
-------|-------------
11 - Denial of Service Resilience | Prompt 11 covers resource exhaustion broadly.  This prompt specifically audits the numeric correctness of calculations that, when wrong, can cause incorrect allocations, buffer boundary errors, financial inaccuracies, or logic bypasses.
15 - PowerShell Robustness & Defensive Scripting | Prompt 15 looks at PowerShell type safety and defensive scripting patterns broadly.  This prompt focuses specifically on overflow, truncation, signedness, precision, and parsing hazards across all languages.
