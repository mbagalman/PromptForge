# Coding Prompts

System prompts for code-quality work: hardening legacy Python into production-grade Python 3.11+ and auditing/refactoring SQL across major RDBMS and cloud warehouse engines. Each prompt is platform-portable and works standalone — they are not designed to chain.

## Files

- **[python-hardener.md](python-hardener.md):** Refactors raw or legacy Python into modern, cross-platform Python 3.11+. Audits across five dimensions (cross-platform hazards, resource integrity, type safety, exception handling, security) and produces a structured response (Diagnostic Assessment → Systemic Vulnerabilities → Optimized Implementation → Execution Protocol → Change Log) with the rewritten script in a single code block.
- **[sql-optimization-engineer.md](sql-optimization-engineer.md):** Audits and refactors SQL queries for performance and maintainability. Engine-aware — applies optimizer specifics for Snowflake, BigQuery, Postgres, MySQL, SQL Server, and Redshift when the user names the target; falls back to engine-agnostic principles otherwise. Produces a four-section response (Query Review → Issues Identified → Refactored SQL → Optimization Summary) with a CTE-modular rewrite.

## Inputs

Each prompt declares its required inputs in its YAML frontmatter (`required_inputs` field). At a high level: the Python hardener takes a Python source file or pasted code plus optional context (target Python version, runtime, banned imports); the SQL engineer takes the query plus optional context (target engine, schema details, performance targets).

## How to Use

Paste the full text of a prompt into your assistant builder's instructions field (Claude Projects, Gemini Gems, or OpenAI Custom GPTs), then provide the code or query in your first message. Both prompts are designed to work autonomously — they will state defensible assumptions in their Diagnostic Assessment or Query Review section rather than asking clarifying questions, unless the input is genuinely unparseable.

## Platform setup

Both prompts paste into the assistant builder's instruction field with no required knowledge files. Tool requirements:

| Prompt | Code execution | File creation | Web search |
| --- | --- | --- | --- |
| [python-hardener.md](python-hardener.md) | Optional — useful for verifying the rewrite parses and runs, but the contract does not require it | Not needed | Off |
| [sql-optimization-engineer.md](sql-optimization-engineer.md) | Not needed — audits and rewrites SQL text; does not execute it | Not needed | Off |

**Claude Projects:** paste into Project Instructions. No project knowledge files required. Toggle Code Execution on for the Python hardener if you want it to validate its own rewrites.

**OpenAI Custom GPTs:** paste into Instructions. Leave Knowledge empty unless you have a stable reference (a house style guide, organization-specific banned-import list, schema dictionary). Custom GPTs do not treat knowledge as ambient — if you add files, append one sentence to the prompt's Knowledge & sources section telling the assistant to consult them.

**Gemini Gems:** paste into Instructions. Same knowledge-file caveat as Custom GPTs.

### Suggested conversation starters

- **Python Hardener:** "Harden this script." / "Make this production-ready." / "Audit this for cross-platform issues." / "Add type hints and fix the exception handling."
- **SQL Optimization Engineer:** "Optimize this query." / "Why is this slow?" / "Refactor this for Snowflake." / "Audit this for SARGability."

## Maintenance notes

Guidance for anyone editing these prompts.

### Python Hardener

- **Most likely to drift:** the audit dimensions list (cross-platform / resource / typing / exception / security). New failure modes (e.g., async-specific resource leaks, modern type-checker friction) should be added as bullets there rather than scattered through the Change Log examples.
- **Coupled sections:** the Optimized Implementation requirements and the Change Log structure. If a new requirement is added to the rewrite (e.g., requiring `tomllib` over third-party TOML parsers on 3.11+), update both together so the Change Log has a place to surface the change.
- **Regression tests:** keep a small set of input scripts representing the failure modes the guardrails cover — a script with hard-coded credentials, one using `os.path` strings, one with bare `except: pass`, one with `pickle.loads` on user input, one that imports a non-existent module, one in a language that isn't Python. Each should hit the documented fallback or section.
- **Do not flatten:** the five-section output contract. The diagnosis-before-rewrite ordering and the Change Log are load-bearing — they make the rewrite reviewable rather than a black-box replacement.

### SQL Optimization Engineer

- **Most likely to drift:** the engine-specific knowledge section. New engines (DuckDB, ClickHouse, Databricks SQL) or material engine version changes (Postgres CTE materialization rules, Snowflake feature releases) should be added there rather than woven into the audit dimensions.
- **Coupled sections:** the Issues Identified categories (`Architecture`, `Performance`, `Style`) and the Optimization Summary dimension labels (`SARGability` / `join cardinality` / `projection` / `architecture` / `engine-specific`). If a new category is added to one, add it to the other.
- **Regression tests:** keep a small set of input queries — one with a non-SARGable predicate, one with a Cartesian product risk, one with `SELECT *` in a CTE, one with a window function that should use `QUALIFY`, one that's Snowflake-specific against an unspecified engine, one that's syntactically broken. Each should hit the documented fallback or produce a specific finding.
- **Do not flatten:** the four-section output contract and the Import → Logical → Final CTE pattern. The CTE modularity is load-bearing for readability in code review, which is half the value of the rewrite.

## License

Distributed under the MIT License. See [../LICENSE](../LICENSE) for details.
