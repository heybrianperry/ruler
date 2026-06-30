# Validate External JSON Configuration with JSON.parse Before Processing: External Json Configuration

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is active for all agent configuration processing and MCP server integration workflows that consume external JSON files.

## Context

- The system integrates multiple AI coding agents (Claude Code, GitHub Copilot, Windsurf) through MCP (Model Context Protocol) server configurations stored as JSON files in nested directory structures
- Agent configurations are read from external files (.mcp.json, firebender.json) that may be user-edited, corrupted, or malformed, requiring validation before processing
- The nested MCP propagation system processes per-directory configurations with agent-specific enable/disable flags, requiring reliable parsing of configuration hierarchies
- Integration tests demonstrate that the system must extract and validate server definitions from JSON structures with varying schemas (mcpServers vs servers properties)

## Problem Statement

External JSON configuration files consumed during agent integration and MCP server setup may contain malformed syntax, unexpected schemas, or corrupted data that could cause runtime failures, security vulnerabilities, or incorrect agent behavior if processed without validation. The system must reliably detect and handle invalid JSON before attempting to use configuration data.

## Decision

1. MUST: All external JSON configuration files MUST be parsed using JSON.parse with explicit error handling before accessing configuration properties

## Policy Block

- MUST All external JSON configuration files MUST be parsed using JSON.parse with explicit error handling before accessing configuration properties

In scope:
- MCP server configuration files (.mcp.json) in nested directory structures
- Agent-specific configuration files (firebender.json, copilot config, windsurf config)
- User-editable JSON configuration consumed by applyAllAgentConfigs and related integration workflows
- Backup configuration files (.mcp.json.bak) when restored or validated

Out of scope:
- JSON responses from HTTP APIs or remote services (covered by separate API validation rules)
- JSON serialization of internal data structures for output (covered by serialization rules)
- JSON used in test fixtures with known-valid content
- Programmatically generated JSON that never touches external storage

Exceptions:
- EXC-001: Configuration files are generated and immediately consumed within the same atomic transaction without external access

## Rationale

- The evidence shows JSON.parse called on fs.readFile results for .mcp.json files in projectRoot, moduleDir, and submoduleDir, establishing a consistent validation pattern across nested configurations
- FirebenderAgent demonstrates defensive parsing with try-catch around JSON.parse of existingContent, logging failures with console.warn, preventing crashes from corrupted user configurations
- Integration tests verify that parsed JSON structures require defensive property extraction (mcpServers vs servers), indicating schema variation requires validation beyond syntax checking
- The nested MCP propagation test creates backup files (.mcp.json.bak) and validates their presence, suggesting configuration integrity is critical enough to warrant backup and validation workflows

## Consequences

Positive:
- Prevents runtime crashes and undefined behavior from malformed JSON configuration files
- Enables graceful degradation with logged warnings when optional configurations are corrupted
- Provides clear error messages identifying which configuration file failed parsing, improving debuggability
- Supports safe integration of multiple agent configurations with varying schemas through defensive parsing

Negative:
- Adds parsing overhead and error handling boilerplate to every configuration read operation
- May silently fall back to defaults if error handling is too permissive, masking configuration issues
- Requires developers to implement consistent error handling patterns across all configuration consumers
- JSON.parse alone does not validate schema correctness, requiring additional validation layers for complex configurations

## Alternatives

- Use a JSON schema validation library (e.g., ajv, zod) for comprehensive validation beyond syntax checking (deferred)
  When valid: Should be adopted when schema complexity increases or when type safety requirements demand compile-time guarantees
- Trust configuration files without validation and let JSON.parse throw unhandled exceptions (rejected)
  Rejected because: Evidence shows FirebenderAgent explicitly catches parse errors to prevent crashes, indicating unhandled exceptions are unacceptable for user-editable configurations
  When valid: Never valid for external configuration files
- Pre-validate JSON files using a linter or validator before application startup (rejected)
  Rejected because: Does not protect against runtime corruption or user edits after validation, and nested MCP configs are processed dynamically per directory
  When valid: Useful as a complementary CI/CD check but insufficient as sole validation mechanism

## Risks

- Inconsistent error handling across different configuration consumers may lead to some code paths crashing while others gracefully degrade
  Mitigation: Establish shared utility functions for configuration loading with standardized error handling and logging patterns
  Owner: Engineering team
- JSON.parse validates syntax but not schema, allowing semantically invalid configurations to pass validation and cause downstream failures
  Mitigation: Implement schema validation for critical configuration properties after JSON parsing, document expected schemas in configuration files
  Owner: Engineering team
- Silent fallback to defaults when parsing fails may hide configuration errors from users who expect their settings to be applied
  Mitigation: Always log parsing failures with clear file paths and error messages, consider failing fast for critical configurations
  Owner: Engineering team

## Implementation Notes

- Wrap all fs.readFile + JSON.parse sequences in try-catch blocks, logging the file path and error message on failure
- For MCP server configurations, use defensive property access patterns like (data.mcpServers ?? data.servers ?? {}) to handle schema variations
- Consider creating a shared loadJsonConfig(filePath, options) utility that encapsulates parsing, error handling, and optional schema validation
- Document expected JSON schemas in comments or separate schema files to guide validation implementation and user configuration

## Continuation Context


Verify commands:
- grep -r 'JSON\.parse' src/ tests/ | grep -v 'try\|catch' | wc -l | awk '{if ($1 > 0) exit 1}'
- grep -r 'fs\.readFile.*\.json' src/ | grep -A 5 'JSON\.parse' | grep -c 'catch'
- npm test -- --testNamePattern='nested-mcp-behavior' --verbose

Accept when:
- All JSON.parse calls on external configuration files are wrapped in try-catch blocks with error logging
- Integration tests for nested MCP propagation pass, demonstrating successful parsing of multi-level configuration hierarchies
- No unhandled JSON parsing exceptions occur during agent configuration loading in production logs

## Enforcement

- Verified by: Code review checklist requiring try-catch around JSON.parse for external files
- Verified by: Integration tests validating configuration loading with malformed JSON fixtures
- Verified by: Static analysis or linting rules detecting unprotected JSON.parse calls on fs.readFile results
- Violation handling: Pull requests with unprotected JSON.parse on external files must be revised before merge
- Violation handling: Production incidents caused by unhandled JSON parsing errors trigger immediate hotfix and post-mortem
- Violation handling: Quarterly code audits identify and remediate unprotected configuration parsing
- Exception process: Request exception through architecture review board with justification for why validation is unnecessary
- Exception process: Document exception in code comments with reference to approval and rationale
- Exception process: Review exceptions annually to determine if circumstances have changed