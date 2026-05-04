# PowerShell Script Robustness & Strict Mode Safety

**Domain:** PowerShell strict mode compatibility, type safety, defensive scripting patterns  
**Agent Type:** `explore`  
**Priority:** Medium

## Purpose

Audit PowerShell scripts (`.ps1`, `.psm1`) for defects that surface under `Set-StrictMode -Version Latest`. These are not hypothetical — they cause silent failures or crashes in production when scripts run under strict mode or when data shapes vary from the developer's test cases. Many of these issues are invisible during casual testing because they only trigger on specific data patterns (single results, null JSON properties, integer values in arrays).

> **⚠️ Authorized Use Only:** Use this prompt only to audit codebases you own or have explicit authorization to assess.

## Prompt

~~~
You are conducting a robustness assessment of PowerShell scripts in the [APPLICATION_NAME] repository at [REPO_PATH].

Your task: **Audit all PowerShell scripts for strict mode violations, type safety issues, and defensive scripting defects.**

### Step 1: AST-Based Function Export

Before reviewing code, extract the script's structure using the PowerShell AST parser:

```powershell
$ast = [System.Management.Automation.Language.Parser]::ParseFile(
    "[SCRIPT_PATH]", [ref]$null, [ref]$null)
$functions = $ast.FindAll(
    { $args[0] -is [System.Management.Automation.Language.FunctionDefinitionAst] }, $true)
foreach ($fn in $functions) {
    $params = $fn.Body.ParamBlock
    $paramList = if ($params) {
        ($params.Parameters | ForEach-Object { $_.Name.VariablePath.UserPath }) -join ', '
    } else { '(none)' }
    "L$($fn.Extent.StartLineNumber): function $($fn.Name) ($paramList)"
}
```

Use this function map to understand the script's architecture before diving into individual issues.

### Step 2: Investigate These Categories

1. **Strict mode property access**: Under `Set-StrictMode -Version Latest`, accessing a property that does not exist on an object throws a terminating error. Check:
   - `ConvertFrom-Json` results: JSON properties accessed directly (e.g., `$json.propertyName`) without guarding via `$obj.PSObject.Properties.Name -contains 'propertyName'`
   - This is especially dangerous when the JSON serializer uses `WhenWritingNull` or `WhenWritingDefault`, which omits null/default properties entirely from the output
   - Objects from `Where-Object` pipelines: single results are returned as scalars, not arrays — `.Count` fails on scalars under strict mode
   - [SPECIFIC_JSON_LOCATIONS]

2. **Array flattening traps**: PowerShell's `@()` operator flattens nested arrays when inner arrays are not comma-separated. Check:
   - Array-of-arrays construction: `@( @('a', 1) @('b', 2) )` creates a FLAT array `('a', 1, 'b', 2)` — the inner arrays are pipeline items that get unrolled
   - Fix requires the unary comma operator: `@( ,@('a', 1) ,@('b', 2) )` preserves each inner array as a single element
   - This causes "Unable to index into an object of type System.Int64" when the flattened array contains integers and code tries `$item[0]`
   - Look for patterns where arrays of label/value pairs are built for iteration

3. **`.Count` on non-array values**: Under strict mode, `.Count` is not available on scalars or single objects. Check:
   - `Where-Object` pipeline results used with `.Count` — single matches return a scalar, not a 1-element array
   - Fix: wrap in `@()` — e.g., `@($collection | Where-Object { ... }).Count`
   - Variables built with `$x = @(); $x += $item` — with a single `+=`, `$x` may still be an array, but function return values may unwrap it

4. **Reserved variable names**: PowerShell reserves several automatic variable names. Using them as local variables causes subtle bugs:
   - `$input` — reserved for pipeline input; using it as a variable silently captures pipeline data instead
   - `$args` — reserved for unbound arguments in simple functions
   - `$this` — reserved for script blocks used as methods
   - Search for `$input`, `$args`, `$this` used as local variable assignments

