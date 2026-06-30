# Validate JSON Configuration Parsing in Test Suites: Tests Validating Mcp

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is active for all test suites that parse JSON configuration files or validate agent configuration outputs.

## Context

- The codebase manages multiple AI agent configurations across different platforms (Gemini CLI, Copilot, Cursor, Claude, OpenCode, Zed, Aider, Junie, Kilocode) that use JSON format for settings and MCP server definitions
- Integration and unit tests must verify that generated configuration files are syntactically valid JSON and contain expected schema elements (mcpServers, contextFileName, servers keys)
- Test suites use JSON.parse() to validate configuration outputs from ruler apply commands and agent-specific settings files across 6 test files
- Configuration validation occurs in multiple test contexts: MCP backup prevention, empty server key fixes, comprehensive integration workflows, merge strategies, and agent-specific settings
- The testing strategy requires parsing JSON from file system reads (fs.readFile) to verify idempotent operations, schema compliance, and proper MCP server configuration propagation

## Problem Statement

Test suites must reliably validate that generated JSON configuration files are syntactically correct and semantically compliant with agent-specific schemas, but parsing failures can cause test failures that are difficult to diagnose without explicit validation of JSON structure before asserting on configuration properties.

## Decision

1. MUST: Tests validating MCP server configurations MUST parse the configuration JSON and verify the presence of configured server entries (filesystem_server, remote_api) within the appropriate key structure

## Policy Block

- MUST Tests validating MCP server configurations MUST parse the configuration JSON and verify the presence of configured server entries (filesystem_server, remote_api) within the appropriate key structure

In scope:
- All test files validating JSON configuration outputs from ruler apply commands
- Integration tests verifying MCP server configuration propagation across agents
- Unit tests for agent-specific configuration generation (GeminiCliAgent, etc.)
- Tests validating merge strategies for MCP configurations
- Tests verifying backup prevention and idempotent operations on JSON configs

Out of scope:
- TOML configuration file validation (uses different parsing approach)
- YAML configuration file validation (uses different parsing approach)
- Markdown content validation (does not require JSON parsing)
- Runtime agent behavior testing (not configuration validation)
- Production configuration loading (covered by application code, not test strategy)

## Rationale

- The evidence shows consistent use of JSON.parse() across 6 test files with 93.17% confidence, indicating an established pattern for validating configuration correctness through parsing
- Multiple agent platforms (Gemini CLI, Copilot, Cursor, OpenCode, Zed, Aider, Junie, Kilocode) use JSON format for configuration, requiring systematic validation in test suites
- JSON parsing serves dual purpose: validates syntactic correctness (will throw on malformed JSON) and enables semantic validation of configuration properties and schema compliance
- The pattern coordinates with fs/promises and path modules to read generated configuration files and verify ruler apply command outputs across comprehensive integration workflows

## Consequences

Positive:
- Test failures immediately identify malformed JSON configurations with clear parse error messages
- Systematic validation ensures all agent configurations meet syntactic requirements before deployment
- Comprehensive coverage across 6 test files provides confidence in configuration generation correctness
- Integration tests can verify complex scenarios like merge strategies, idempotent operations, and MCP server propagation with reliable JSON validation

Negative:
- JSON.parse() throws exceptions on malformed input, requiring try-catch blocks or expect assertions to handle validation failures gracefully
- Large configuration files increase test execution time due to parsing overhead
- Tests become tightly coupled to JSON format, requiring updates if agents migrate to alternative configuration formats
- Parsing does not validate semantic correctness beyond structure (e.g., invalid URLs or command paths will not be caught)

## Alternatives

- Use JSON schema validation libraries (ajv, joi) to validate configuration structure and types (rejected)
  Rejected because: Evidence shows direct JSON.parse() usage without schema validation libraries; adds dependency overhead for test suites that primarily need syntactic validation
  When valid: When semantic validation of configuration values (URLs, paths, types) is required beyond structural correctness
- Read configuration files as text and use regex patterns to validate expected content (rejected)
  Rejected because: Does not validate JSON syntactic correctness; regex patterns are fragile and cannot handle JSON formatting variations
  When valid: When validating non-JSON configuration formats or searching for specific text patterns in logs
- Use agent-specific CLI commands to validate configurations instead of parsing files directly (deferred)
  Rejected because: Not all agents provide validation commands; adds external dependencies and execution overhead to test suites
  When valid: When agents provide robust validation CLIs and end-to-end validation is more important than unit-level configuration testing

## Risks

- JSON.parse() exceptions in tests may not provide sufficient context about which configuration file or agent caused the failure
  Mitigation: Wrap JSON.parse() calls with descriptive error messages or use try-catch blocks that log file paths and agent names before re-throwing
  Owner: engineering team
- Tests may pass with syntactically valid but semantically incorrect configurations (e.g., wrong server URLs, invalid command paths)
  Mitigation: Supplement JSON parsing with explicit assertions on critical configuration values like mcpServers keys, contextFileName values, and schema URLs
  Owner: engineering team
- Changes to agent configuration formats (JSON to YAML/TOML) will break all tests using JSON.parse() validation
  Mitigation: Encapsulate parsing logic in agent-specific helper functions that can be updated independently when formats change
  Owner: engineering team

## Implementation Notes

- Use JSON.parse() immediately after fs.readFile() calls to validate configuration files before asserting on properties
- Structure tests with describe/it blocks organized by agent type (Gemini CLI, Copilot, Cursor, etc.) to isolate validation failures
- For comprehensive integration tests, parse and validate JSON configurations for all agents in the expectedFiles list to ensure complete coverage
- When testing merge strategies or idempotent operations, parse JSON before and after operations to validate content stability and correctness

## Continuation Context


Verify commands:
- grep -r 'JSON.parse' tests/ --include='*.test.ts' | wc -l
- grep -r 'JSON.parse.*readFile' tests/ --include='*.test.ts'
- npm test -- --testPathPattern='mcp|apply|integration' --verbose

Accept when:
- All test files that read JSON configuration files include JSON.parse() calls to validate syntactic correctness
- Integration tests successfully parse and validate JSON configurations for all supported agents (Gemini CLI, Copilot, Cursor, OpenCode, Zed, Aider, Junie, Kilocode)
- Test suites pass with clear error messages when JSON parsing fails, indicating which configuration file and agent caused the failure

## Enforcement

- Verified by: CI pipeline runs test suites that include JSON parsing validation for all agent configurations
- Verified by: Code review verifies that new tests reading JSON configuration files include JSON.parse() validation
- Verified by: Test coverage reports show JSON parsing validation in integration and unit tests for agent configuration
- Violation handling: Tests that read JSON configuration files without parsing validation will fail to catch malformed JSON, causing production issues
- Violation handling: Code review feedback requests addition of JSON.parse() validation for configuration file reads
- Violation handling: CI failures due to malformed JSON configurations trigger investigation of test coverage gaps
- Exception process: Tests validating non-JSON configuration formats (TOML, YAML) use format-appropriate parsing libraries
- Exception process: Tests validating markdown content or text files do not require JSON parsing
- Exception process: Performance-critical tests may defer JSON parsing to reduce execution time if syntactic validation is not the primary concern