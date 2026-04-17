# SQL Injection / Dynamic SQL

**Domain:** Query construction, parameterization, input sanitization  
**Agent Type:** `explore`  
**Priority:** High

## Purpose

Audit all database query construction for injection vulnerabilities. This covers both traditional SQL databases and embedded/local databases (SQLite, DuckDB, etc.) where string interpolation into queries is equally dangerous.

## Prompt

~~~
You are conducting a security assessment of the [APPLICATION_NAME] repository at [REPO_PATH].

Your task: **Audit for SQL injection and unsafe dynamic SQL** across the entire codebase.

Specifically investigate:

1. **Application-side query construction**: Search all source files for query building. Look for:
   - String concatenation or interpolation used to build queries (`$"SELECT..."`, `"SELECT " + variable`, `string.Format`, f-strings, template literals)
   - Proper use of parameterized queries ([PARAMETER_SYNTAX])
   - Any user input flowing into query strings without parameterization
   - ORM raw query methods ([RAW_QUERY_METHODS])

2. **Embedded/local database queries**: Focus on [LOCAL_DB_SERVICE_FILES]. Check:
   - Are all WHERE clause values parameterized?
   - Are file paths in functions like read_parquet(), LOAD, ATTACH properly escaped?
   - Are table/column names validated against a whitelist when dynamically constructed?

3. **External query interfaces**: Check [API_OR_TOOL_FILES] — these accept queries from external sources. How are they validated or sandboxed?

4. **Database scripts**: Check [SCRIPT_DIRECTORIES] for dynamic SQL construction:
   - Are EXEC/EXECUTE calls using parameterized sp_executesql?
   - Are identifiers properly quoted (QUOTENAME, bracket escaping, backtick escaping)?
   - Do any scripts accept and execute user-provided strings?

5. **Data access layer**: Check [DATA_ACCESS_FILES] for query construction patterns.

Search patterns:
- `string.Format.*SELECT|INSERT|UPDATE|DELETE`
- `\$".*SELECT|INSERT|UPDATE|DELETE` (C# interpolation)
- `f".*SELECT|INSERT|UPDATE|DELETE` (Python f-strings)
- `` `.*SELECT|INSERT|UPDATE|DELETE` `` (JS template literals)
- `"SELECT.*" \+` (concatenation)
- `EXEC\s*\(|EXECUTE\s*\(`
- `sp_executesql`
- `cmd\.CommandText\s*=`
- `CreateCommand|prepare|query\(`
- `.raw\(|.rawQuery\(|RawSql|FromSqlRaw`

Provide a detailed findings report with file paths, line numbers, code snippets, and severity ratings (Critical/High/Medium/Low/Info). Distinguish between cases where the interpolated value comes from trusted internal sources vs. potentially untrusted external input.
~~~

## Customization Guide

| Placeholder | Example Values |
|-------------|---------------|
| `[PARAMETER_SYNTAX]` | `DuckDBCommand.Parameters, SqlCommand.Parameters` (.NET), `cursor.execute(sql, params)` (Python), `pool.query(sql, [params])` (Node) |
| `[RAW_QUERY_METHODS]` | `FromSqlRaw, ExecuteSqlRaw` (EF Core), `Sequelize.query()` (Node), `connection.execute()` (Python) |
| `[LOCAL_DB_SERVICE_FILES]` | `Services/LocalDataService*.cs`, `repositories/*.py` |
| `[API_OR_TOOL_FILES]` | `Mcp/McpQueryTools.cs`, `api/query-endpoint.ts` |
| `[SCRIPT_DIRECTORIES]` | `install/*.sql`, `migrations/`, `db/seeds/` |
| `[DATA_ACCESS_FILES]` | `Services/DatabaseService*.cs`, `repositories/`, `dal/` |

## What Good Looks Like

- All user-facing queries use parameterized statements
- Table/column names validated against enum or whitelist when dynamic
- File paths in database functions escaped (single quotes doubled)
- ORM raw query methods used sparingly with explicit parameterization
- Dynamic SQL in scripts uses QUOTENAME/proper identifier quoting
- Clear separation between trusted-source interpolation (documented) and untrusted input (parameterized)