5. **`$WhatIf` vs `$WhatIfPreference`**: Scripts using `[CmdletBinding(SupportsShouldProcess)]` get the `-WhatIf` switch automatically, but the corresponding variable is `$WhatIfPreference`, not `$WhatIf`. Under strict mode, referencing `$WhatIf` directly throws because it is not declared as a parameter.

6. **Process execution safety**: Check all `Start-Process`, `[System.Diagnostics.Process]::Start()`, and `Invoke-Expression` calls:
   - `WaitForExit()` without a timeout parameter blocks indefinitely
   - Standard output/error must be read BEFORE `WaitForExit()` to avoid deadlocks
   - Missing `$proc.Dispose()` leaks handles
   - `Invoke-Expression` with any user-influenced string is command injection

7. **Comment style in code blocks**: PowerShell uses `#` for line comments and `<# #>` for block comments. C-style `/* */` comments are NOT comments in PowerShell — the `/` is a division operator and `*` is a wildcard, causing parse errors or unexpected expression evaluation.

8. **Credential handling**: Check for:
   - Passwords stored in plain text variables (use `SecureString`)
   - Credentials written to config files in plain text
   - `ConvertFrom-SecureString` without `-Key` parameter stores credentials tied to the machine/user DPAPI context only
   - [SPECIFIC_CREDENTIAL_LOCATIONS]

9. **Parameter block integrity**: Verify:
   - Every script with `[CmdletBinding()]` has a matching `param()` block
   - Comment-based help (`<# .SYNOPSIS #>`) is well-formed (matched `<#` and `#>` tags)
   - No parameters accidentally removed during edits (diff against previous version if available)

Search all `.ps1` and `.psm1` files for these patterns:
- `\.Count` (without preceding `@(`)
- `\$\w+\.[a-z][a-zA-Z]+` (direct camelCase property access on JSON objects)
- `\$input\b`, `\$args\b`, `\$this\b` (reserved variable usage)
- `\$WhatIf\b` (should be `$WhatIfPreference`)
- `WaitForExit\(\)` (missing timeout)
- `Invoke-Expression`
- `/\*` (C-style comment in PowerShell)
- `@\(.*@\(` (nested array-of-arrays without comma separators)
- `ConvertFrom-Json` (trace all downstream property accesses)

Provide a detailed findings report with file paths, line numbers, code snippets, and severity ratings (Critical/High/Medium/Low/Info). **Do NOT include actual credential values, API keys, tokens, or passwords in your output** — use `[REDACTED]` placeholders.
~~~

## Customization Guide

> **⚠️ Placeholder Safety:** Placeholder values are substituted directly into the prompt text. A crafted value could act as a prompt injection. Only use placeholder values you trust — do not accept them from untrusted sources.

| Placeholder | Example Values |
|-------------|---------------|
| `[APPLICATION_NAME]` | sql-cert-inspector, MyDeploymentTool |
| `[REPO_PATH]` | Full path to the repository root |
| `[SCRIPT_PATH]` | `scripts/Deploy.ps1`, `tools/Invoke-HealthCheck.ps1` |
| `[SPECIFIC_JSON_LOCATIONS]` | `ConvertFrom-Json at line 250 in Invoke-HealthCheck.ps1 — trace all property accesses on the result` |
| `[SPECIFIC_CREDENTIAL_LOCATIONS]` | `SMTP credential handling in Send-Email function`, `Windows Credential Manager P/Invoke in Helper Functions region` |

## What Good Looks Like

- All JSON property accesses guarded by `$obj.PSObject.Properties.Name -contains 'prop'` or via a `Get-SafeProperty` helper function
- All `Where-Object` results wrapped in `@()` before accessing `.Count`
- Array-of-arrays use the unary comma operator `,@()` to prevent flattening
- No reserved variable names (`$input`, `$args`) used as local variables
- `$WhatIfPreference` used instead of `$WhatIf` for ShouldProcess checks
- `WaitForExit()` called with a timeout; process disposed after use
- Passwords handled as `SecureString`; config files store usernames only
- Comment-based help present and well-formed on all exported functions
- `Set-StrictMode -Version Latest` enabled and script passes cleanly
- PowerShell parser validation (`[Parser]::ParseFile()`) returns zero errors
