---
name: codegraph
description: >-
  Code dependency graph analysis, symbol call chains, upstream/downstream callers and callees,
  and blast radius impact analysis using CodeGraph. Use whenever you need to explore code structure,
  trace function invocations, find callers, evaluate refactoring impact, or understand cross-file
  dependencies before editing code.
metadata:
  repo: colbymchenry/codegraph
  version: "1.0.0"
---

# CodeGraph — Code Dependency Graph & Structural Intelligence

CodeGraph is a pre-indexed AST knowledge graph (powered by tree-sitter and SQLite) for tracking code symbols, call hierarchies, and cross-file dependencies across 30+ programming languages.

## When to Use

- **Before modifying any existing function or component**: Inspect its upstream callers and downstream dependencies to calculate the blast radius.
- **Tracing execution paths**: Determine how data or control flows from an API route or UI handler down to the database or utility layer.
- **Refactoring & renaming**: Identify all direct and indirect references across the entire codebase before changing function signatures or data structures.
- **Debugging & root-cause analysis**: Follow the actual AST call chain instead of guessing or relying on imprecise text searches.

## Core Commands & Workflows

### 1. Initialization (Per Project)
Ensure the repository has an active CodeGraph index:
```bash
codegraph init
```
This builds or syncs `.codegraph/` and starts auto-tracking file changes.

### 2. Exploring Symbols & Call Paths
Use `codegraph explore` or the MCP `codegraph_explore` tool to query symbols, endpoints, and call paths:
```bash
# Query a symbol's definition, callers, and blast radius
codegraph explore "<symbol_name>"

# Trace call paths between two symbols or endpoints
codegraph explore "<sourceSymbol> <targetSymbol>"

# Natural-language architectural queries
codegraph explore "how does auth middleware connect to user session"
```

### 3. Impact & Dependency Analysis
To inspect what depends on a target function before making edits:
```bash
# Find all callers of a function
codegraph callers <function_name>

# Find all callees / dependencies invoked by a function
codegraph callees <function_name>

# Calculate blast radius / affected files
codegraph impact <symbol_or_file>
```

## Anti-Patterns & Iron Laws

1. **Never guess dependency impacts**: Do not modify an exported function or API endpoint without verifying all callers across the codebase.
2. **Do not use grep for call-graph tracing**: Regex/grep misses indirect calls, dynamic dispatches, and interface implementations, while producing false positives on shared names.
3. **Verify blast radius before commits**: After changing shared logic, inspect affected downstream components and ensure corresponding unit/integration tests are updated.
