# File Path & Process Execution

**Domain:** Path traversal, command injection, process spawning  
**Agent Type:** `explore`  
**Priority:** High

## Purpose

Audit all file system operations and external process execution for injection and traversal vulnerabilities. `Process.Start` / `child_process.exec` / `subprocess.run` with user-influenced arguments is one of the most dangerous patterns in application code.

## Prompt

~~~
You are conducting a security assessment of the [APPLICATION_NAME] repository at [REPO_PATH].

Your task: **Audit file path handling and process execution** across the entire codebase.

Specifically investigate:

1. **External process execution**: Find ALL process spawning call sites. Check:
   - Is shell execution enabled? (e.g., `UseShellExecute=true`, `shell=true`, `exec` vs `execFile`)
   - Are file paths or arguments user-controlled or derived from user input?
   - Could a malicious path or argument lead to arbitrary command execution?
   - Are URLs opened via shell execute validated for safe protocols (http/https only)?
   - [SPECIFIC_PROCESS_START_LOCATIONS]

2. **File path construction**: Search for:
   - Path building with user-influenced components
   - String concatenation to build file paths (vs. Path.Combine / path.join)
   - Glob patterns that could match unintended files
   - Database file path construction
   - Temp file creation patterns

3. **Path traversal**: Could any file read/write operations be tricked into accessing files outside expected directories? Check:
   - Can `../` sequences escape the intended directory?
   - Are paths canonicalized before access checks?
   - [SPECIFIC_FILE_OPERATION_LOCATIONS]

4. **File permissions**: Are created files/directories using appropriate permissions? Check:
   - Are sensitive files readable only by the owning user?
   - Are temp files created securely (unpredictable names, restrictive permissions)?
   - Do directories inherit overly permissive ACLs?

Search all source files for: Process.Start, ProcessStartInfo, child_process, subprocess, exec, spawn, Path.Combine, path.join, os.path.join, File.Create, File.Open, File.Write, File.Read, Directory.Create, fs.readFile, fs.writeFile, open(), glob, read_parquet, LOAD, ATTACH.

Provide a detailed findings report with file paths, line numbers, code snippets, and severity ratings.
~~~

## Customization Guide

| Placeholder | Example Values |
|-------------|---------------|
| `[SPECIFIC_PROCESS_START_LOCATIONS]` | `Check the View Log button in MainWindow.xaml.cs`, `Check the PDF export in ReportService` |
| `[SPECIFIC_FILE_OPERATION_LOCATIONS]` | `Archive service file paths`, `Upload handler in controllers/`, `Import/export functionality` |

## What Good Looks Like

- `UseShellExecute=false` / `shell: false` for all process spawning
- File paths validated against an expected root directory
- Protocol whitelist (http/https only) for URL opening
- `Path.Combine` / `path.join` used consistently (never string concatenation)
- TOCTOU avoided: files opened directly without pre-checking existence
- Temp files use `Path.GetTempFileName()` or equivalent secure creation
- Created directories have restrictive ACLs (current user only for sensitive data)
