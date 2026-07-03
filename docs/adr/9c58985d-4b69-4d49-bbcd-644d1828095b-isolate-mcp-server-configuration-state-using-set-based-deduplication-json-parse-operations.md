# Isolate MCP Server Configuration State Using Set-Based Deduplication: Json Parse Operations

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is active for all MCP configuration management code paths that handle server registration, gitignore updates, and configuration state tracking across nested directory hierarchies.

## Context

- The codebase manages MCP (Model Context Protocol) server configurations across nested directory hierarchies with per-agent and per-directory enablement rules
- Multiple agents (Claude, Copilot, Windsurf) require isolated configuration files that must be tracked in gitignore without duplication
- Configuration state must be parsed from JSON files and merged across directory boundaries while maintaining agent-specific enablement flags
- The system must handle backup file generation and gitignore updates atomically to prevent configuration loss or duplicate entries
- Set-based deduplication patterns appear in both test validation code and production agent configuration logic

## Problem Statement

When managing MCP server configurations across nested directories with multiple agents, duplicate gitignore entries and configuration keys can accumulate without explicit deduplication, leading to configuration drift, merge conflicts, and inconsistent agent behavior across directory boundaries.

## Decision

1. MUST: JSON.parse operations on MCP configuration files MUST be wrapped in error handling that logs parse failures and provides fallback behavior

## Policy Block

- MUST JSON.parse operations on MCP configuration files MUST be wrapped in error handling that logs parse failures and provides fallback behavior

In scope:
- MCP server configuration file generation and updates
- Gitignore entry management for .mcp.json, .mcp.json.bak, and agent-specific config files
- Configuration state tracking across nested .ruler directories
- JSON parsing and validation of existing MCP configuration files
- Backup file generation for Claude and other agent configurations

Out of scope:
- Non-MCP configuration file management
- Agent instruction file content (AGENTS.md)
- TOML configuration parsing for ruler.toml files
- Test harness setup and teardown logic
- File system path resolution outside MCP configuration contexts

## Rationale

- The evidence shows Set-based deduplication in both test validation (expectedGitignoreEntries.add) and production code (seen.add(key)), indicating a deliberate pattern for preventing duplicate state
- MCP configuration files are parsed from JSON across multiple directory levels (projectRoot, moduleDir, submoduleDir) requiring consistent deduplication to prevent configuration conflicts
- The pattern isolates configuration state per agent and per directory while maintaining global uniqueness constraints through Set data structures
- Error handling around JSON.parse operations in FirebenderAgent demonstrates defensive programming against malformed configuration files

## Consequences

Positive:
- Eliminates duplicate gitignore entries across nested directory hierarchies with multiple agent configurations
- Prevents configuration key collisions when merging MCP server definitions from multiple directory levels
- Provides O(1) lookup and insertion performance for configuration state tracking compared to array-based approaches
- Simplifies validation logic by guaranteeing unique entries in gitignore and configuration key collections

Negative:
- Set data structures require explicit conversion to arrays for serialization to JSON or file output
- Developers must remember to use Set.add() rather than array push() operations, increasing cognitive load
- Debugging Set contents requires conversion or iteration since Sets do not support index-based access
- Memory overhead of Set objects may be higher than arrays for small collections (< 10 items)

## Alternatives

- Use array-based collections with manual deduplication via filter/includes checks before insertion (rejected)
  Rejected because: O(n) lookup performance for duplicate checking scales poorly with nested directory depth and multiple agents; Set provides O(1) guarantees
  When valid: Valid only for single-directory, single-agent configurations with < 5 total entries
- Use Map data structures with configuration keys as keys and metadata as values (rejected)
  Rejected because: Evidence shows simple membership tracking without associated metadata; Map overhead unnecessary for boolean presence checks
  When valid: Valid if configuration entries require associated metadata like timestamps, sources, or validation state
- Deduplicate at write time by reading existing gitignore/config and filtering duplicates (rejected)
  Rejected because: Introduces race conditions in concurrent configuration updates and requires additional file I/O for every insertion
  When valid: Valid for single-threaded batch processing where all entries are known upfront

## Risks

- Set iteration order is not guaranteed in older JavaScript engines, potentially causing non-deterministic gitignore file ordering
  Mitigation: Convert Set to sorted array before writing to files; add integration tests validating gitignore content regardless of order
  Owner: engineering team
- JSON.parse failures on malformed MCP configuration files could propagate uncaught exceptions if error handling is incomplete
  Mitigation: Enforce try-catch wrappers around all JSON.parse calls with fallback to empty configuration objects; log parse errors with file paths
  Owner: engineering team
- Set-based deduplication may mask configuration errors where duplicate keys indicate misconfiguration rather than intentional redundancy
  Mitigation: Add debug logging when duplicate keys are detected before deduplication; provide --strict mode that fails on duplicates
  Owner: engineering team

## Implementation Notes

- Initialize Set collections at the beginning of configuration processing functions: const seen = new Set<string>()
- Use Set.add() for all gitignore entry accumulation and configuration key tracking operations
- Convert Sets to arrays using Array.from(set) or [...set] before JSON serialization or file writing
- Wrap all JSON.parse calls in try-catch blocks with console.warn or logger.error for parse failures
- Normalize file paths to relative forward-slash format before adding to gitignore Sets to ensure cross-platform consistency

## Continuation Context


Verify commands:
- grep -r 'new Set<' src/ tests/ | grep -E '(gitignore|mcp|config)' | wc -l
- grep -r 'JSON\.parse' src/ | xargs grep -L 'try\|catch' | wc -l
- grep -r '\.add(' src/ tests/ | grep -E '(expectedGitignoreEntries|seen)' | wc -l

Accept when:
- All gitignore entry collections use Set data structures (verify_commands[0] returns > 0)
- All JSON.parse operations are wrapped in try-catch blocks (verify_commands[1] returns 0)
- Set.add() patterns are present in configuration state tracking code (verify_commands[2] returns > 0)

## Enforcement

- Verified by: Integration tests validating gitignore content uniqueness across nested directory configurations
- Verified by: Code review checklist requiring Set usage for configuration state collections
- Verified by: Static analysis rules detecting array-based duplicate checking patterns in MCP configuration code
- Violation handling: CI pipeline fails if integration tests detect duplicate gitignore entries
- Violation handling: Code review blocks merge if array-based deduplication patterns are introduced in MCP configuration paths
- Violation handling: Runtime warnings logged when duplicate configuration keys are detected during processing
- Exception process: Document technical justification for array-based approach in ADR comment or inline code comment
- Exception process: Obtain approval from two senior engineers with evidence that Set-based approach is inappropriate
- Exception process: Add explicit test coverage demonstrating correctness of alternative deduplication approach