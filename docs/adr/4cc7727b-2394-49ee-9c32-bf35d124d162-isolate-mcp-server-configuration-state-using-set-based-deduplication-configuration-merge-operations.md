# Isolate MCP Server Configuration State Using Set-Based Deduplication: Configuration Merge Operations

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is active for all agent configuration management code that handles MCP server definitions and gitignore entry tracking.

## Context

- The codebase manages MCP (Model Context Protocol) server configurations across multiple agents (Claude, Copilot, Windsurf) with per-directory configuration files that must be merged and deduplicated
- Agent configuration files are written to multiple nested directories (projectRoot, moduleDir, submoduleDir) with inheritance and override semantics requiring state isolation to prevent duplicate entries
- Gitignore entries for MCP configuration files and their backups must be tracked without duplication across multiple write operations in nested directory structures
- JSON parsing of existing MCP configuration files from the filesystem requires validation and error handling to prevent corruption of agent state
- The system uses Set data structures (expectedGitignoreEntries.add, seen.add) to maintain unique collections of configuration keys and file paths during multi-agent configuration application

## Problem Statement

When managing MCP server configurations across multiple agents and nested directory hierarchies, duplicate entries in gitignore files and configuration state can occur if deduplication is not enforced, leading to configuration drift, merge conflicts, and potential security issues from inconsistent agent behavior.

## Decision

1. SHOULD: Configuration merge operations SHOULD validate that parent directory servers do not leak into child directory configurations unless explicitly inherited

## Policy Block

- SHOULD Configuration merge operations SHOULD validate that parent directory servers do not leak into child directory configurations unless explicitly inherited

In scope:
- All code paths that write MCP configuration files (.mcp.json, firebender.json) for any agent
- Functions that aggregate gitignore entries for agent configuration files and backups
- JSON parsing operations that read existing agent configuration state from the filesystem
- Configuration merge logic that combines MCP server definitions from multiple directory levels
- State tracking for seen configuration keys during multi-agent application workflows

Out of scope:
- Non-MCP agent configuration files that do not use server-based architecture
- User-provided custom configuration that explicitly opts out of deduplication
- Test fixtures and mock data that intentionally contain duplicates for validation
- Legacy configuration migration scripts that require duplicate detection for cleanup

Exceptions:
- EXC-001: Migration scripts need to detect and report duplicate entries in legacy configurations

## Rationale

- The evidence shows Set-based deduplication (expectedGitignoreEntries.add, seen.add) is already used in the integration test and agent implementation, indicating this is an established pattern for preventing duplicate configuration entries
- Multiple JSON.parse operations on MCP configuration files across nested directories (projectRoot, moduleDir, submoduleDir) require consistent error handling to prevent state corruption when files are malformed or missing
- The test explicitly validates that MCP servers do not leak between directory levels (rootServers should not have module_remote, moduleServers should not have root_stdio), demonstrating the importance of isolated configuration state
- Gitignore management for both primary files and .bak backups requires deduplication to prevent accumulation of duplicate entries across multiple configuration application runs

## Consequences

Positive:
- Eliminates duplicate gitignore entries and MCP server definitions across nested directory configurations
- Provides clear error boundaries when JSON parsing fails, preventing silent corruption of agent state
- Enables safe concurrent or sequential configuration updates without state collision
- Simplifies validation logic by guaranteeing uniqueness constraints at the data structure level

Negative:
- Set-based deduplication adds memory overhead for tracking seen keys during configuration aggregation
- Requires additional code to convert between Set and array representations for serialization
- May mask configuration errors where duplicate keys indicate actual conflicts that should be resolved explicitly
- Increases complexity of configuration merge logic compared to simple array concatenation

## Alternatives

- Use array-based configuration storage with post-processing deduplication filters (rejected)
  Rejected because: Post-processing deduplication is error-prone and allows duplicates to exist temporarily in memory, increasing risk of state corruption if errors occur mid-processing
  When valid: Acceptable only for read-only configuration analysis where duplicates need to be detected and reported
- Implement global shared state for all MCP server configurations across directory levels (rejected)
  Rejected because: Violates the isolation requirement demonstrated in tests where each directory level must maintain independent server configurations with explicit inheritance
  When valid: Only valid for single-directory projects with no nested configuration requirements
- Use database or external state store for configuration deduplication (deferred)
  Rejected because: Adds external dependency and complexity for a problem that can be solved with in-memory data structures
  When valid: May be reconsidered if configuration state needs to be shared across multiple processes or persisted between runs

## Risks

- Set-based deduplication may hide legitimate configuration conflicts where the same MCP server key is defined differently in multiple locations
  Mitigation: Implement validation that compares server definitions before deduplication and logs warnings when conflicting definitions are merged
  Owner: engineering team
- JSON parsing errors in existing configuration files could cause complete configuration application failure rather than partial updates
  Mitigation: Implement graceful degradation where parsing errors are logged but allow new configuration to be written, with backup files preserved for recovery
  Owner: engineering team
- Memory usage for Set-based tracking could grow unbounded in projects with very large numbers of nested directories and MCP servers
  Mitigation: Profile memory usage in large monorepo scenarios and implement streaming or chunked processing if needed
  Owner: engineering team

## Implementation Notes

- Use TypeScript Set<string> for tracking gitignore entries and configuration keys, with explicit .add() calls at each insertion point
- Wrap all JSON.parse() calls for MCP configuration files in try-catch blocks with console.warn or structured logging on failure
- When adding gitignore entries for MCP files, always add both the primary path and the .bak backup path in a single operation
- Implement helper functions like extractServers() to normalize access to MCP server definitions across different agent configuration schemas (mcpServers vs servers keys)
- Validate configuration isolation by asserting that parent directory server keys do not appear in child directory configurations unless explicitly inherited

## Continuation Context


Verify commands:
- grep -r 'new Set<' src/ tests/ | grep -E '(gitignore|seen|keys)' | wc -l
- grep -r 'JSON\.parse' src/agents/ | xargs grep -L 'try.*catch' | wc -l
- grep -r '\.add.*\.bak' tests/integration/ | grep -c 'expectedGitignoreEntries'

Accept when:
- All MCP configuration aggregation code uses Set data structures for deduplication, verified by grep showing Set usage in relevant files
- All JSON.parse operations on agent configuration files are wrapped in try-catch blocks with error handling
- Gitignore entry addition includes both primary and .bak backup paths, verified in integration tests

## Enforcement

- Verified by: Integration tests that validate no duplicate MCP server keys appear in generated configuration files
- Verified by: Code review checklist requiring Set-based deduplication for new configuration aggregation code
- Verified by: Static analysis rules detecting JSON.parse without surrounding try-catch blocks in agent configuration paths
- Violation handling: CI pipeline fails if integration tests detect duplicate MCP server keys in generated configurations
- Violation handling: Code review blocks merge if JSON parsing lacks error handling in agent configuration code
- Violation handling: Runtime warnings logged when duplicate configuration keys are detected during application
- Exception process: Document the specific reason why deduplication cannot be applied in the code comments
- Exception process: Obtain engineering lead approval for exceptions via pull request review
- Exception process: Add compensating validation logic to detect and handle potential duplicates explicitly