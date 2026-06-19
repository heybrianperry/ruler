# Isolate MCP Server Configuration State Using Set-Based Deduplication: Parse Validate Json

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is active for all agent configuration systems that manage MCP server state across nested directory hierarchies.

## Context

- The system manages MCP (Model Context Protocol) server configurations across multiple agents (Claude, Copilot, Windsurf) in nested directory structures with per-directory .ruler/ruler.toml files
- Configuration state must be deduplicated to prevent duplicate server entries when merging configurations from parent and child directories
- The codebase uses Set data structures (expectedGitignoreEntries.add()) to track unique configuration artifacts and prevent redundant entries in version control ignore files
- Multiple JSON configuration files (.mcp.json) are parsed and validated across directory boundaries, requiring consistent state management to avoid conflicts
- The system must handle backup files (.mcp.json.bak) and maintain referential integrity between configuration files and gitignore entries

## Problem Statement

When managing MCP server configurations across nested directory hierarchies with multiple agent-specific config files, duplicate entries can accumulate during configuration merging, leading to inconsistent state, redundant gitignore entries, and potential configuration conflicts. Without explicit deduplication boundaries, the system risks propagating stale or conflicting server definitions across directory scopes.

## Decision

1. MUST: Parse and validate JSON configuration files using JSON.parse() before merging or applying configuration state

## Policy Block

- MUST Parse and validate JSON configuration files using JSON.parse() before merging or applying configuration state

In scope:
- MCP server configuration files (.mcp.json) managed by agent configuration systems
- Gitignore entries for agent-specific configuration artifacts
- Nested directory hierarchies with per-directory .ruler/ruler.toml configuration
- Configuration backup files with .bak extension
- JSON-based configuration state managed by FirebenderAgent and similar agent implementations

Out of scope:
- Non-JSON configuration formats (YAML, TOML parsing is handled separately)
- Global agent configuration not scoped to directory hierarchies
- Runtime MCP server process management or lifecycle
- Network communication with remote MCP servers
- User-facing configuration UI or interactive prompts

Exceptions:
- EXC-001: Legacy configuration files exist without backup capability (e.g., read-only filesystems)

## Rationale

- The evidence shows explicit use of Set.add() for expectedGitignoreEntries deduplication, indicating a deliberate architectural choice to prevent duplicate state accumulation
- Multiple JSON.parse() calls across nested directory configurations (projectRoot, moduleDir, submoduleDir) demonstrate the need for validated state boundaries to prevent malformed configuration propagation
- The test evidence verifies that server definitions are isolated by directory (root_stdio only in root, module_remote only in module), proving the pattern enforces scope-based configuration boundaries
- The pattern reduces configuration drift and merge conflicts by establishing clear ownership boundaries for MCP server definitions at each directory level

## Consequences

Positive:
- Eliminates duplicate gitignore entries and configuration artifacts through Set-based deduplication
- Prevents configuration conflicts by isolating MCP server definitions to their declared directory scope
- Enables safe configuration rollback through automatic backup file creation
- Provides clear ownership boundaries for configuration state in monorepo or nested project structures

Negative:
- Requires additional memory overhead for Set data structures during configuration processing
- Increases filesystem I/O due to backup file creation before each configuration write
- May cause confusion when parent directory server definitions are not automatically inherited by child directories
- Adds complexity to configuration merging logic with explicit scope isolation rules

## Alternatives

- Use array-based configuration state with manual duplicate checking via indexOf() or includes() (rejected)
  Rejected because: Array-based duplicate checking has O(n) lookup complexity compared to Set's O(1), and does not provide the same semantic clarity for deduplication intent
  When valid: Only acceptable for very small configuration sets (< 10 entries) where performance is not a concern
- Implement global MCP server registry with automatic inheritance down directory tree (rejected)
  Rejected because: Global inheritance violates the observed pattern of explicit per-directory scope isolation and would reintroduce the duplicate entry problem the Set-based approach solves
  When valid: Valid for simpler single-directory projects without nested configuration requirements
- Use database or external state store for configuration deduplication (deferred)
  Rejected because: Adds external dependency and deployment complexity not justified by current evidence of file-based configuration
  When valid: Consider for distributed systems with multiple concurrent configuration writers or when configuration state exceeds filesystem performance limits

## Risks

- Set-based deduplication may mask legitimate duplicate entries that should trigger validation errors (e.g., same server name with different configurations)
  Mitigation: Implement key-based uniqueness checks that include configuration content hash, not just artifact paths. Add validation tests that detect conflicting server definitions with identical names.
  Owner: Engineering team
- JSON.parse() failures on malformed configuration files may leave system in inconsistent state if error handling is insufficient
  Mitigation: Wrap all JSON.parse() calls in try-catch blocks with console.warn() logging as shown in evidence. Implement configuration validation schema using JSON Schema or similar before applying changes.
  Owner: Engineering team
- Backup file accumulation (.bak files) may consume excessive disk space in long-running systems without cleanup policy
  Mitigation: Implement backup rotation policy (keep last N backups) or time-based cleanup. Add monitoring for backup file count and disk usage.
  Owner: Operations team

## Implementation Notes

- Initialize Set data structures at the beginning of configuration processing functions before any add() operations to ensure clean state
- Use path.relative() and path.sep normalization when adding filesystem paths to Sets to ensure consistent key representation across platforms
- Implement extractServers() helper functions to normalize access to configuration data structures (mcpServers vs servers field names) as shown in test evidence
- Create backup files synchronously before writing new configuration to ensure atomic rollback capability on write failure
- Log all JSON.parse() errors with file path context to aid debugging of malformed configuration files

## Continuation Context


Verify commands:
- grep -r 'new Set<' src/ tests/ | grep -i 'gitignore\|config\|mcp' # Verify Set usage for configuration deduplication
- grep -r 'JSON\.parse' src/ tests/ | grep -v 'try\|catch' | wc -l # Should be 0 - all JSON.parse calls must have error handling
- find . -name '*.mcp.json' -exec sh -c 'jq empty "$1" 2>/dev/null || echo "Invalid JSON: $1"' _ {} \; # Validate all MCP JSON files

Accept when:
- All configuration artifact collections use Set data structures and demonstrate O(1) membership testing in code review
- All JSON.parse() calls are wrapped in try-catch blocks with appropriate error logging to console.warn() or similar
- Integration tests verify that MCP server definitions are isolated by directory scope and do not leak between parent/child configurations
- Backup files are created before configuration writes and gitignore entries include both primary and .bak file patterns

## Enforcement

- Verified by: Integration tests that verify Set-based deduplication behavior across nested directory configurations
- Verified by: Code review checklist requiring Set usage for all configuration artifact collections
- Verified by: Static analysis rules detecting JSON.parse() calls without surrounding try-catch blocks
- Verified by: CI pipeline validation of all .mcp.json files using jq or equivalent JSON validator
- Violation handling: CI build failure if JSON.parse() calls lack error handling or if configuration validation tests fail
- Violation handling: Code review rejection for array-based configuration state that should use Set deduplication
- Violation handling: Runtime console.warn() logging for malformed JSON files with graceful degradation to skip invalid configurations
- Violation handling: Post-deployment monitoring alerts for excessive backup file accumulation or configuration parse errors
- Exception process: Document exception rationale in code comments with reference to EXC-001 or new exception ID
- Exception process: Obtain engineering lead approval for exceptions via pull request review
- Exception process: Add compensating controls (e.g., alternative validation, manual review) when Set-based deduplication cannot be used
- Exception process: Review all active exceptions quarterly to determine if architectural constraints have changed