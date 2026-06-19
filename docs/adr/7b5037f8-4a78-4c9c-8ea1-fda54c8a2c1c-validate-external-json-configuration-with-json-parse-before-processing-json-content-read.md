# Validate External JSON Configuration with JSON.parse Before Processing: Json Content Read

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all agent configuration processing and MCP server integration workflows.

## Context

- The system integrates with multiple AI coding agents (Claude, Copilot, Windsurf) through MCP (Model Context Protocol) configuration files that are read from and written to the filesystem
- Agent configuration files (.mcp.json, firebender.json) are external inputs that may be malformed, corrupted, or contain unexpected data structures
- The codebase processes nested directory structures with per-directory configuration inheritance, requiring validation at multiple filesystem levels
- Integration tests demonstrate that the system must parse and extract server configurations from JSON objects with varying schema structures (mcpServers vs servers properties)
- The fs/promises and path modules are used extensively to read configuration files across project root, module, and submodule directories

## Problem Statement

External JSON configuration files from filesystem sources must be validated for syntactic correctness and structural integrity before the system attempts to access properties or perform business logic operations, preventing runtime errors from malformed input and ensuring graceful degradation when configuration is invalid.

## Decision

1. MUST: All JSON content read from filesystem configuration files MUST be parsed using JSON.parse() before accessing properties or performing operations on the data structure

## Policy Block

- MUST All JSON content read from filesystem configuration files MUST be parsed using JSON.parse() before accessing properties or performing operations on the data structure

In scope:
- All MCP configuration files (.mcp.json) read from project root, module, and submodule directories
- Agent-specific configuration files (firebender.json) loaded from filesystem
- Any JSON configuration file processed by applyRulerConfig, loadExistingConfig, or applyAllAgentConfigs functions
- Test fixtures and integration test scenarios that validate configuration parsing behavior

Out of scope:
- JSON responses from HTTP APIs or network services (covered by separate API integration patterns)
- In-memory JSON serialization for internal data structures
- JSON.stringify() operations for writing configuration files
- Static JSON imports using TypeScript/JavaScript import statements

## Rationale

- The evidence shows 4 distinct JSON.parse() invocations across integration tests and agent configuration code, demonstrating a consistent pattern of validating external JSON before processing
- The FirebenderAgent implementation explicitly catches and logs JSON parse failures with console.warn, establishing precedent for graceful error handling
- Integration tests validate that parsed configuration objects can have varying structures (mcpServers vs servers), requiring defensive parsing before property access
- The nested directory structure (projectRoot, moduleDir, submoduleDir) with per-directory configurations increases the risk of malformed files, making validation critical for system stability

## Consequences

Positive:
- Prevents runtime crashes from malformed JSON configuration files, improving system resilience
- Enables graceful degradation when configuration files are corrupted or invalid, allowing the system to continue with defaults or fallbacks
- Provides clear error messages and logging context for debugging configuration issues
- Supports schema flexibility by validating syntax first, then checking for expected properties dynamically

Negative:
- Adds try-catch overhead to every configuration file read operation, slightly increasing code complexity
- May mask configuration errors if fallback behavior is too permissive, making it harder to detect misconfiguration
- Requires consistent error handling patterns across multiple modules (agents, paths, lib) to maintain reliability
- Does not validate semantic correctness of configuration values, only syntactic validity of JSON structure

## Alternatives

- Use a JSON schema validation library (e.g., ajv, zod) to validate both syntax and structure in a single step (rejected)
  Rejected because: Evidence shows only JSON.parse() usage without schema validation libraries; adding schema validation would be a new pattern not reflected in current codebase
  When valid: When configuration schemas become complex enough to require formal validation beyond property existence checks
- Assume all configuration files are valid and let the system crash on malformed JSON (rejected)
  Rejected because: FirebenderAgent explicitly implements try-catch error handling for JSON.parse(), demonstrating intentional defensive programming against invalid input
  When valid: Never valid for production systems that read external configuration files
- Pre-validate configuration files using a separate validation step before the main application runs (deferred)
  Rejected because: Current evidence shows runtime validation during configuration loading; pre-validation would require additional tooling not present in the codebase
  When valid: When configuration errors should block application startup rather than allowing graceful degradation

## Risks

- Inconsistent error handling across different modules may lead to some code paths crashing while others gracefully degrade
  Mitigation: Establish a shared utility function for JSON configuration parsing with standardized error handling and logging
  Owner: engineering team
- Silent failures with fallback behavior may hide configuration errors from users, making debugging difficult
  Mitigation: Ensure all JSON.parse() failures are logged with sufficient context (file path, error message) and consider surfacing warnings to users
  Owner: engineering team
- Performance degradation if large configuration files are parsed repeatedly without caching
  Mitigation: Implement configuration caching after successful parse and validation to avoid redundant filesystem reads and parsing
  Owner: engineering team

## Implementation Notes

- Wrap all fs.readFile() operations for JSON configuration files with JSON.parse() in try-catch blocks, following the pattern in FirebenderAgent.ts
- Use console.warn() or a structured logging framework to record parse failures with file path and error details
- After successful JSON.parse(), use optional chaining (?.) or explicit property checks before accessing nested configuration properties
- Consider extracting a shared parseJsonConfig(filePath: string) utility function to centralize error handling and logging logic across agents and configuration modules

## Continuation Context


Verify commands:
- grep -r 'JSON\.parse' src/ tests/ | grep -v 'try\|catch' | wc -l | grep '^0$'
- grep -r 'fs\.readFile.*\.json' src/ | grep -L 'JSON\.parse'
- npm test -- --testNamePattern='Nested MCP propagation' --verbose

Accept when:
- All JSON.parse() invocations for configuration files are wrapped in try-catch blocks with error logging
- Integration tests for nested MCP configuration successfully parse JSON from multiple directory levels without crashes
- Grep verification shows no unprotected JSON.parse() calls in configuration loading code paths

## Enforcement

- Verified by: Integration test suite validates JSON parsing across nested directory structures (tests/integration/nested-mcp-behavior.test.ts)
- Verified by: Code review checks for try-catch blocks around JSON.parse() in configuration loading functions
- Verified by: Static analysis with grep patterns to detect unprotected JSON.parse() calls in src/ and tests/ directories
- Violation handling: CI pipeline fails if integration tests detect unhandled JSON parse errors or crashes
- Violation handling: Code review rejects pull requests that read JSON configuration files without proper error handling
- Violation handling: Runtime errors from unvalidated JSON.parse() are logged and reported through error monitoring systems
- Exception process: Exceptions may be granted for internal JSON serialization that does not involve external file input
- Exception process: Static JSON imports using TypeScript import statements are exempt from runtime validation requirements
- Exception process: Exception requests must document why the JSON source is guaranteed to be valid and include compensating controls